---
title: Deploying a Virtual Machine to Azure using Bicep and CloudShell
author: TomWhi
date: 2022-08-08
categories: [DevOps]
tags: [devops,bicep,azure,cloudshell]
---

## Overview 

I wanted to document an easy example to show how to use Bicep and deploy something simple. 

I've tried to simplify the steps from the MS Guide and and included steps for deploying using the Cloud Shell.  Of course this is possible via PowerShell on a local machine but avoids those additional steps to focus on Bicep and the output it generates. 



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

Copy the following code into the bicep file 

```console
@description('Username for the Virtual Machine.')
param adminUsername string

@description('Password for the Virtual Machine.')
@minLength(12)
@secure()
param adminPassword string

@description('Unique DNS Name for the Public IP used to access the Virtual Machine.')
param dnsLabelPrefix string = toLower('${vmName}-${uniqueString(resourceGroup().id, vmName)}')

@description('Name for the Public IP used to access the Virtual Machine.')
param publicIpName string = 'myPublicIP'

@description('Allocation method for the Public IP used to access the Virtual Machine.')
@allowed([
	'Dynamic'
	'Static'
])
param publicIPAllocationMethod string = 'Dynamic'

@description('SKU for the Public IP used to access the Virtual Machine.')
@allowed([
	'Basic'
	'Standard'
])
param publicIpSku string = 'Basic'

@description('The Windows version for the VM. This will pick a fully patched image of this given Windows version.')
@allowed([
'2008-R2-SP1'
'2008-R2-SP1-smalldisk'
'2012-Datacenter'
'2012-datacenter-gensecond'
'2012-Datacenter-smalldisk'
'2012-datacenter-smalldisk-g2'
'2012-Datacenter-zhcn'
'2012-datacenter-zhcn-g2'
'2012-R2-Datacenter'
'2012-r2-datacenter-gensecond'
'2012-R2-Datacenter-smalldisk'
'2012-r2-datacenter-smalldisk-g2'
'2012-R2-Datacenter-zhcn'
'2012-r2-datacenter-zhcn-g2'
'2016-Datacenter'
'2016-datacenter-gensecond'
'2016-datacenter-gs'
'2016-Datacenter-Server-Core'
'2016-datacenter-server-core-g2'
'2016-Datacenter-Server-Core-smalldisk'
'2016-datacenter-server-core-smalldisk-g2'
'2016-Datacenter-smalldisk'
'2016-datacenter-smalldisk-g2'
'2016-Datacenter-with-Containers'
'2016-datacenter-with-containers-g2'
'2016-datacenter-with-containers-gs'
'2016-Datacenter-zhcn'
'2016-datacenter-zhcn-g2'
'2019-Datacenter'
'2019-Datacenter-Core'
'2019-datacenter-core-g2'
'2019-Datacenter-Core-smalldisk'
'2019-datacenter-core-smalldisk-g2'
'2019-Datacenter-Core-with-Containers'
'2019-datacenter-core-with-containers-g2'
'2019-Datacenter-Core-with-Containers-smalldisk'
'2019-datacenter-core-with-containers-smalldisk-g2'
'2019-datacenter-gensecond'
'2019-datacenter-gs'
'2019-Datacenter-smalldisk'
'2019-datacenter-smalldisk-g2'
'2019-Datacenter-with-Containers'
'2019-datacenter-with-containers-g2'
'2019-datacenter-with-containers-gs'
'2019-Datacenter-with-Containers-smalldisk'
'2019-datacenter-with-containers-smalldisk-g2'
'2019-Datacenter-zhcn'
'2019-datacenter-zhcn-g2'
'2022-datacenter'
'2022-datacenter-azure-edition'
'2022-datacenter-azure-edition-core'
'2022-datacenter-azure-edition-core-smalldisk'
'2022-datacenter-azure-edition-smalldisk'
'2022-datacenter-core'
'2022-datacenter-core-g2'
'2022-datacenter-core-smalldisk'
'2022-datacenter-core-smalldisk-g2'
'2022-datacenter-g2'
'2022-datacenter-smalldisk'
'2022-datacenter-smalldisk-g2'
])
param OSVersion string = '2022-datacenter-azure-edition-core'

@description('Size of the virtual machine.')
param vmSize string = 'Standard_D2s_v5'

@description('Location for all resources.')
param location string = resourceGroup().location

@description('Name of the virtual machine.')
param vmName string = 'simple-vm'

var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
var nicName = 'myVMNic'
var addressPrefix = '10.0.0.0/16'
var subnetName = 'Subnet'
var subnetPrefix = '10.0.0.0/24'
var virtualNetworkName = 'MyVNET'
var networkSecurityGroupName = 'default-NSG'

resource stg 'Microsoft.Storage/storageAccounts@2021-04-01' = {
	name: storageAccountName
	location: location
	sku: {
	name: 'Standard_LRS'
	}
	kind: 'Storage'
}

resource pip 'Microsoft.Network/publicIPAddresses@2021-02-01' = {
	name: publicIpName
	location: location
	sku: {
	name: publicIpSku
	}
	properties: {
	publicIPAllocationMethod: publicIPAllocationMethod
	dnsSettings: {
		domainNameLabel: dnsLabelPrefix
	}
	}
}

resource securityGroup 'Microsoft.Network/networkSecurityGroups@2021-02-01' = {
	name: networkSecurityGroupName
	location: location
	properties: {
	securityRules: [
		{
		name: 'default-allow-3389'
		properties: {
			priority: 1000
			access: 'Allow'
			direction: 'Inbound'
			destinationPortRange: '3389'
			protocol: 'Tcp'
			sourcePortRange: '*'
			sourceAddressPrefix: '*'
			destinationAddressPrefix: '*'
		}
		}
	]
	}
}

resource vn 'Microsoft.Network/virtualNetworks@2021-02-01' = {
	name: virtualNetworkName
	location: location
	properties: {
	addressSpace: {
		addressPrefixes: [
		addressPrefix
		]
	}
	subnets: [
		{
		name: subnetName
		properties: {
			addressPrefix: subnetPrefix
			networkSecurityGroup: {
			id: securityGroup.id
			}
		}
		}
	]
	}
}

resource nic 'Microsoft.Network/networkInterfaces@2021-02-01' = {
	name: nicName
	location: location
	properties: {
	ipConfigurations: [
		{
		name: 'ipconfig1'
		properties: {
			privateIPAllocationMethod: 'Dynamic'
			publicIPAddress: {
			id: pip.id
			}
			subnet: {
			id: resourceId('Microsoft.Network/virtualNetworks/subnets', vn.name, subnetName)
			}
		}
		}
	]
	}
}

resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = {
	name: vmName
	location: location
	properties: {
	hardwareProfile: {
		vmSize: vmSize
	}
	osProfile: {
		computerName: vmName
		adminUsername: adminUsername
		adminPassword: adminPassword
	}
	storageProfile: {
		imageReference: {
		publisher: 'MicrosoftWindowsServer'
		offer: 'WindowsServer'
		sku: OSVersion
		version: 'latest'
		}
		osDisk: {
		createOption: 'FromImage'
		managedDisk: {
			storageAccountType: 'StandardSSD_LRS'
		}
		}
		dataDisks: [
		{
			diskSizeGB: 1023
			lun: 0
			createOption: 'Empty'
		}
		]
	}
	networkProfile: {
		networkInterfaces: [
		{
			id: nic.id
		}
		]
	}
	diagnosticsProfile: {
		bootDiagnostics: {
		enabled: true
		storageUri: stg.properties.primaryEndpoints.blob
		}
	}
	}
}

output hostname string = pip.properties.dnsSettings.fqdn
```

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

**Once you have completed the testing**, don't forget to delete the resources you have created, or delete the resource group containing all the resources. 

# Links 

<https://docs.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-bicep?tabs=CLI>