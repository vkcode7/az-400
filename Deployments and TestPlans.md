# Blue Green Deployment
Current prod is called Blue and another env is created for real users called Green. Once everything looks ok, prod env can be switched to Green, or users can be moved back to Blue.

# Azure Web App - Deployment Slots
- Version 1 is on Prod slot
- Version 2 is on Staging slot

Once testing is completed, users are moved to Staging slot and it becomes the prod env, and hence very low downtime.<br>

Under your Web App resource -> Deployment -> Deployment Slots, you can create a staging slot and deploy your version 2 app on that. The current app running on the webapp is tagged as Production, so you have two web apps running in a web app resource - Production and Staging. There is also a "Swap" button there under Deployment Slots, and once testing is completed you can click Swap and thus swap the environment, making staging as new production.

# Azure Web App - Deployment Slots via Release pipelines
The creation of a staging slot and deployment to it can be done via Azure Release pipelines.
- Create a Agent job task of type Azure CLI and add CLI command to create a staging slot
- Add "Azure App Service Deploy" task to deploy the app on staging slot
- Add another Stage "Swap" for Swapping the slots, add a preapproval trigger to that as well
- In Swap stage add a task of type "Azure App Service manage"
- Add an "Agentless job" to Swap stage to conduct a manual intervention task to do a sanity check after swap is done.
- Add another Agent job of type Azure CLI with commands to delete the staging slot

# Canary Deployment
When a new feature is released and is available to a subset of users. You can direct a certaon percentage of users to env with new feature.

# Azute Traffic Manager (TM)
This is DNS based traffic load balancer. You can distribute traffic across different Azure regions or based on different routing methods.

- To create a traffic manager resource, look for "Traffic Manager profile" and hit Create
- Select Routing method and it will be created at a Global level.
- IN the configuratuon setting, tweak the setting (HTTP etc)
- Go to Settings -> Endpoints and add endpoints (your apps)
- You can then take the DNS name from TM and use it to access the app.

# Azure TM and Release pipelines
Assume we already have a TM in place and it has only one endpoint - the prod. As part of Release pipeline we will add another endpoint to TM for canary deployment. For that:
- You can add an Agent Job of type "Azure App Service Deploy" and select the deploymentapp1000-staging webapp to deploy on.
- Add "Azure PowerShell" job next and this will be used to create TM endpoint (see below)
```
$TrafficManagerProfileName="deploymentprofile"
$ResourceGroupName ="template-grp"
$TargetResourceId="/subscriptions/6912d7a0-bc28-459a-9407-33bbba641c07/resourceGroups/template-grp/providers/Microsoft.Web/sites/deploymentapp1000-staging"
$TargetEndPoint="deploymentapp1000-staging.azurewebsites.net"


$TrafficManagerEndpoint = New-AzTrafficManagerEndpoint -Name "staging" `
-ProfileName $TrafficManagerProfileName -Type AzureEndpoints `
-TargetResourceId $TargetResourceId -EndpointStatus Enabled `
-Weight 100 -ResourceGroupName $ResourceGroupName 

Add-AzTrafficManagerCustomHeaderToEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint -Name "host" -Value $TargetEndPoint
Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint
```

- Add another Agent job of type CLI to change the weightage percentage of redirected users
```
$TrafficManagerEndpoint = Get-AzTrafficManagerEndpoint -Name "production" -Type AzureEndpoints -ProfileName "deploymentprofile" -ResourceGroupName "template-grp"
$TrafficManagerEndpoint.Weight = 1
Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint


$TrafficManagerEndpoint = Get-AzTrafficManagerEndpoint -Name "staging" -Type AzureEndpoints -ProfileName "deploymentprofile" -ResourceGroupName "template-grp"
$TrafficManagerEndpoint.Weight = 1000
Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint
```

# Rolling Deployments
You update one machine at a time.

# Making Switch between Blue/Green Env

### VMs setup
We can create 2 VMs of Windows Server 2019 Datacenter. One will represent Blue (prod) and another Green (staging) env, and we will make a switch using Load Balancer. Create both machines as part of same virtual network. Login into the VM using RDP (remote desktop connection). You will be presented with a "Configure this Local Server" page, click on Add roles and features, in the screens click on "Web Server (IIS)" and hit install. This will install IIS. Do the same on 2nd VM. On these machines in C:\inetpub\wwwroot, place a Default.html with Welcome message (Welcome to VM 1 or 2). If you take IP of these and do x.y.z.a/default.html, your pages will open.

Next we will put these behind a Load Balancer.

On each machine go to Settings -> Networking, look for "Network Interface" and click on it. For network interface, click on Settings -> IP configurations. Click on ipconfig1 (line where public IP is displayed) and "disassociate" the public IP address, hit Save.

### Load Balancer
Create New resource, look for "Load Balancer" and hit Create, name it app-balancer. Select standard as SKU, type as Public and tier as Regional. Next screen is "Frontend IP configuration", click on "Add a frontend IP config", give it a name "load-frontend-ip", create new Public IP address named "load-ip", click on Add. Leave everything else as is and Create.

- Now click on Settings -> Backend pools.
- Add backend pool and name it "production-pool" and add the VM 1 using its private IP address. You can see this IP in VM -> Overview.
- Add another backend pool with name "staging-pool" and add VM 2 to it.
- Next go to LB -> Settings -> Health Probe and Add a probe listening on port 80 and click Add.
- Next go to LB -> Settings -> Load Balancing Rules, Add and select "production-pool" as Backend Pool, 80 as port, 80 as backend port, selecxt health probe we created and click Add
- Now if you take the front end IP of LB, you will be redirectd to prod VM 1.
- You can update your DNS Record on godaddy.com to use the friontend IP, so your website will point to LB IP.

Now once you test the changes in Staging, all you have to do is go back to LB -> Settings -> Load Balancing Rules, and change the Backend Pool to staging-pool. We are telling LB that new requests should be directed to staging pool.

### Making switch via Pipeline
Create a stage with Empty job named Swap Pool. Add Agent Job of type Azure Powershell script and use the following PS lines as inline script
```
$ResourceGroupName="load-grp"
$LoadBalancerName="app-balancer"
$BackEndPoolName="staging-pool"

