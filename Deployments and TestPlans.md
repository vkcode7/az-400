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

# Azute Traffic Manager
This is DNS based traffic load balancer. You can distribute traffic across different Azure regions or based on different routing methods.
