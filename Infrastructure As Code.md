So far we have used Azure Portal to create/manage resouces but we can use code to accomplish that.<br>
<br>
Tasks usually involved:
- Creating your infrastructure
- Making changes (such as change size of VM etc)
- Replicating same infrastructure across different env (test, staging, prod etc)

Developing your infra as code (IaC) helps that. You can define your infr, replicate across multiple env, make changes to it easily and version control the changes.

There are various tools to do that such as:
- ARM templates
- Bicep
- Terraform
- Azure CLI/ PowerShell script

Earlier we built the infrastructre using Azure Portal and then used it. Now we will do it via ARM templates. Following is a basic syntax of ARM template from msdn<br>
https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/syntax
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "languageVersion": "",
  "contentVersion": "",
  "apiProfile": "",
  "definitions": { },
  "parameters": { },
  "variables": { },
  "functions": [ ],
  "resources": [ ], /* or "resources": { } with languageVersion 2.0 */
  "outputs": { }
}
```

Note: Bicep is a new language that offers the same capabilities as ARM templates but with a syntax that's easier to use. If you're considering infrastructure as code options, we recommend looking at Bicep.

Note: You can install Azure Resource Manager (ARM) extension in VS Code to build ARM templates.

ARM templates can be downloaded based on resoource type from<br>
https://learn.microsoft.com/en-us/azure/templates/microsoft.web/sites?pivots=deployment-language-arm-template<br>
You then work on them as the starting point

Once template is ready, here is how to deploy.
- Create a resource group say template-grp
- Create a resource, search "template" and select "Template Deployment"
- Click on "Build your own template in editor"
- Copy your ARM template and click on "Save"
- Select resource group template-grp and hit "Create"
- It will create resources (app service plan and app service in this case)

### ARM template for Azure WebApp
Below is the ARM template
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Web/serverfarms",
            "apiVersion": "2022-03-01",
            "name": "plan787878",
            "location": "[resourceGroup().location]",
            "sku": {
                "name":"F1",
                "capacity":1
            },
            "properties":{
                "name":"plan787878"
            }
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2022-03-01",
            "name": "newapp994848vk",
            "location": "[resourceGroup().location]",
            "properties":{
                "name":"newapp994848vk",
                "serverFarmId":"[resourceId('Microsoft.Web/serverfarms','plan787878')]"
            },
            "dependsOn":[
                "[resourceId('Microsoft.Web/serverfarms','plan787878')]"
            ]
        }
    ],
    "outputs": {}
}
```

### ARM template for Azure SQL DB (sqlserver400505vk/appdb)
2 resources are needed - SQL DB Server (sqlserver400505vk) to host the DB and DB (appdb) itself.<br>
Follow same process as before and use the template below
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "SQLLogin": {
            "type":"string",
            "metadata":{
                "description":"The administrator user name"
            }
        },
        "SQLPassword": {
            "type":"secureString",
            "metadata":{
                "description":"The administrator password"
            }

        }
    },
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Sql/servers",
            "apiVersion": "2022-02-01-preview",
            "name": "sqlserver400505vk",
            "location": "[resourceGroup().location]",
            "properties":{
                "administratorLogin": "[parameters('SQLLogin')]",
                "administratorLoginPassword": "[parameters('SQLPassword')]"
            }
        },
        {
               "type": "Microsoft.Sql/servers/databases",
               "apiVersion": "2022-02-01-preview",
               "name": "[format('{0}/{1}','sqlserver400505vk','appdb')]",
               "location": "[resourceGroup().location]",
               "sku":{
                "name":"Basic",
                "tier":"Basic"
               },
               "properties":{},
                "dependsOn":[
                    "[resourceId('Microsoft.Sql/servers','sqlserver400505vk')]"
                ]
               
        }
    ],
    "outputs": {}
}
```

### ARM template for Azure VM
A VM consists of many other resources such as disk, ip, network interface, network security group etc. All these needs to be defined in ARM template for VM creation.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "appnetwork",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2020-11-01",
            "location": "North Europe",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.0.0.0/16"
                    ]
                },
                "subnets": [
                    {
                        "name": "SubnetA",
                        "properties": {
                            "addressPrefix": "10.0.0.0/24",
                            "networkSecurityGroup": {
                                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', 'app-nsg')]"
                              }
                        }
                    },
                    {
                        "name": "SubnetB",
                        "properties": {
                            "addressPrefix": "10.0.1.0/24"
                        }
                    }
                ]
            }
        },
        {
            "name": "app-ip",
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2020-11-01",
            "location": "North Europe",
            "properties": {
                "publicIPAllocationMethod": "Dynamic"
                }
            },
            {
                "name": "app-interface",
                "type": "Microsoft.Network/networkInterfaces",
                "apiVersion": "2020-11-01",
                "location": "North Europe",            
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipConfig1",
                            "properties": {
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'app-ip')]"
                                  },
                                "subnet": {
                                    "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'appnetwork', 'SubnetA')]"
                                }
                            }
                        }
                    ]
                },
                "dependsOn": [
                    "[resourceId('Microsoft.Network/publicIPAddresses', 'app-ip')]",
                    "[resourceId('Microsoft.Network/virtualNetworks', 'appnetwork')]"
                ]
            },        
        {
            "name": "vmstore8677676",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2021-04-01",
            "location": "[resourceGroup().location]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2"
        },
        {
            "name": "app-nsg",
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-11-01",
            "location": "[resourceGroup().location]",
            "properties": {
                "securityRules": [
                    {
                        "name": "Allow-RDP",
                        "properties": {
                            "description": "description",
                            "protocol": "Tcp",
                            "sourcePortRange": "*",
                            "destinationPortRange": "3389",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*",
                            "access": "Allow",
                            "priority": 100,
                            "direction": "Inbound"
                        }
                    }
                ]
            }
        },
        {
            "name": "appvm",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2021-03-01",
            "location": "[resourceGroup().location]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', toLower('vmstore8677676vk'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "Standard_D2s_v3"
                },
                "osProfile": {
                    "computerName": "appvm",
                    "adminUsername": "demousr",
                    "adminPassword": "Azure@123"
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "2019-Datacenter",
                        "version": "latest"
                    },
                    "osDisk": {
                        "name": "windowsVM1OSDisk",
                        "caching": "ReadWrite",
                        "createOption": "FromImage"
                    },
                    "dataDisks": [
                        {
                            "name":"vm-data-disk",
                            "diskSizeGB":16,
                            "createOption": "Empty",
                            "lun":0
                        }
                    ]
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', 'app-interface')]"
                        }
                    ]
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true,
                        "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', toLower('vmstore8677676'))).primaryEndpoints.blob]"
                    }
                }
            }
        }
    ],
    "outputs": {}
}
```

