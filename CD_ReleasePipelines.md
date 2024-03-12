# CD - Continuous Delivery - Release Pipelines

Deployment could be to a VM, WebApp, or a container based service such as Kubernetes

Delivery could be to Test Env, Staging, and finally Prod.

## Create a Web App resource in Azure
- Create a web app resouce, name it **webapp202020** in our case. Select runtime stack as .NET 6, leave OS as Windows.
- In monitoring disable application insights.

## Update yaml file so it can publish the artifacts after build is completed
Here is updated build pipeline yaml file. This will ensure that build agent wont discard the artifacts after build is complete but keep them somewhere for deployment via release pipeline.

```yaml
trigger:
- master

pool:
#  name: agentpool
  vmImage: 'ubuntu-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '6.x'

- task: DotNetCoreCLI@2
  inputs:
    command: publish
    publishWebProjects: True
    arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)' 
    artifactName: 'webapp-artifact'
```

## First Step
Go to Pipelines -> Release -> New Pipeline
This will let you create release pipeline using classic GUI based version. yaml version is also available as another approach.

- Now we will define our Stage. Under Select a Template, choose "Empty Job", give Stage Name as "Deploy to test env", click on "Save" and close the popup.
- We can have multiple stages - Test, Prod Staging, Prod etc
- Now we add an artifact. Click Add an artifact, select source type as Build, select ptroject, source as webapp pipeline and click Add.
- Go back to Stage section and click on "1 job, 0 task" link.
- In the new window, click on "+" next to Agent Job and search "Azure Web App" and click Add
- Now click on this newly added item, select "Azure Subscription" and authorize.
- With the above step DevOps can now work with Azure resources under your subscription
- Select "webapp202020" as App Name
- Rename the pipeline (webapp-deployment) and click Save. Click on Pipeline tab.

How it works? Once a change is committed, it will create a build and that will trigger the release pipeline created above. However, we can also click "Create Release" to do it at will.

Lets try that and the deployment failed. Click on Logs to see the error. It failed as webapp202020 wasnt available. I had to head back to Azure WebApp202020 resouce, click on Deployment -> Deployment Center and enable Azure Repos there.

## Multiple Stages in a pipeline
- Create another webapp resource with name webappstaging2020
- Connect to linux vm from Mac Terminal: ssh linuxusr@ipaddress
- 


- go back to DevOps and Edit your release pipeline created in previous step
- Hover on to Deploy to Test... stage and click Add button underneath to add another stage
- Click Empty Job, change name to "Deploy to Staging env", Save and OK
- Follow steps as earlier to add new staging env

Lets make a change in index page to trigger build and deploy.

## Approval and Gates
In real world, there are time gaps between stages and only after one stage is tested we move to next. Approvals and gates are for that purpose of user signoffs/approvals. Gates are for automated approach of same. Gates can chcek if all the bugs logged in previous stage have been closed before deployment to next stage (kind of pre deployment gate). Similarly there could be post deployment gates.

### Approvals
Edit the pipeline and click on "Pre-deployment condition" figure. Available triggers are "After release", "After stage", "manual only".<br>
Add an approver too (vkcode7@gmail.com) in this case, lets make a change to homepage to trigger the release.<br>
The deployment now will wait for the approver to login into devops accopunt, go to release pipeline and "Approve/reject". Once approved, deployment will commence.

### Gates
Via gates, one can have various checks - Azure policy compliance chcek, invoke azure functions, REST API, query alerts, query work items, add sonarcloud quality chcek etc.<br>
Scenario 1: (Task queries)  Move from one stage to another if there are no open tasks in Azure Boards for the project. For this create a query that retursn tasks that are not in "Closed" state and save it as shared query. <br>
In this case, we can select "Query work items" in gates, and select the shared query. Configure other options such as continuously evaluate the condition every 2 hours etc. Save and exit.
Note: Go to shared queries -> Security, and give permission to build service.

