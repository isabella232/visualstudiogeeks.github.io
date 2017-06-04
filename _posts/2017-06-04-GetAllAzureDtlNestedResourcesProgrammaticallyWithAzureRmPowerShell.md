---
layout: post
title: "Get all Azure DTL nested resources programmatically with AzureRm PowerShell"
date: 2017-06-04
author: tarun
tags: ["Azure", "AzureDevTestLabs", "PowerShell"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/PowerShellNinjaAzureRmAzureDtl.jpg"
description: "The Azure Dev Test Lab virtual machines are encapsulated in their own resource group, wondering how you can retrieve all resources and the contents of their nested resource groups programmatically through AzureRm PowerShell... Check this blogpost for a solution to get all Azure DTL nested resource groups and their contents through AzureRm PowerShell..."
permalink: /DevOps/GetAllAzureDtlNestedResourcesProgrammaticallyWithAzureRmPowerShell
published: true
keywords: "DevOps, Azure, AzureRm, AzurePowerShell, PowerShell, Get-AzureRmResource, Dtl, AzureDtl, Azure DevTestLabs, ResourceGroup, ARM, RGT, ResourceGroupName, Microsoft.DevTestLab/labs/virtualMachines"
---
Since all VMs with in an Azure Dev Test Lab (Azure DTL) are within their own resource group and have some sort of a random numeric postfix attached at the end of the resource group name. There isn't yet an out of box AzureRm PowerShell commandlet available to query dev test lab resources. I've have knocked together a script which you can use to get all resources & their nested resources within an Azure Dev Test Lab using PowerShell...
<!--more--> 

### Azure DTL resources as seen in Azure Portal
As you can see in the screen shot below, I have a bunch of resources with in the Azure Dev Test Lab like data disks, key vaults and virtual machines. When you click on one of these virtual machines you'll see that the random resource group name I alluded to earlier, with in this resource group you can see the nested resources.  

![All nested resources within Azure DTL in Azure Portal](/images/screenshots/tarun/AzureDtlResourceGroupInAzurePortal.PNG)

### Azure DTL resources via AzureRM PowerShell 
Alright, now that we know what the structure of the Azure DTL nested resources are, let's see how we can retrieve all resources within an Azure DTL resources along with their nested resources through AzureRM PowerShell script... 

``` powershell 
# Login to the Azure RM Account that contains your Dev Test Lab
Login-AzureRmAccount

# Specify the name of the Azure Dev Test Lab Resource Group
$labName = 'azsu-rg-devtest-dtlab-emt-001'

<# Get all resources that are attached to this resource group
  Note - Rather than using Get-AzureRmResourceGroup use Get-AzureRmResource
         using Get-AzureRmResource with the name of the lab allows you to get 
         all resources that are pinned to the Azure Dev Test Lab resource group 
#>
$resources = Get-AzureRmResource | Where-Object { $_.ResourceGroupName -eq $labName } | Sort-Object ResourceType -Descending


# Loop through each of the resources with in the Azure Dev Test Lab resource group 
foreach($r in $resources){

    # Print the resource name 
    write-host "`n"
    Write-Host $r.Name " ," $r.ResourceGroupName " ," $r.ResourceType

    # If the resource type is a virtual machine, there are nested resources within
    # you need to identift the cryptic resource group name of the virtual machine resource 
    if($r.ResourceType -eq 'Microsoft.DevTestLab/labs/virtualMachines'){
        
        # Interesting the resource group name is actually hidden away as a property of this resource group 
        $props = Get-AzureRmResource -ResourceId $r.ResourceId | Select Properties 

        # Now that we have the rgName of the nested resource, let's get the resources in this resource group 
        $subItems = Get-AzureRmResource | Where-Object {$_.ResourceGroupName -eq $props.Properties.computeId.Split('/')[4]} 
        
        # Loop through all the nested resources within this dev test lab virtual machine resource group 
        foreach($subItem in $subItems){
            Write-Host $subItem.Name " , " $subItem.ResourceType " ," $subItem.ResourceGroupName, $subItem
        }
        Write-Host "`n"        
    }
}

Write-Host "**** Found this useful? Let others on Twitter know :> @arora_tarun *****"


```

![Output of PowerShell script for Get all nested resources within Azure DTL with AzureRm PowerShell](/images/screenshots/tarun/OutputOfAzureDtlGetAllResourcesByPowerShell.png)

### So what's the secret sauce? 
The key thing that glues the virtual machine resource group in Dev Test Lab to it's cryptic resource group name is a property, see below... You can use the following snippet to get all the properties of the Azure DTL Virtual Machine... You have some interesting details here such as who created the VM, when was it created, whether a library image was used and exactly which one... 

``` powershell

        # Interesting the resource group name is actually hidden away as a property of this resource group 
        $props = Get-AzureRmResource -ResourceId $r.ResourceId | Select Properties 

        $props.Properties

```

The `ComputerId` property is what holds the cryptic resource group name... You need to split this value to retrieve the resource group name `$props.Properties.computeId.Split('/')[4]`, unfortunately there is no other easy way to retrieve this value as it stands today... 

![Get all Azure DTL Virtual Machine Properties Programmatically with AzureRm PowerShell](/images/screenshots/tarun/AzureDtlVmPropertiesPowerShellAzureRm.png)

That's it folks! Hope you found this useful. Leave a comment if you find a better way of doing this... 

Tarun 