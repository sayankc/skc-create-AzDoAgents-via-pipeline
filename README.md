[![Build Status](https://dev.azure.com/sayankc/devops-agile/_apis/build/status/Create%20Agent%20via%20201%20template?branchName=main)](https://dev.azure.com/sayankc/devops-agile/_build/latest?definitionId=12&branchName=main)

# Create-AzDo Agents via Azure Pipeline

 These files enable you to create an automated Agent Pool for Azure DevOps. The vms are deployed to Azure.

Idea is reffered from two links. 

-  <http://4bes.nl/2019/09/29/deploy-azure-devops-agents-through-an-azure-devops-pipeline>. 
- https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-windows 

### How to start
To start, refer the pipeline file names `alternate1-201-template.yml`.

This pipeline will not work without preparations.
## Prerequisites:
- The agent pool should be created in advanced in the portal. 
- A PAT key with the right permissions should be generated.
- Create the following variables:
    - Password :the admin password for the machines
    - Pat: The PAT key 

## What the pipeline does
I’ll walk through all the tasks in the pipeline

### Stage 1: Build
This stage will validate the ARM template.

### Stage 2: Deploy
Delete Existing Resource Group
Although the mode is set to complete, it will not delete existing resources if they are in the template. It is however my goal to create fresh new resources, so the first step is to delete the Resource Group.

(you can remove this step and make the deployment incremental, but the custom script extension is not working that well when deployed to an existing VM)

### Deploy New Azure Resources
The resources for the agents are deployed in this step through an ARM template. Also, the VMs are enabled for WinRM, so the files in the next step can be copied. A storage account is created to make it possible to copy the installation files. If you have one available already, it is possible to remove this from the template.


### Copy installation files
The files that are located in the installationfiles folder are copied to the VMs. This is basically a PowerShell script that is ran in the next step. The files will be placed in a folder called C:\temp.

### Deploy Custom Script Extension
This is an Azure PowerShell task that places a custom script extension on each VM. To do this, it will first remove the existing extension. After that, the new extension will call on the script that was place in C:\temp in the previous step. That script (Install-Agent.ps1) takes care of the Agent installation. As it runs locally, that script could also be used to provide the servers with other features, like certain Windows features or PowerShell modules. The installation files are downloaded from the Github Repository, as you would need to login in Azure DevOps to download it there.

### Result
In the end the agents will be placed in the pool.