Scenario 2: (Monitor alerts) In azure you can create an alert on a resource, say Memory threshold alert on webapp and name it say "webapp memory alert". You can then select this alert and create a condition that if alert is fired then dont move to the next stage. Example: you dont want to deploy to staging if web app is exceeding memory limits in test env. One can manually intervene though to pass thru the gate.

Scenario 3: (Policy) You created a policy in azure and non compliance is detected, you can create a gate where deployment will check policy compliance before moving to next stage.

## Agentless task
To your build pipeline you can also add an "Agentless job" so that a manual intervention is needed before deployment steps by Agent are triggered. So a user has to manually "Resume" the deployment after it is triggered.

## Deployment Groups - deploying on Multiple VMs at same time
One may need to deploy on multiple VMs that are part of test env.<br>
Steps<br>
- create 2 Wondows 2019 Server VMs in same resource group and network, install .NET 6 on them
- Click on Piplelines -> Deployment groups and create one
- It will display a PS scrips, run it on the newly created Server VMs.
- Now under target tab, you will see both the machines listed

You can then create a release pipeline wherein add agent as Deployment group type. Make some selections such as IIS server to deploy on etc.<br>
Once deploy is complete, we can use the IP address of the 2 machines to run the site.


## Deploying a web app along with data in Azure SQL database
- Create a SQL database resource in Azure
- Now as in previous steps after we add an Agent Job wherein we search "Azure Web App" and click Add.
- We add another task where in search SQL and add "Add Azure SQL database deployment".
- In the screen that follows we can provide SQL server details and add the script that we want to run on the DB.
- So now in addition to deploying web app, the pipeline will also execute teh SQL script.

Note: Instead of embedding SQL connection details in the c# webapp code, we can create a connection string in azure web app itself(webapp -> Configuration -> App Settings) to store Az SQL Server connection info.

You can also add Azure App Settings in agent job as well and define app settings level stuff there too such as Connection strings.

## Integration with Azure Boards
If you click on your release pipeline -> Edit -> Options, there you can integrate it with azure boards/Jira/repository etc. How it works is that when you commit the change, you specify work item along with commit. So when deployment is complete you will see the link of that in particular work item. That way you have end to end traceability.

## Creating a Docker image (Linux)
- Create a linux VM on azure (Ubuntu 20.04 server)
- Install docker engine via https://docs.docker.com/engine/install/ubuntu/
- Copy webapp publish folder to Linux VM using SFTP tool such as WinSCM
- Create a Docker file named "Dockerfile". Here is how it looks like:
                FROM mcr.microsoft.com/dotnet/aspnet:6.0
                WORKDIR /app
                COPY  . .
                EXPOSE 80
                ENTRYPOINT ["dotnet", "webapp.dll"]

- this file tells Docker about how to contruct a docker image that uses .NET
- FROM instruction tells that base image should be baed on .NET 6.0, create a workdir "/app" and copy all the contents (webapp files), expose port 80 and run webapp.dll
- copy the Dockerfile to Publish folder on linux vm and run the following to build docker image based on Dockerfile: sudo docker build -t webappimage .
- "sudo docker images" will show the newly created image
- The image is then needs to be published to a registry so that we can pull that image and run the container.

## Creating Container Registry resource to publish the image
- Create resource, search for "container registry" and click on Microsoft based
- Give a unique name "localregistry2020" and Create
- You can then go to Repositories under it
<br>
To Publish your image to this registry, you first need to install Azure CLI tool on linux vm. 
<br>
Here are the steps involved:<br>

1. Install the Azure CLI via the following URL<br>
https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt

2. First log into the Azure container registry from CLI. Under localregistry2020 -> Access Keys, we can get the password<br>
sudo az acr login --name localregistry2020 --username localregistry2020 --password ahXW2jdkwiqgj1WST3oB2/f0SWzhLNi/xntmfM3aZY+ACRBYs0gU

3. Then tag your image<br>
sudo docker tag webappimage localregistry2020.azurecr.io/webappimage

4. Then push the image to the Azure Container registry<br>
sudo docker push localregistry2020.azurecr.io/webappimage

Once published you can see the image under Repositories in azure under your container registry.