### Nested and Linked Templates
In Nested one can define the template in parent itself and link it to other resources.<br>
In Link, we can define the resource templates in another file and reference it in others. That way we can resue the templates.

### Incremental vs Complete mode
In Incremental resources can be added in a resource group and existing ones in the RG wont be affected. In complete, the existing resources will be edeleted and only those that are in template will exist. nested/linked works only in incremental mode.

#### Nested
Here is a Nested Template that creates both webapp and a sql serer + DB.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "SQLLogin": {
            "type":"string",
            "metadata":{
                "description":"The administrator user name"
            }
        },
        "SQLPassword": {
            "type":"secureString",
            "metadata":{
                "description":"The administrator password"
            }

        }
    },
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Web/serverfarms",
            "apiVersion": "2022-03-01",
            "name": "plan787878",
            "location": "[resourceGroup().location]",
            "sku": {
                "name":"F1",
                "capacity":1
            },
            "properties":{
                "name":"plan787878"
            }
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2022-03-01",
            "name": "newapp994848vk",
            "location": "[resourceGroup().location]",
            "properties":{
                "name":"newapp994848vk",
                "serverFarmId":"[resourceId('Microsoft.Web/serverfarms','plan787878')]"
            },
            "dependsOn":[
                "[resourceId('Microsoft.Web/serverfarms','plan787878')]"
            ]
        },
        {
            "type":"Microsoft.Resources/deployments",
            "apiVersion":"2021-04-01",
            "name":"childTemplate",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "resources": [
        {
            "type": "Microsoft.Sql/servers",
            "apiVersion": "2022-02-01-preview",
            "name": "sqlserver500505vk",
            "location": "[resourceGroup().location]",
            "properties":{
                "administratorLogin": "[parameters('SQLLogin')]",
                "administratorLoginPassword": "[parameters('SQLPassword')]"
            }
        },
        {
               "type": "Microsoft.Sql/servers/databases",
               "apiVersion": "2022-02-01-preview",
               "name": "[format('{0}/{1}','sqlserver500505vk','appdb')]",
               "location": "[resourceGroup().location]",
               "sku":{
                "name":"Basic",
                "tier":"Basic"
               },
               "properties":{},
                "dependsOn":[
                    "[resourceId('Microsoft.Sql/servers','sqlserver500505vk')]"
                ]
               
        }
    ]
    
                }
            }

        }
    ],
    "outputs": {}
}
```

#### Linked
We will create two separate ARM template files - sqldatabase.json and sqldatabase-parameters.json and link them in main file main.json. The parameters json has sql user and pwd for the sql database.
- We need to create a "Storage Account" resource to store our templates in azure. Lets give it name "storage2024vk".<br>
- Click on new Storage Account. Go to Data Storage -> Containers and create a container named "templates".
- Upload sqldatabase.json in templates and note down the URL and update in main.json (https://storage2024vk.blob.core.windows.net/templates/sqldatabase.json)
- Upload sqldatabase-parameters.json and note down the URL
- Update the URLs in main.json
- Create the resource using template as in previous sections, this time just providing main.json as the template body.

main.json
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {        
    },
    "functions": [],
    "variables": {},
    "resources": [
        {
            "name": "plan787878",
            "type": "Microsoft.Web/serverfarms",
            "apiVersion": "2020-12-01",
            "location": "[resourceGroup().location]",
            "sku": {
                "name": "F1",
                "capacity": 1
            },
            "properties": {
                "name": "plan787878"
            }
        },
        {
            "name": "newapp99484811",
            "type": "Microsoft.Web/sites",
            "apiVersion": "2020-12-01",
            "location": "[resourceGroup().location]",            
            "dependsOn": [
                "[resourceId('Microsoft.Web/serverfarms', 'plan787878')]"
            ],
            "properties": {
                "name": "newapp99484811",
                "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', 'plan787878')]"
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2021-04-01",
            "name": "LinkedTemplate",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "https://storage2024vk.blob.core.windows.net/templates/sqldatabase.json",
                    "contentVersion": "1.0.0.0"
                },
                "parametersLink": {
                    "uri": "https://storage2024vk.blob.core.windows.net/templates/sqldatabase-parameters.json",
                    "contentVersion": "1.0.0.0"
                }
    }}
    ],
    "outputs": {}
}
```

