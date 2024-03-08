## Continuous Integration - Build Pipelines
CI is automatic building and testing code everytime someone commits code changes

concept behind CI: code is committed -> built -> tested

Rather than various team members committing their changes and than build, each commit (pull request) triggers a build. So any build issue, if there can be identified early enough.

## Azure pipelines
- build and test your projects
- can be used for both CI and CD
- supports multiple languages
- can deploy to multiple targets / environments

Pipeline is defined using YAML (markup language), ex: azure-pipelines.yaml. The file is also added to git in your code repo. There is also a GUI available to define the pipelines.

Assuming we have webapp code in Azure Repos and there is a default branch set (master, main, dev, uat etc), here are steps to create a pipeline:
- click Pipelines
- click New Pipeline, it will then ask where source code is? Azure Repos, Github, Bitbucket? Select Azure Repos here in this example
- once repo is selected it will ask what type of application? .NET Webapp, Desktop, Xamarin blah blah
- select .NET Webapp and it will open up azure-pipelines-1.yml which is a YAML based pipeline definition file.

Here is how it looks like:

trigger is master, when any change to master branch is made, the build will be triggered. In this it will use a MS VM by the name of windows-latest, it will copy the code there, build it and then destroy the VM. However, you can provide your own custom VM.

```yaml
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- master

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'

- task: VSBuild@1
  inputs:
    solution: '$(solution)'
    msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:DesktopBuildPackageLocation="$(build.artifactStagingDirectory)\WebApp.zip" /p:DeployIisAppPath="Default Web Site"'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'
```

click and "Save and Run", this will commit it to master branch and run it.

It will then initialize the job, checkout webapp code, build the image where build will happen etc.

Note: When you open the .yaml file, on right side you see the tasks, any of these can be added to the yaml using GUI interface.

### Self hosted agents
Benefits
- persist the builds
- have custom software installed

#### How to create
- Create a Windows 2019 VM, assuming that is your build box (lets say its name is agentvm)
- Install .NET 6.0 SDK and Git on that (.NET 6.0 is webapp requirement)
- Click on agentvm VM and then "Connect", that will download a RDP file
- Click on that RDP file on a windows laptop, that will open the Remote Desktop and connect to agentvm
- Once connected, open Edge and download .NET 6.0/GIT and install them

- Next on agentvm, open dev.azure.com, from your user profile, create a PAT (personal access token), copy it in notepad
- Click on Organization Settings -> Pipelines -> Agent Pools -> Default
- Under that click "New Agent", select Windows x64 and "Download", extract the file in c:\vsts-agent....
- Open PowerShell and CD to above folder
- Run .\config.cmd on cmd prompt in PS, enter PAT token when asked, and c:\work as work folder. Create c:\work on agentvm too.
- Enter Y for Run Agent as Service, also Y for enable SERVICE_SID_TYPE_xxx
- Press Enter for subsequent questions.

If you go back to Org Settings -> Pipelines -> Agent Pools -> Default -> Agents, you will see agentvm as "Online"

Go to AgileProject -> Pipelines -> Webapp and click "Edit" button on top right side. This will open the .yaml file. Change it as below.

Note: In our case we created a new pool 'agentpool' instead of using 'Default'.

```yaml
pool:
  vmImage: 'windows-latest'
```

```yaml
pool:
  name: agentpool
  vmImage: agentvm
```
Run the pipeline and it will use the agentvm.

Here is the updated complete yaml file. Notice we replaced the VSBuild task with UseDotNet@2, otherwise we have to install VS on agentvm.

```yaml
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- master

pool:
  name: agentpool
  vmImage: agentvm

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

- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: build
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'
```

For more details about build variables refer to YAML schema for azure pipelines on MSDN.
https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml

## Mend Tool
This is an open souce software security and compliance management tool. It integrates with Azure Devops set of tools in the pipelines and look for any software vulnerabilities or licensing issues (say in nuget packages from 3rd parties used in builds).

