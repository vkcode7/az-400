Security and Compliance

## Variables
Variables can be used in yaml files as below
```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
  databaseServerName: server4000

steps:
  - script: |
      echo $(databaseServerName)
  - bash: echo "##vso[task.setvariable variable=databaseServerName]server5000"
  - script: |
      echo $(databaseServerName)  
  - script: |
      echo ${{ variables.databaseServerName }}  
```

## Variable Groups
Here you can store values and secrets. Variable groups can be used across multiple pipelines in a project. 
Go to Pipelines -> Library. Here can create a Variable Group, and add variable names/values under it. Suppose name of Var Group is SharedVariables and we created a variable "databaseServerName" in it.<br>
We can then use it in yaml pipeline as:
```yml
trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
- group: SharedVariables

steps:
  - script: |
      echo $(databaseServerName)
```

In classic pipeline (gui based Release pipeline), you can access these by going to "variables" tab and clicking on "variable groups", there you can link the SharedVariables and can then access the variables from that group.

## Azure Key Vault
Azure Key vault is a managed resource where we can safely store our keys, passwords and certificates.
- Create Resource -> Key Vault and Create
- Once created you can go to Settings and there select Keys, Secrets or Certificates to store these
- We can create a secret to store dbpassword.
- We can use a secret by seatrching and adding "Key Vault" in yaml pipeline file

However, befoer we can access the Key Vault we need to give access to pipeline. To do that follow these steps:
- go to Project Settings -> Pipelines -> Service Connections
- Click on "Azure Subscription 1" service connection
- Under Details, click on Manage Service Principal
- A new window will appear with a Display Name (vkcode7-AgileProject-f23a81dd-b03a-4625-a27d-11e4f12844f5)
- Copy that and go back to Key Vault and there, search and give read access to this access object identity

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:

- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
    KeyVaultName: 'appvault787878'
    SecretsFilter: '*'
    RunAsPreJob: false
  
- script: |
      echo $(dbpassword) > dbpassword.txt

- task: CopyFiles@2
  inputs:
    Contents: dbpassword.txt
    targetFolder: '$(Build.ArtifactStagingDirectory)'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'secrets'
    publishLocation: 'Container'
```

Note: The output in logs will be starred (****) as this is a secret, you can only use it in your scripts but cannot output it in logs. To see the value you have to use the following in yaml file:<br>
      echo $(dbpassword) > dbpassword.txt

This will output the value in a text file. The CopyFiles@2 then copies it in build artifacts directory which we can access and see in logs itself.

### Azure Key Vault and ARM Templates

In Access policies of your Key Valut, give access to ARM templates

Static reference: You take ID of key vault and use it to access secret as below.
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "SQLLogin": {
      "value": "sqlusr"
    },
    "SQLPassword": {
     "reference": {
        "keyVault": {
          "id": "/subscriptions/6912d7a0-bc28-459a-9407-33bbba641c07/resourceGroups/template-grp/providers/Microsoft.KeyVault/vaults/keyvault67767"
        },
        "secretName": "dbpassword"
      }
    }
  }
}
```

Dynamic reference: We cannot use variables from nested ARM templates with dynamic reference, we have to use it in same ARM template.
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {
        "subscriptionId":"[subscription().subscriptionId]"
    },
    "resources": [
        {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "dbDeployment",
      "properties": {
       "mode": "Incremental",
       "expressionEvaluationOptions": {
        "scope": "inner"
      },
      "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          "SQLLogin": {
            "type": "string",
            "metadata": {
              "description": "The administrator user name for the Azure SQL Server"
            }
          },
          "SQLPassword": {
            "type": "secureString",
            "metadata": {
              "description": "The administrator password for the Azure SQL Server"
            }
          }
    },
   "resources": [
            {
            "name": "sqlserver600505",
            "type": "Microsoft.Sql/servers",
            "apiVersion": "2021-08-01-preview",
            "location": "[resourceGroup().location]",
            "properties": {
                "administratorLogin": "[parameters('SQLLogin')]",
                "administratorLoginPassword": "[parameters('SQLPassword')]"
              }
        },
        {
            "name": "[format('{0}/{1}', 'sqlserver600505', 'appdb')]",
            "type": "Microsoft.Sql/servers/databases",
            "apiVersion": "2021-08-01-preview",
            "location": "[resourceGroup().location]",            
            "dependsOn": [
                "[resourceId('Microsoft.Sql/servers', 'sqlserver600505')]"
            ],
            "sku": {
                "name": "Basic",
                "tier": "Basic"
              },
              "properties": {}
        }
    ],
    "outputs": {}},
    "parameters": {
          "SQLLogin": {
            "value": "sqlusr"
          },
          "SQLPassword": {
            "reference": {
              "keyVault": {
                "id": "[resourceId(variables('subscriptionId'), 'template-grp', 'Microsoft.KeyVault/vaults', 'keyvault67767')]"
              },
              "secretName": "dbpassword"
            }
          }
    }}}]
}
```

## OWASP Tool
This can be used with Release pipeline to scan fort security risks.

## GitHub Code Scanning
GitHUb also has a code scanning tool to detect vulnerabilities. For that go to Actions -> Choose a workflow and search for Security, and CodeQL Analysis.

#  Monitor Service (Just search Monitor in Azure dashboard)
- You can add alerts, see metrics, activity logs etc. If some resource deployment fails, you can see the error message details in Monitor activity log.
- Alerts can be added for a resource based on activity, metrics (CPU utlization etc) or log (particular keyword or string in log)
- For alerts, you can set notification on email or SMS
- You can also set an action such as running an Azure Funtion or send data to Event Hub etc.

## Log Analytics Workspace
This is where you can store logs from VMs, on prem resources, database etc. Advantage is that you can use a query lang to search the data. To use it create a resource of type "Log Analytics Workspace".
- You can then go to "Workspace data sources" -> Virtual Machines... It displays available VMs. Click on it and hit "Connect"
- This will apply an extension "Loag analytics agent" to VM and that extension will collect the logs from VM and send here
- Under Settings -> Agent Management, you can create a data collection rule. there you can define data sources such as Event Logs or Performance Counters and from them what to collect
- You can also go to SQL server resouce, click on Monitoring -> Diagnbotic Settings and from there redirect the logs to Log Analytics Workspace.

## Service Map
Can show the services running on a machine and which ports are open and where the connections connecting to. Aslo who are the clients connected to it (which IP and ports).

## Applications Insights
Provides monitioring of web resources, their performances etc. You can enable it while creating a Web App resource and then add it to your web app project in .NET. VS automatically embeds it with few clicks. You can then see the Live Metrics of your app too under application insights.

Follow language-specific guidelines to enable Live Metrics:<br>
https://learn.microsoft.com/en-us/azure/azure-monitor/app/live-stream?tabs=dotnet6#get-started
- ASP.NET: Live Metrics is enabled by default.
- ASP.NET Core: Live Metrics is enabled by default.
- .NET/.NET Core Console/Worker: Live Metrics is enabled by default.
- .NET Applications: Enable using code.
- Java: Live Metrics is enabled by default.
- Node.js