sqldatabase.json
```json
{
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
    "functions": [],
    "variables": { },
    "resources": [
        {
            "name": "sqlserver8005051",
            "type": "Microsoft.Sql/servers",
            "apiVersion": "2021-08-01-preview",
            "location": "[resourceGroup().location]",
            "properties": {
                "administratorLogin": "[parameters('SQLLogin')]",
                "administratorLoginPassword": "[parameters('SQLPassword')]"
              }
        },
        {
            "name": "[format('{0}/{1}', 'sqlserver8005051', 'appdb')]",
            "type": "Microsoft.Sql/servers/databases",
            "apiVersion": "2021-08-01-preview",
            "location": "[resourceGroup().location]",            
            "dependsOn": [
                "[resourceId('Microsoft.Sql/servers', 'sqlserver8005051')]"
            ],
            "sku": {
                "name": "Basic",
                "tier": "Basic"
              },
              "properties": {}
        }
    ],
    "outputs": {}
}
```

sqldatabase-parameters.json
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "SQLLogin": {
      "value": "sqlusr"
    },
    "SQLPassword": {
      "value": "Azure@123"
    }
  }
}
```

## Using ARM templates in Release pipelines.
ARM templates can be used to create resources via Release pipeline and then used for deployment in subsequent steps.

Lets add main.json too in the storage account.

- In the Release pipeline where we create Agent Jobs, add an agent job of type "ARM template deployment"
- Make it the first task as we want to use the resources created via ARM in our release pipeline.
- In the config screen, provide the URL of main.json
- You can  also pass on the values from your ARM template such as VM name, SQL Server name etc in subsequent Release pipeline tasks
- Instead of storing ARM template in Azure Storage Account and using URL, it can be stored with code base (in Templates folder) and artifact can be used in pipeline. Update azure-pipeline.yml as
  ```yml
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.SourcesDirectory)/webapp/Templates' 
    artifactName: 'template-artifact'
  ```

Note: If you run your pipeline again, it will see that resources are already available and wont recreate them.

## Cleanup of Resources created via Release pipelines after we are done
Suppose we created the resources using ARM templates using Release pipeline. Once testing is done and we move to next stage (say prod staging), we can add an stage after Testing stage to do resource cleanup. For that simply add Agent Job of type "Azure CLI" and use Power Shell inline script for cleanup. Here is an example script.

```bash
az resource delete -g template-grp -n "sqlserver700505/appdb" --resource-type "Microsoft.Sql/servers/databases"
az resource delete -g template-grp -n "sqlserver700505" --resource-type "Microsoft.Sql/servers"
az resource delete -g template-grp -n newapp994848 --resource-type "Microsoft.Web/sites"
az resource delete -g template-grp -n plan787878 --resource-type "Microsoft.Web/serverfarms"
```

## Dynamic Resource creation (resources with dynamic names)
So far we have given the resource names such as SQL Server name etc in the ARM template itself. Now we will see how to do that dynamically. For example, we want to dynamically generate a SQL Server name and then use it in subsequent steps in Release pipeline.<br>
This is done in ARM itself using "variables" as in below. Here we defined a new variable named sql-server-name and used in ARM template. Also notice the "output" section, it is used to output the variable name and that output is used by other action tasks in Release pipelines. For ex, using the SQL server name and then creating a DB in it or running q SQL script.
sqldatabase.json
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "SQLLogin": {
            "type":"string",
            "metadata":{
                "description":"The administrator user name"
            }
        },
        "SQLPassword": {
            "type":"secureString",
            "metadata":{
                "description":"The administrator password"
            }

        }
    },
    "functions": [],
    "variables": {
        "sql-server-name":"[concat('server',uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "type": "Microsoft.Sql/servers",
            "apiVersion": "2022-02-01-preview",
            "name": "[variables('sql-server-name')]",
            "location": "[resourceGroup().location]",
            "properties":{
                "administratorLogin": "[parameters('SQLLogin')]",
                "administratorLoginPassword": "[parameters('SQLPassword')]"
            }
        },
        {
               "type": "Microsoft.Sql/servers/databases",
               "apiVersion": "2022-02-01-preview",
               "name": "[format('{0}/{1}',variables('sql-server-name'),'appdb')]",
               "location": "[resourceGroup().location]",
               "sku":{
                "name":"Basic",
                "tier":"Basic"
               },
               "properties":{},
                "dependsOn":[
                    "[resourceId('Microsoft.Sql/servers',variables('sql-server-name'))]"
                ]
               
        }
    ],
    "outputs": {
        "sqlserverfqdn": {
            "type": "string",
            "value":"[reference(variables('sql-server-name')).fullyQualifiedDomainName]"
        }
    }
}
```