$LoadBalancer = Get-AzLoadBalancer -Name $LoadBalancerName -ResourceGroupName $ResourceGroupName
$Probe=Get-AzLoadBalancerProbeConfig -Name "probeA" -LoadBalancer $LoadBalancer
$BackendPool = Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $ResourceGroupName -LoadBalancerName $LoadBalancerName -Name $BackEndPoolName

$LoadBalancer | Set-AzLoadBalancerRuleConfig -Name "RuleA" -FrontendIPConfiguration $LoadBalancer.FrontendIpConfigurations[0] -Protocol "Tcp" -FrontendPort 80 -BackendPort 80 -BackendAddressPool $BackendPool -Probe $Probe
$LoadBalancer | Set-AzLoadBalancer
```

## Package Management
### How to Publish
Azure Artifacts - developers can publish their packages and consumes other.
Lets say we have a .NET library "CoreServices" that priovides Core functionality that other projects in org can use. To store this library we can use Azure Artifacts on DevOps page for the project in DevOps.
- First thing we have to do is click on "Create Feed" and creare a feed with a name, say CoreServices in this case
- Next click on "Connect to feed" and it will show you how to install the package from this feed, click on NuGet to see how to publish using NuGet
- Next we will publish the package
- Per the NuGet instructions to publish, you need to download nuget.exe and place it in project folder
- Next add nuget.config as per the instructions in "Connect to feed"
  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <packageSources>
      <clear />
      <add key="CoreServices" value="https://pkgs.dev.azure.com/vkcode7/_packaging/CoreServices/nuget/v3/index.json" />
    </packageSources>
  </configuration>
  ```
- Next build the package by opening the cmd, and run the cmd: ./nuget.exe pack ./CoreServices.csproj
- Now publish it using: nuget.exe push -Source "CoreServices" -ApiKey Core CoreServices.1.0.0.nupkg
- We can now go back to Artifacts page and see that our package is published and available
- We can also publish the package from pipelines yaml itself
```yaml
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- main

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '6.x'
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'

- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: build
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'

- task: NuGetCommand@2
  inputs:
    command: pack
    packagesToPack: '**/*.csproj'
    versioningScheme: byEnvVar 
    versionEnvVar: Build.BuildId
    packDestination: '$(Build.ArtifactStagingDirectory)'

- task: NuGetAuthenticate@0
  displayName: 'NuGet Authenticate'
- task: NuGetCommand@2
  displayName: 'NuGet push'
  inputs:
    command: push
    packagesToPush: '$(Build.ArtifactStagingDirectory)\*.nupkg'
    publishVstsFeed: 'AgileProject/CoreServices'
    allowPackageConflicts: true
```

### How to consume it
From the Artifacts, go to Connect to Feed, and copy teh URL for VS and then add the package source in VS settings. It will then show CoreServices package that you can add to project.

### Upstream sources
We can also publish the projects from upstream sources such as nuget to Artifactory and later install from there.

## Azure App Config
Under each resource there is a configuration setting where you can define key/value pair such as connection strings. Instead we can do that at app level by creating a App Config resource:

Azure App Configuration allows developers to store, retrieve, and manage access to application configuration all in one place. It is easy to set up and simple to use from any application. It gives developers the ability to modify an application's behavior on demand without having to redeploy the application, while meeting rigorous performance,security, and compliance requirements. Beyond configuration, it also supports feature flagging capabilities to customize feature rollouts and limit new feature access to target groups.

App Configuration provides:

A fully managed service that can be set up in minutes.
Flexible key representations and mappings.
Versioning with labels.
Point-in-time restores of configuration up to 30 days later.
Native integration with popular frameworks, including .NET, Java, Python, and Javascript.
Enhanced security through Azure-managed identities.
Complete data encryption at rest or in transit.
CI/CD integration with Azure Pipelines and Github Actions.
Key Vault Reference support for direct access to secrets and certificates.
Feature management with filters to limit the time or target group availability of each flag.

# Test Plan
Azure DevOps has Test Plans functionality but is a paid one. You can create a Test Plan and under Test Plan can create Test Suites that can be linked to User Stories. The Test Suites can then have Test Cases under it to test the user story. This was Test Plan can be linked to Azure Boards. Test suites can be assigned to testers who can run them, and reports can be generated.