### Creating container instance
Once published we can create a resource "Container instances" type and select our localregistry2020, select the container webappimage and "Create".<br>
With that container instance will be in place, you can copy the IP and access the web app from internet.

## Implementing the previous steps via Azure Pipeline itself - build a docker image and push it to containeer registry (CR)
- Create a copy of webapp code and create a new azure repo webapp-docker with it
- Delete the yaml pipeline file(s)
- Create a Dockerfile and copy it where source code is (where csproj is).
- Click on "Setup Build", in the configure step it will show you Docker/Kubernetes options because it detects Dockerfile in source
- Select "Docker - Build and push an image to Azure Container Registry"
- It will ask subscription ddetails and ask you to select container registry created previously
- It will then show you generated pipeline yaml file
- The yaml file wont create the correct image and need some tweaks (it uses code to build image instead of Publish artifacts)
- Edit the yaml file, and commit it
- The "pwsh" command is to see the build outputs (for easier debugging) on the MS VM (this is not a self hosted where we can browse and see)
- The above will create a List Content task that we can see in Build outputs
- I was running into issue and have to fix the path in yaml for task Docker@2 => buildContext: '$(Build.ArtifactStagingDirectory)/s'
- Per List Content, output was in Directory: /home/vsts/work/1/a/s
- Commit will trigger the build and deploy the image in CR
- You can see it in localregistry2020 | Repositories | webappdocker
- You can then test it by creating a container instance using that image webappdocker and opening the page in browser

Here is the yaml file:
```yaml
  # Agent VM image name
  vmImageName: 'ubuntu-latest'

stages:
- stage: Build
  displayName: Build and push stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '6.x'

    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: build
        projects: '**/*.csproj'
        arguments: '--configuration $(buildConfiguration)'

    - task: DotNetCoreCLI@2
      inputs:
        command: publish
        publishWebProjects: True
        zipAfterPublish: false
        arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    
    - pwsh: |       
       Get-ChildItem -Path $(Build.ArtifactStagingDirectory)\*.* -Recurse -Force | Out-String -Width 180
      errorActionPreference: continue
      displayName: 'List content'
      continueOnError: true
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        buildContext: '$(Build.ArtifactStagingDirectory)/s'
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest

```

## Using Release pipeline to launch Az Container instances
In the previous step we pushed an image into container registry and then use teh Azure portan to launch an instance.
This can be done using Azure CLI via release pipeline. The docker image is already in the CR, all we have to do is launch an instance. 

Go to Pipelines -> Release -> New Pipeline
This will let you create release pipeline using classic GUI based version. 

- Now we will define our Stage. Under Select a Template, choose "Empty Job", give Stage Name as "Launch container instance", click on "Save" and close the popup.
- We can have multiple stages - Test, Prod Staging, Prod etc
- We dont need an artifact as Docker Image is already in CR.
- Go to Stage section and click on "1 job, 0 task" link.
- In the new window, click on "+" next to Agent Job and search "Azure CLI" and click Add
- Now click on this newly added item, select "Azure Subscription" and authorize.
- Select script type as "Batch", and s cript location as inline script, and paste the az command
- az container create -g devops-grp --name appinstance20030 --cpu 1 --memory 1  --ports 80 --ip-address Public --image localregistry2020.azurecr.io/webappdocker:latest --registry-username localregistry2020 --registry-password XeVWwKmmKD2IAS34A8B8i7E+MWJlneju
- Rename the pipeline (container-launch) and click Save. Click on Pipeline tab.


## Kubernetes Service
Add a resource of type "Kubernete Service" (KS), and create a Kubernetes Cluster. Give it a name "devcluster".
- Under Node Pool, click on "Add a node pool" and configure your cluster (I named it kuberpool, also cretaed another one named agentpool)
- Under integration select your CR localregistry2020 and create the cluster
  
- Create two files app.yaml and service.yaml, save these under "Manifests" folder in the webapp-docker project
- app.yaml (deployment manifest file)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: localregistry2020.azurecr.io/webappdocker:latest
        name: myapp
        ports:
        - containerPort: 80
          name: myapp