main.json
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {
        "web-app-name":"[concat('webapp',uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "type": "Microsoft.Web/serverfarms",
            "apiVersion": "2022-03-01",
            "name": "plan787878",
            "location": "[resourceGroup().location]",
            "sku": {
                "name":"F1",
                "capacity":1
            },
            "properties":{
                "name":"plan787878"
            }
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2022-03-01",
            "name": "[variables('web-app-name')]",
            "location": "[resourceGroup().location]",
            "properties":{
                "name":"[variables('web-app-name')]",
                "serverFarmId":"[resourceId('Microsoft.Web/serverfarms','plan787878')]"
            },
            "dependsOn":[
                "[resourceId('Microsoft.Web/serverfarms','plan787878')]"
            ]
        },
        {
            "type":"Microsoft.Resources/deployments",
            "apiVersion": "2021-04-01",
            "name":"LinkedTemplate",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "https://templatestore465656.blob.core.windows.net/templates/sqldatabase.json",
                    "contentVersion":"1.0.0.0"
                },
                "parametersLink": {
                    "uri": "https://templatestore465656.blob.core.windows.net/templates/sqldatabase-parameters.json",
                    "contentVersion":"1.0.0.0"
                }
            }
        }
    ],
    "outputs": {
        "webappname": {
            "type": "string",
            "value":"[variables('web-app-name')]"
        },
        "sqlserverfqdn": {
            "type": "string",
            "value":"[reference('LinkedTemplate').outputs.sqlserverfqdn.value]"
        }
    }
}
```

You can then test these templates by directly deploying them in azure and see if everything works and Deployment -> Outputs has the output data values.

Once our templates are ready, lets see how to use these outputs in Release pipeline:

In Release Pipeline, next to Task, there is a tab named "Variables", we will use that. In our main.json above we are exposing 2 output variables webappname and sqlserverfqdn. We need to add these in Variables tab and next to them check mark the "Settable are Release time". These values will be populated using a PS script.

Now go back to task "ARM Template Deployment" and in the bottm part of config screen, you will see "Deployment Outputs" section, give it a name say "resourcesdeployed". This will have the entire "outputs" json which we will process using following PS script and set the variable names we created previously. This file could be added to codebase under Templates folder.

script.ps1
```bash
param (
    [Parameter(Mandatory=$true)][string]$ARMValues
)

Write-Host "Starting the script"

$outputs = $ARMValues | convertfrom-json

foreach ($output in $outputs) {
$value1=$output.'webappname'.value
$value2=$output.'sqlserverfqdn'.value

Write-Host $value1
Write-Host $value2

Write-Host "##vso[task.setvariable variable=webappname;]$value1"
Write-Host "##vso[task.setvariable variable=sqlserverfqdn;]$value2"
}
```

After the ARM Deployment task, add another task of type "PowerShell Script", this will run our PS script. Specify the path from artifacts and in the Arguments text box, type: -ARMValues '$(resourcesdeployed)'. What we are doing is taking the output of ARM deployment and processing it via PS script, the PS Script is then setting the variable values in Release pipeline for use by subsequent tasks in the pipeline such as DB creation or running SQL scripts. In subsequent tasks you can simply use $(webappname) and $(sqlserverfqdn) to access the dynamic values. You can also add a "Bash shell" task and output the variable names from that script using echo command.

## Creating resources using Az CLI
In Azure portal, on top bar there is an icon for Azure Shell, clicking that opens Power Shell in lower half of  browser and you can use that to run CLI commands.<br>
Following CLI commands can be run there to create the resources we created earlier using ARM template.

```bash
az appservice plan create -g template-grp -n plan787878 --sku F1 --l "North Europe"
az webapp create -g template-grp -p plan787878 -n newapp1094848 --runtime "dotnet:6"

