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

Once temnplate is ready, here is how to deploy.
- Create a resource group say template-grp
- Create a resource, search "template" and select "Template Deployment"
- Click on "Build your own template in editor"
- Copy your ARM template and click on "Save"
- Select resource group template-grp and hit "Create"
- It will create resources (app plan and app service in this case)

Below is the ARM template
```arm
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
            "name": "newapp994848",
            "location": "[resourceGroup().location]",
            "properties":{
                "name":"newapp994848",
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
