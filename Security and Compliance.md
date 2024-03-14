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









