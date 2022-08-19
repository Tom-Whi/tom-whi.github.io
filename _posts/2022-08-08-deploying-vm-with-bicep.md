---
title: Deploying a Virtual Machine to Azure using Bicep and CloudShell
author: TomWhi
date: 2022-08-08
categories: [DevOps]
tags: [devops,bicep,azure,cloudshell,lab]
---

## Overview 

I wanted to document an easy example to show how to use Bicep and deploy something simple. 

I've tried to simplify the steps from the MS Guide and included steps for deploying using the Cloud Shell.  Of course this is possible via PowerShell on a local machine but avoids those additional steps to focus on Bicep and the output it generates. 



## Steps

You will need a Cloud Shell.  If you don't have one already then set one up using [this guide](https://blog.tomwhiteley.com/posts/creating-azure-cloudshell/)

Create a folder in your CloudShell called bicep.  Move into that folder. 

```powershell
mkdir bicep
cd bicep
```

Create a new file in that folder called vm.bicep

```powershell
echo "x" > vm.bicep
```

Open the file in VS Code within the CloudShell 

```powershell
code vm.bicep
```

Copy the following code from my Github repo into the bicep file. Use the raw version to avoid any formatting issues! 

- <https://github.com/Tom-Whi/Random-Things/blob/main/Bicep/vm.bicep>

Look at the file's properties - it's going to create 

- The parameters at the top allows the Bicep file to be called with parameters.  In most cases there are defaults for each parameter (so check these are ok for you before you deploy it). 
	
- There are variables - these cannot be overridden from outside of the Bicep file, they can be changed to Parameters though if you want to pass them in like that. 
	
- It's then going to create the resources
	
	- Storage account for Boot diags 
	- PIP
	- NSG
	- VNET & Subnet
	- NIC
	- VM
		
- Finally there is then an output in the Bicep file which outputs the generated DNS record for the VM. 
	
Make sure you're happy with the AZ Context of the shell (which sub it'll deploy into if you have more than one)

```powershell
#Get current context
Get-AzContext | Select-Object -Property Name, Subscription, Tenant | Format-List

#Get the subscriptions in your account 
Get-AzSubscription | Select-Object -Property Id, Name, TenantId | Format-List

#Set the subscription if you want to change it (enter a valid Sub ID)
Set-AzContext -SubscriptionId 22222222-2222-4000-22222222222222222
```

Deploy the  a resource group through the CloudShell

```powershell
New-AzResourceGroup -Name exampleRG -Location "East US"
```

Deploy the Bicep file into that Resource Group - notice the parameters I've used.  In my subscription I have a policy against deploying D series VMs in my Azure so I need to override to deploy a B series one, this parameter matches the ones in the file above. 

```powershell 
New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./vm.bicep -adminUsername "tom" -vmSize "Standard_B2s" 
#You will be prompted for a Password after you hit enter
```

While the VM is deploying, go to the Azure Resource Group and click Deployments - notice the deployment is in progress. 

Once the powershell has completed you will be given the output of the DNS name (as this is specified as an output in the Bicep file).  RDP to that DNS entry and logon with the credentials you have used. 

> Once you have completed the testing, don't forget to `delete the resources` you have created, or `delete the resource group` containing all the resources. 
{: .prompt-danger }

# Homework

Once you've completed the deployment above maybe try the additional steps 

- Change the size of the VM to *Standard_B2ms* on the command and rerun the command. 
- Remove the VMsize on the command line and change the default in the vm.bicep file, and rerun the command. 
- Delete the VM and redeploy the VM with a different type of OS - don't forget about the limit imposed by the bicep file. This could be done in two ways, via the command line as a parameter, or as the default value in the bicep file.

# Links 

<https://docs.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-bicep?tabs=CLI>