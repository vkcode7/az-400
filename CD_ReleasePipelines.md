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
- 