az webapp list-runtimes

az sql server create --name sqlserver420505vkcli --resource-group template-grp --location "North Europe" --admin-user sqlusr --admin-password Azure@123
az sql db create --resource-group template-grp --server sqlserver420505 --name appdb --service-objective "Basic"
```

## Using CLI to create resources in Release Pipeline
Instead of using ARM Deployment task in pipeline, use "PowerShell script" task and copy paste the above CLI commands inline there. That will create the resources using CLI via Release pipeline.

# TERRAFORM (By HashiCorp)
This is similar to ARM templates but open source and can be used on other cloud providers as well, code is human readable too. The extesnion of file is .tf, main.tf.

How to work with TerraForm (TF):
- download terraform tool which is a simple exe in a folder (https://developer.hashicorp.com/terraform/install)
- install Visual Studio extensions - Azure Terraform and Hashicorp Terraform

Below is the basic structure needed. The required_providers tells which provider? Google, Azure, AWS etc... and then we have block of credentials.
```
terraform {
  required_providers {
    azurerm={
        source="hashicorp/azurerm"
        version="3.17.0"
    }
  }
}

provider "azurerm" {
  subscription_id = "f23a81dd-b03a-4625-a27d-11e4f12844f5"
  tenant_id = "1a738633-1476-4d96-828c-fa5727a31ca7"
  client_id = "32db6859-28ab-4d09-8fc1-91678060f9b7"
  client_secret = "Qth8Q~HI6bAeeQ0p_1-URbbRSIsjrqHQbAnW~arj"
  features {    
  }
}
```
To fill the credentials, go to Azure Portal -> Microsoft Entra ID. There click on Manage -> App Registrations. Click on "New Registration", fill application name as "terraform" and hit "Register". It will show you the info that you can use to fill the above. You can also generate the client_secret there. For subscription_id, go to your subscriptions and copy the id from there. This will help TF to connect to your azure account. Next thing you need to do is go to your subscriptions. Click on subscription -> Access Control (IAM) -> Add -> Add role assignment. Select "Contribute" access, click on Members -> Select members and look for "terraform", select it and do a Review & assign of the role.

The next step is to define resource blocks. Here is how a complete tf file looks like:
```
terraform {
  required_providers {
    azurerm={
        source="hashicorp/azurerm"
        version="3.17.0"
    }
  }
}

provider "azurerm" {
  subscription_id = "6912d7a0-bc28-459a-9407-33bbba641c07"
  tenant_id = "70c0f6d9-7f3b-4425-a6b6-09b47643ec58"
  client_id = "3feda701-6a3d-4915-8d26-343827060a8e"
  client_secret = "kKH8Q~54-LwKs2lYfNj6ECD_VmAt-cKSllRG4bfE"
  features {    
  }
}

resource "azurerm_service_plan" "plan787878" {
  name                = "plan787878"
  resource_group_name = "template-grp"
  location            = "North Europe"
  os_type             = "Windows"
  sku_name            = "F1"
}

resource "azurerm_windows_web_app" "newapp1002030" {
  name                = "newapp1002030"
  resource_group_name = "template-grp"
  location            = "North Europe"
  service_plan_id     = azurerm_service_plan.plan787878.id

  site_config {
    always_on = false
    application_stack{
        current_stack="dotnet"
        dotnet_version="v6.0"
    }
  }

  depends_on = [
    azurerm_service_plan.plan787878
  ]
}

resource "azurerm_mssql_server" "sqlserver468985656" {
  name                         = "sqlserver468985656"
  resource_group_name          = "template-grp"
  location                     = "North Europe"
  version                      = "12.0"
  administrator_login          = "sqlusr"
  administrator_login_password = "Azure@123"  
}