```

-service.yaml (service manifest file)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: appservice
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
### Deploying WebAPP manually via Azure Portal
If we want to configure our cluster manually, we need to copy paste these two deployment files in azure portal. Lets do that.<br>
- click on "devcluster" we created
- click "Kubernetes resources" -> Workloads -> Create [Apply with YAML]. In the text area copy paste the app.yaml, and click "Add"
- click "Kubernetes resources" -> Services and Ingresses -> Create [Apply with YAML]. In the text area copy paste the service.yaml, and click "Add"
- Go back to Workloads and see that "myapp" is now ready
- Go back to "Services and Ingresses", verify that LoadBalancer is ready with an Expernal IP address.
- Click and that IP and voila, your webapp is launched in a web browser

### Deploying via Azure classic Build Pipelines
- Delete the myapp entry from Workloads that we created in previous step, Delete LoadBalancer entry from Servives and Ingresses too
- Go to Pipelines/webapp-docker/<Edit>, basically build pipeline we did for pushing the build to CR
- We are now Editing the same to publish it to Kubernetes cluster as well
- Before we do that create a Kubernetes service connection [Project Settings -> Pipelines -> Service Connection]
- New Service Connection -> Kubernetes, Select Azure Subscription, Select cluster, pick Namespace and default, service conn name as "kubernetes-connection" and select Grant access to pipelines.
- Now we have 2 yaml files in Manifests folder
- Lets continue with Editing the azure-pipelines.yaml
- On right side, in Tasks, look for Kubernetes, click on "Deploy to Kubernetes", in the Manifests text box, type app.yaml and hit Add
- Edit the entry by updating app.yml and service.yml entries so it looks like this
  ```yaml
      - task: KubernetesManifest@1
      inputs:
        action: 'deploy'
        connectionType: 'kubernetesServiceConnection'
        kubernetesServiceConnection: 'kubernetes-connection'
        manifests: |
          $(Build.SourcesDirectory)/Manifests/app.yml
          $(Build.SourcesDirectory)/Manifests/service.yml
  ```
- Click on Save and commit to trigger the build
- Now once deployment is complete, you will see "myapp" in "Kubernetes resources" -> Workloads and "appservice" in "Kubernetes resources" -> Services and Ingresses
- This confirms that deployment in Azure Cluster is complete
- There you can see the public IP address, click on it and Voila, the website is open

### Deploying via Azure Release Pipelines
Previousy we saw how we deploy using the classic build pipeline yaml file (that builds the code as well as deployed on Kuber cluster). In this we wiull do it via Release pipeline.<br>
- delete the myapp and appservice from "Kubernetes resources"
- delete the "KubernetesManifest@1" entry from yaml file we updated in previous section so that build yaml file wont dd it
- instead add a Publish task so that the artifacts (the two yaml files - app and service) are available in "Release" pipeline from where we will publish them to cluster
  ```yaml
      - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Pipeline.Workspace)'
        artifact: 'sourcefiles'
        publishLocation: 'pipeline'
  ```
- the above change will trigger a build, wait for it to complete and click on "Build" section to see the log summary
- their click on "1 artifact produces" link, it will open sourcefiles, expand the "s" directory, under which you will see Manifests folder with 2 yaml files.
  
- Go to Pipeline -> Releases -> Create a New Release pipeline, say named "Kubernetes Deployment"
- Start with Empty Job, stage named to "Deployment to Kubernetes", Save and close
- Click on "1 job, 0 task" link and then "+" next to Agent Job
- Search for Kubernetes and Add "Deploy to Kubernetes"
- Change "Display name" to "app.yml deployment"
- Configure and select Kubernetes Connection created earlier in Project Settings
- We need to add artifacts in Manifests box, which are missing, so go back to Pipeline tab and "Add an artifact"
- Select Build -> webapp-docker and click Add
- Go back to Tasks tab, and in Manifests box, browse to app.yml (we can see source code now) and click on Save
- Now click "+" again and this time add service.yml
- So we will have two afent jobs - "app.yml deployment" and "service.yml deployment"


  




  





