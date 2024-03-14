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

# TERRAFORM
This is similar to ARM templates but open source and can be used on other cloud providers as well, code is human readable too.





