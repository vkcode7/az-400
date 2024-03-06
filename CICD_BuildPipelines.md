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

Go to AgileProject -> Pipelines -> Webapp and click "Edit" button on top right side. This will open the .yaml file. Change the vmImage under pool to Default.
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