resource "azurerm_mssql_database" "appdb" {
  name           = "appdb"
  server_id      = azurerm_mssql_server.sqlserver468985656.id
  collation      = "SQL_Latin1_General_CP1_CI_AS"
  license_type   = "LicenseIncluded"
  max_size_gb    = 2  
  sku_name       = "Basic"
  depends_on = [
    azurerm_mssql_server.sqlserver468985656
  ]
}
```

Step 1: To deploy the resources go to dir where terraform exe is installed on your machine in console and do a "terraform init". This will download the necessary plugin etc.<br>
Step 2: create a plan using [terraform plan -out main.tfplan]<br>
Step 3: Apply the plan generated in step 2 and this will create the resources in Azure - [terraform apply "main.tfplan"]<br>

If you do a "terraform destroy", it will delete the resources created previously.

## Terraform in Release pipeline
To do that add main.tf to your codebase say under Templates folder. Pipeline can then access this artifact and run it.<br>
- In Release pipeline -> Agent Job... search for Terraform and install the extension if needed.
- Add "Install Terraform latest" from the search results as agent job so it can install the terraform tools
- Add "Terraform" task as next one for terraform init" (step 1)
- In the config screen, fill display as "Terraform: init", "azurerm" as Provider, "init" in the Command, in the Config directory select the "templates" artifact folder.
- This will also need a "Storage Account" for terraform to store the state, create one and select it in config screen
- Fill "terraform.tfstate" as the Key and done
- Add one more "Terraform" task for terraform plan" (step 2)
- Fill "plan" as command and rest as in previous step.
- Add one more "Terraform" task for terraform apply" (step 3)
- Choose "apply" as command. In the Additional command arguments, fill "-auto-approve" and done

Run the pipeline and it will create the resources and use them in subsequent steps.

# Using azure-pipeline.yaml file itself for deployment
If you go to Release pipeline, select a task there, you see a "View YAML" link on top-right corner, you can click and copy that YAML code and paste it in azure-pipeline.yaml that we used for build. Here is how a sample yaml that will do everything from build to release looks like

```yaml
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- main

resources:
- repo: self

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    steps:
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

    - task: DotNetCoreCLI@2
      inputs:
        command: publish
        publishWebProjects: True
        arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
        zipAfterPublish: True

# We still need to take the staging directory files and publish it as a build artifact

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.ArtifactStagingDirectory)' 
        artifactName: 'sqlapp-secure-artifact'

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.SourcesDirectory)/sqlapp/Templates' 
        artifactName: 'template-artifact'

- stage: Deploy
  displayName: Deployment stage
  jobs:
  - job: Deploy
    displayName: Deploy
    steps:

    - download: current
      artifact: sqlapp-secure-artifact

    - task: AzureCLI@2
      displayName: 'Azure CLI - Azure Web App'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
         az appservice plan create -g template-grp -n plan787878 --sku F1 --l "North Europe"
     
         az webapp create -g template-grp -p plan787878 -n newapp3094848 --runtime "dotnet:6"
    
    - task: AzureCLI@2
      displayName: 'Azure CLI - Azure SQL Database'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
         az sql server create --name sqlserver4205058 --resource-group template-grp --location "North Europe" --admin-user sqlusr --admin-password Azure@123
     
         az sql db create --resource-group template-grp --server sqlserver4205058 --name appdb --service-objective "Basic"
    
    - task: SqlAzureDacpacDeployment@1
      displayName: 'Azure SQL Table creation'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        ServerName: sqlserver4205058.database.windows.net
        DatabaseName: appdb
        SqlUsername: sqlusr
        SqlPassword: 'Azure@123'
        deployType: InlineSqlTask
        SqlInline: |
         IF (NOT EXISTS (SELECT * 
                      FROM INFORMATION_SCHEMA.TABLES 
                      WHERE TABLE_SCHEMA = 'dbo' 
                      AND  TABLE_NAME = 'Products'))
         BEGIN
     
         CREATE TABLE Products
         (
          ProductID int,
          ProductName varchar(1000),
          Quantity int
         )
     
         INSERT INTO Products(ProductID,ProductName,Quantity) VALUES (1,'Mobile',100)
     
         INSERT INTO Products(ProductID,ProductName,Quantity) VALUES (2,'Laptop',200)
     
         INSERT INTO Products(ProductID,ProductName,Quantity) VALUES (3,'Tabs',300)
     
         END
        IpDetectionMethod: IPAddressRange
        StartIpAddress: 0.0.0.0
        EndIpAddress: 0.0.0.0
        DeleteFirewallRule: false
    
    - task: AzureWebApp@1
      displayName: 'Azure Web App Deploy: newapp3094848'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        appType: webApp
        appName: newapp3094848
        Package: '$(Pipeline.Workspace)/sqlapp-secure-artifact/**/*.zip'
    
    - task: AzureAppServiceSettings@1
      displayName: 'Azure App Service Settings: newapp3094848'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        appName: newapp3094848
        resourceGroupName: 'template-grp'
        connectionStrings: |
          [
            {
              "name": "SQLConnection",
              "value": "Server=tcp:sqlserver4205058.database.windows.net,1433;Initial Catalog=appdb;Persist Security Info=False;User ID=sqlusr;Password=Azure@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;",
              "type": "SQLAzure",
              "slotSetting": false
            }
          ]