To install tools such as Mend, go to Azure DevOps, Org Settings -> General -> Extensions -> Browse Marketplace and search for Mend or others as needed.

This tool needs you to use MS Entra (aka Active Dir). This can be done in devops (https://dev.azure.com/vkcode7) by clicking on user in top right corner and switching to AD (Switch Directory). Select the Default Directory and click on Switch.

The above step will create a brand new DevOps org (earlier vkcode7 was created using MS Account). The new one is named as vkcode7ad, it will also keep the existing one and we can switch between these.

Only diff is that vkcode7ad is associated with AD (Entra now) account and vkcode7 with MS account.

Click Org Settings -> Microsoft Entra to see that you are connected to AD.

We can now go to Extensions, search Mend and install "Mend Bolt". Go back to Org Settings and you will see "Mend" under "Extensions" section. Click on Mend and fill the info.

### Using Mend Tool

Create a basic project say BasicProject and from git add the webapp code to the BasicProject.

Select the repo and click on "Set up Build". Choose ASP.NET Core. The yaml file will open, delete the VSTest2 task from that and add the Mend task by searching Mend on right side from assistant. Leave everything as is and click Add. This will add the following to yaml file:

```yaml
- task: WhiteSource@21
  inputs:
    cwd: '$(System.DefaultWorkingDirectory)'
```

Next do Save and Run, this will kick the job.

Once job is complete and you go back to Pipelines, you will see extra tab "Mend Bolt". You can see the report there for Licensing risks and Securities vulnerability.

All this new org vkcode7ad was created only for the demo of Mend tool. We will go back to our old org vkcode7 back.

## Reconnecting back to vkcode7 repo from vkcode7ad
```bash
(base) gs@GSs-MacBook-Pro webapp % git remote remove origin
(base) gs@GSs-MacBook-Pro webapp % git remote add origin https://vkcode7@dev.azure.com/vkcode7/AgileProject/_git/webapp
(base) gs@GSs-MacBook-Pro webapp % git branch --set-upstream-to=origin/master  master 
Branch 'master' set up to track remote branch 'master' from 'origin'.
(base) gs@GSs-MacBook-Pro webapp % git pull
```

## Adding a Test Project to existing webapp project (webapps repo)
Right click on webapp solution, Add New Project, search xUnit and add a "xUnit Test Project", name in webtest, select .NET 6.0. <br>
Rename UnitTest1.cs to WebTest.cs. This is how the file looks like:
```code
namespace webtest;

public class WebTest
{
    [Fact]
    public void DemoTest()
    {
        int i = 1;
        bool result = false;
        if (i == 1)
            result = true;
        Assert.True(result, "value should be equal to 1");
    }
}
```
Build it and run it from VS (View -> Tests). It passed.

yaml looks like this:
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
  displayName: Build
  inputs:
    command: build
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'

- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: build
    projects: '**/webtest.csproj'
    arguments: '--configuration $(buildConfiguration)'

- task: DotNetCoreCLI@2
  displayName: Test
  inputs:
    command: test
    projects: '**/webtest.csproj'
    arguments: '--configuration $(buildConfiguration)'
```

Under Tests tab, we can see the test results.

## Adding code coverage
For this add a folder named Module in webapp project and under that add a class Functions as below:
```code
using webapp.Modules;

namespace webtest;

public class WebTest
{
    [Fact]
    public void DemoTest()
    {
        int i = 1;
        bool result = false;
        if (i == 1)
            result = true;
        Assert.True(result, "value should be equal to 1");
    }

    [Fact]
    public void CheckAddFunction()
    {
        Functions func = new Functions();
        
        int y = func.Add(3, 2);
        bool result = false;
        if (y == 5)
            result = true;
        Assert.True(result, "value should be equal to 5");
    }
}
```

Update WebTest.cs to

```code
using webapp.Modules;

namespace webtest;

public class WebTest
{
    [Fact]
    public void DemoTest()
    {
        int i = 1;
        bool result = false;
        if (i == 1)
            result = true;
        Assert.True(result, "value should be equal to 1");
    }

    [Fact]
    public void CheckAddFunction()
    {
        Functions func = new Functions();
        
        int y = func.Add(3, 2);
        bool result = false;
        if (y == 5)
            result = true;
        Assert.True(result, "value should be equal to 5");
    }
}
```

From NuGet PM, Add Coverlet.msbuild to webtest project.

Here is the updated yaml

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  displayName: Install .NET 6
  inputs: 
    packageType: 'sdk'
    version: 6.x

- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: build
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'

- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  inputs:
    command: test
    projects: '**/webtest.csproj'
    arguments: '/p:CollectCoverage=true /p:CoverletOutputFormat=cobertura /p:CoverletOutput=./MyCoverage/'
    publishTestResults: true

- task: PublishCodeCoverageResults@1
  displayName: 'Publish Code Coverage Results'
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Build.SourcesDirectory)/**/MyCoverage/coverage.cobertura.xml'
```

You will then see a "Code Coverage" tab once the pipleline finishes. The objective is to see if your Unit Tests covers 100% of your code.


## Azure Pipeline and GitHub

Do a "New Pipeline", it will ask for repo type, select GitHub, it will ask for GitHub user/pwd and ask for the repo that you want to use to build pipeline. Will ask you to configure pipleline yaml, and then commit the yaml file. That will initiate a build. Only diff is that now code will be picked up from github instead of azure repo.

### Adding status badge
Click on "Status Badge" (click 3 vertival buttons next to "Run Pipeline" button of yout guthub pipeline), copy the sample markdown and copy it in the ReadMe.md on your github account. Once committed you will see Azure Pipelines status badge. The update to ReadMe will also trigger a build on azure.

Also in Org Settings -> Settings, turn off "Disable anonymous access to badges". Also do the same in Project Settings -> Pipeline -> Settings. After this you can see the suucess status on github repp badge.

One can also use PAT to connect to github. For that go to Settings -> Dev Settings -> PAT and generate a new token. Copy the token. Now go back to Azure, Project Settings -> Pipelines -> Service Connection, select GitHub and paste the token, give it any name and save.

Here you can also select "Other Git" when creating a pipeline, which could be any git or github. Use your service connection to connect. In this one, yaml is not used, instead a GUI based page is used to configure your pipeline.

## SonarCube and SonarCloud - Code quality tool.
First step is to sign up on SonalCloud, open source projects are free. Can login using GitHub or Azure Devops. It will ask to chose the repos from them, import the org and create a free account. YOu can generate the security token there to use in Azure.

Install the SonarCloud extension in Azure in order to use it.

Now go to Project settings -> Service connection and add the token created on sonar website here. This will connect azure to sonar account.

You can then edit the yaml pipeline file, by looking for SonarCloud on right and add entries for Prepare Analysis, Code Analysis and Publish. It will ask for org name from SonarCloud account and also the project key (has to be created on sonarcloud site by creating a project there).

Once run, you can then see the report details (code issues etc) on SonarCloud site.

## Parallel Jobs
So far we ran 1 pipeline job at a time but in a large org, a pipeline can have multiple jobs that can run in parallel. To do that we have to buy license from Microsoft.
Go to Org Settings -> Pipelines -> Parallel Jobs to see the details.

## Pipeline Caching
Instead of installing dependent packages (nugets etc) with each Job run, the pipeline can cache those and reuse them for subsequent job runs. This can be done using "Cache" task in yaml.

## Integration with Slack and Teams
Azure pipelines can be integrated with these, just like Azure boards.

## Github Actions
Integrating github with Azure pipelines. This is done by creating a yaml file in the .github folder -> workflows (the .github will be sibling of project folder), adding an azure pipeline name into that file, azure project URL, a PAT created on azure devops so github can connect to pipeline. The PAT is not directly stored in yaml but you can go to github settings and there you can create a Secret token and save it there (GitHub -> Project Settings -> Security -> Secrets -> Action). The secret token name can then go to the yaml file.


## Jenkins - CI tool

### Jenkins installation on a ubuntu linux server
For using Jenkins, we need to install it on a VM first.

Lets create an "Ubuntu Server 20.04 LTS" vm. Name it jenkinsvm. Select Auth type as Password with user as linuxusr/G...ru...m123!<br>
Virtual network is devops-grp-vnet (newly created), public ip name is jenkinsvm-ip. Create the VM.

After it is created and launched, note down the IP Address. we need it to connect to this VM and install jenkins tool.<br>
Go to putty.org and download the PuTTY tool, we need it to connect to linuxvm on Windows. On Mac we can simply use Terminal windows to connect to our jenkinsvm using ssh linuxusr@52.168.83.77

```bash
Last login: Fri Mar  1 12:29:25 on ttys006
(base) gs@GSs-MacBook-Pro webapp % ssh linuxusr@52.168.83.77
The authenticity of host '52.168.83.77 (52.168.83.77)' can't be established.
ED25519 key fingerprint is SHA256:5sGbzCyqD1gMz1tv8115qIMr+yJT18g+bYCuunXA2CQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '52.168.83.77' (ED25519) to the list of known hosts.
linuxusr@52.168.83.77's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1057-azure x86_64)
```

Here are the jenkins installation commands that we need to run on the linux vm

```bash
# commands from DevOps course material
sudo apt-get install openjdk-11-jdk-headless
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo ufw allow 8080
sudo systemctl start jenkins
sudo systemctl status jenkins
sudo -s
```

```bash
# Steps to install from jenkins from chatgpt that we ran
sudo apt update #ensure package index is up to date
sudo apt install default-jdk #install jdk
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -   #Add Jenkins Repository Key
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'  #Add Jenkins Repository
sudo apt update #update package index
sudo apt install jenkins #install jenkins
```

The jenkins install was failing so ran these too
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```

we need to open port on jenkinsvm, head to azure -> vm -> Networking and create an inbound port security rule for port 8080. Just leave everything as is and hit "Add".<br>
Head back to terminal and continue with jenkins installation

```bash
sudo ufw allow 8080 #create firewall rule at linkux level to allow traffic on 8080
sudo systemctl start jenkins # start jenkins instance
sudo systemctl status jenkins # check status of jenkins
```

With this Jenkins is now running, go to a web browser and type => http://52.168.83.77:8080/<br>
This will open a page that prompts to unlock jenkins by entering pwd which is at: /var/lib/jenkins/secrets/initialAdminPassword on linux vm.

Head back to terminal

```bash
root@jenkinsvm:/home/linuxusr# sudo -s
root@jenkinsvm:/home/linuxusr# cd /var/lib/jenkins/secrets
root@jenkinsvm:/var/lib/jenkins/secrets# more initialAdminPassword
9561bfbd8c054eb5b37dd9e874bbaaee
root@jenkinsvm:/var/lib/jenkins/secrets#
```

9561bfbd8c054eb5b37dd9e874bbaaee is the password. use it to unlock on the webpage and click on "Install selected plugin" on webpage.<br>

It then asks to create Admin user. Created user jenkinsadmin with pwd jenkins. Finish and start using Jenkins.

### jenkins - creating a build pipeline

Jenkins is a CICD tool but on the same machine we have to install other tools needed for build such as .NET Core.


#### Create a locak webapp and push it to github
Created a webapp project locally using local git
added and committed the files locally
```bash
git add .
git commit "initial commit"
```
Create a github repo by the name of webapp and run the following in bash
git remote add origin https://github.com/vkcode7/webapp.git
```bash
git branch --set-upstream-to=origin/main master
git fetch origin main
git merge --allow-unrelated-histories origin/main
git push origin master:main
```




