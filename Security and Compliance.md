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