```

# Using azure-pipeline.yaml with ARM templates for deployment
Very similar to what we did in previous step. Here is how a file will look like
```yaml
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- main

pool:
  vmImage: 'windows-latest'

resources:
- repo: self

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    steps:
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

    - task: DotNetCoreCLI@2
      inputs:
        command: publish
        publishWebProjects: True
        arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
        zipAfterPublish: True

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.ArtifactStagingDirectory)' 
        artifactName: 'sqlapp-artifact'

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.SourcesDirectory)/sqlapp/Templates' 
        artifactName: 'template-artifact'

- stage: Deploy
  displayName: Deployment stage
  jobs:
  - job: Deploy
    displayName: Deploy
    steps:

    - download: current
      artifact: sqlapp-artifact

    - download: current
      artifact: template-artifact
      
    - pwsh: |       
       Get-ChildItem -Path $(Pipeline.Workspace)\*.* -Recurse -Force | Out-String -Width 180
      errorActionPreference: continue
      displayName: 'List content'
      continueOnError: true

    - task: AzureResourceManagerTemplateDeployment@3
      displayName: 'ARM Template deployment: Resource Group scope'
      inputs:
        azureResourceManagerConnection: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        subscriptionId: '6912d7a0-bc28-459a-9407-33bbba641c07'
        resourceGroupName: 'template-grp'
        location: 'North Europe'
        csmFile: '$(Pipeline.Workspace)/template-artifact/main.json'
        action: 'Create Or Update Resource Group'
        deploymentMode: 'Incremental'        

    - task: SqlAzureDacpacDeployment@1
      displayName: 'Azure SQL InlineSqlTask'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        ServerName: sqlserver8000505.database.windows.net
        DatabaseName: appdb
        SqlUsername: sqlusr
        SqlPassword: 'Azure@123'
        deployType: InlineSqlTask
        SqlInline: |
         IF (NOT EXISTS (SELECT * 
                      FROM INFORMATION_SCHEMA.TABLES 
                      WHERE TABLE_SCHEMA = 'dbo' 
                      AND  TABLE_NAME = 'Products'))
         BEGIN
     
         CREATE TABLE Products
         (
          ProductID int,
          ProductName varchar(1000),
          Quantity int
          )
     
     
          INSERT INTO Products(ProductID,ProductName,Quantity) VALUES (1,'Mobile',100)
     
          INSERT INTO Products(ProductID,ProductName,Quantity) VALUES (2,'Laptop',200)
     
          INSERT INTO Products(ProductID,ProductName,Quantity) VALUES (3,'Tabs',300)
     
          END
        IpDetectionMethod: IPAddressRange
        StartIpAddress: 0.0.0.0
        EndIpAddress: 0.0.0.0
        DeleteFirewallRule: false
    - task: AzureRmWebAppDeployment@4
      displayName: 'Azure App Service Deploy'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        WebAppName: newapp1094848
        Package: '$(Pipeline.Workspace)/sqlapp-artifact/**/*.zip'
    
    - task: AzureAppServiceSettings@1
      displayName: 'Azure App Service Settings: newapp1094848'
      inputs:
        azureSubscription: 'Azure subscription 1 (6912d7a0-bc28-459a-9407-33bbba641c07)'
        appName: newapp1094848
        resourceGroupName: 'template-grp'
        connectionStrings: |
          [
            {
              "name": "SQLConnection",
              "value": "Server=tcp:sqlserver8000505.database.windows.net,1433;Initial Catalog=appdb;Persist Security Info=False;User ID=sqlusr;Password=Azure@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;",
              "type": "SQLAzure",
              "slotSetting": false
            }
          ]
```

# Azure Automation DSC (Desired Service Configuration) [https://learn.microsoft.com/en-us/powershell/dsc/reference/resources/windows/fileresource?view=dsc-1.1]
Ensures that a VM has required software/state such as IIS on Windows VM that will be hosting web applications.
- You can do so by creating a resource of type "Automation"
- Suppose we create a new Windows 2019 DataCenter server VM (newly created wont have IIS installed)
- We can do so via a PS script config file. Here is a sample script.ps1
  ```
  configuration NewConfig
  {
      Node localhost
      {
          WindowsFeature IIS
          {
              Ensure               = 'Present'
              Name                 = 'Web-Server'
              IncludeAllSubFeature = $true
          }
      }
  }
  ```
- This basically is telling that the desired role of "Web-Server" should always be present.
- Now go back to "Automation" resource we created, click on "State configuration (DSC)"
- Go to Configurations Tab and click on "Add" and upload the config file script.ps1. Name it as "NewConfig" as in ps1 file.
- After upload, the next step is to compile it gy clicking on "Compile".
- You can then see it in "Compiled configurations". Click on that.
- There you can add your nodes or machines to it.
- Add (Onboard) the Windows 2019 VM we created previously.
- You can specify refresh frequency (say 30 mins). The DSC will then check it every 30 mins.
- This will ensure that IIS is always present, if missing, it will be installed.
- If you go to VM itself, under Extensions + applications, you can see the DSCX Powershell entry there.
- As a test, you can login to machine and remove IIS forcefully
- After some time DSCX will again install IIS on that

# Custom Script Extension
You can create a custom script, that can be used to download and run on Azure VMs. These can be used to automate the post deployment tasks as well.

# VM Scale Sets
VMs can automatically scale up or down based on the load. For the exercise, you will need to create a Load Balancer, then create a VM scale set with initial capacity of 2. Here is the ARM template to deploy the VM scale set.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2021-05-01",
            "name": "app-balancer",
            "location": "[resourceGroup().location]",
            "properties": {
              "frontendIPConfigurations": [
                {
                  "name": "LoadBalancerFrontEnd",
                  "properties": {
                    "publicIPAddress": {
                      "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'app-ip')]"
                    }
                  }
                }
              ],
              "backendAddressPools": [
                {
                  "name": "app-pool"
                }
              ],
              "loadBalancingRules": [
                {
                  "name": "RuleA",
                  "properties": {
                    "frontendIPConfiguration": {
                      "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'app-balancer', 'loadBalancerFrontEnd')]"
                    },
                    "backendAddressPool": {
                      "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'app-balancer', 'app-pool')]"
                    },
                    "protocol": "Tcp",
                    "frontendPort": 80,
                    "backendPort": 80,
                    "enableFloatingIP": false,
                    "idleTimeoutInMinutes": 5,
                    "probe": {
                      "id": "[resourceId('Microsoft.Network/loadBalancers/probes', 'app-balancer', 'tcpProbe')]"
                    }
                  }
                }
              ],
              "probes": [
                {
                  "name": "tcpProbe",
                  "properties": {
                    "protocol": "Tcp",
                    "port": 80,
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2
                  }
                }
              ]},
            "dependsOn": [
              "[resourceId('Microsoft.Network/publicIPAddresses', 'app-ip')]"
            ]
          },
          {
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "apiVersion": "2021-11-01",
            "name": "app-set",
            "location": "[resourceGroup().location]",
            "sku": {
              "name": "Standard_D2s_v3",
              "tier": "Standard",
              "capacity": 2
            },
            "properties": {
              "overprovision": true,
              "upgradePolicy": {
                "mode": "Automatic"
              },
              "singlePlacementGroup": true,
              "platformFaultDomainCount": 3,
              "virtualMachineProfile": {
                "storageProfile": {
                  "osDisk": {
                    "caching": "ReadWrite",
                    "createOption": "FromImage"
                  },
                  "imageReference": {
                    "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "2019-Datacenter",
                        "version": "latest"                    
                  }
                },
                "osProfile": {
                  "computerNamePrefix": "appvm",
                  "adminUsername": "demousr",
                  "adminPassword": "Azure@123"
                },
                "networkProfile": {
                  "networkInterfaceConfigurations": [
                    {
                      "name": "app-nic",
                      "properties": {
                        "primary": true,
                        "ipConfigurations": [
                          {
                            "name": "ipConfig",
                            "properties": {
                              "subnet": {
                                "id": "[reference(resourceId('Microsoft.Network/virtualNetworks', 'app-network')).subnets[0].id]"
                              },
                              "loadBalancerBackendAddressPools": [
                                {
                                  "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'app-balancer', 'app-pool')]"
                                }
                              ]
                            }
                          }
                        ]
                      }
                    }
                  ]
                }}},
                "dependsOn": [
                    "[resourceId('Microsoft.Network/virtualNetworks', 'app-network')]",
                    "[resourceId('Microsoft.Network/loadBalancers', 'app-balancer')]"
                  ]
                },
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2021-05-01",
                    "name": "app-network",
                    "location": "[resourceGroup().location]",
                    "properties": {
                      "addressSpace": {
                        "addressPrefixes": [
                          "10.0.0.0/16"
                        ]
                      },
                      "subnets": [
                        {
                          "name": "SubnetA",
                          "properties": {
                            "addressPrefix": "10.0.0.0/24"
                          }
                        }
                      ]
                    }
                  },
                  {
                    "type": "Microsoft.Network/publicIPAddresses",
                    "apiVersion": "2021-05-01",
                    "name": "app-ip",
                    "location": "[resourceGroup().location]",
                    "properties": {
                      "publicIPAllocationMethod": "Static",
                      "dnsSettings": {
                        "domainNameLabel": "app-set"
                      }
                    }
                  }
    ]
}
```
