---
layout: post
title: "How to restore Azure SQL DW GeoBackup across Azure subscriptions.."
date: 2018-01-24
author: tarun
tags: ["DevOps", "PowerShell", "AzureSQLDW"]
categories:
- "DevOps"
- "Azure"
image: "/images/screenshots/tarun/azureSqlDwGeoBackupRestore.jpg"
description: "Wondering how to restore an Azure SQL DW geo backup from one Azure subscription to another, look no further, this blog shows you how to leverage Azure PowerShell to automate the restore of the Azure SQL DW from one Azure subscription to another... "
permalink: /DevOps/RestoringAzureSQlDwGeoBackupBetweenAzureSubscriptions
published: true
keywords: "Azure, SQL DW, Azure SQL DW, SQLDW, SQL Datawarehouse, Geobackup, Restore, Restore Geobackup, Azure Subscription, Geobackup Restore SQL DW between Azure subscriptions, How to restore Azure SQL DW between Azure subscriptions, Automate geobackup restore of SQL DW between Azure subscriptions, PowerShell SQL DW Geobackup restore between Azure subscriptions, Azure PowerShell, Automate backup restore for SQL, Move SQL DW between Azure subscriptions, Azure SQL DW Geobackup restore "
---
If you have arrived here, you are likely using Azure SQL DW and wondering how to restore a backup from one Azure subscription to another...? While geo backup's are enabled on Azure SQL DW by default, unfortunately there isn't native support to restore a SQL DW geo backup from one Azure subscription to another. It's very common for enterprises customers to keep production workloads in a separate Azure subscription to test workloads. Look no further, no matter what the question - PowerShell is always the answer! 
<!--more-->

Approach - Geo backups can be restored to a new instance of Azure SQL DW in an existing subscription; there is support to move a SQL instance with all it's databases from one Azure subscription to another... Voila! But if you had to do this manually everyday, it would be error prone and painful! This is where the AzureRM PowerShell comes to rescue... 


``` PowerShell

$restorePtDt = "" # use format yyyymmddHH:mm:ss

$Source_Dw_rg = "azsu-rg-prod-myapp-001"
$Source_Dw_svr = "azsu-sqldw-paas-prod-myapp-001"
$Source_Dw_db = "myapp_dw"

$GeoPair_Source_Dw_rg = "azwu-rg-prod-myapp-001"

$Target_Dw_Rg = "azsu-rg-devtest-emt-cia-001"
$rdm = (Get-Random -Minimum 100 -Maximum 999)
$Tagret_Dw_svr = "azsu-prod2dev-emt-vault-" + $rdm
$Target_Dw_usr = "azureuser"
$Target_Dw_pwd = "iL0veMyJob!"

$Target_Azure_SubscriptionId = "xxxxxx-xxxx-xxx-xxxx-xxxxxxx"

# Get a geo restore backup for the specified datetime
if(!$restorePtDt){
    $allBackUps = ((Get-AzureRmSqlDatabaseGeoBackup `
                    -ResourceGroupName $Source_Dw_rg -ServerName `
                    $Source_Dw_svr -DatabaseName $Source_Dw_db))[-1..-500] 
    $Source_Dw_svr_geoBackup = $allBackUps `
                | Where {$_.LastAvailableBackupDate `
                    -eq [datetime]::ParseExact($restorePtDt, 'yyyyMMddHH:mm:ss', `
                    $null).ToString('dd/MM/yyyy HH:mm:ss')}
}

# Get the last available Geo Restore Backup
if(!$Source_Dw_svr_geoBackup){
    $Source_Dw_svr_geoBackup = ((Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName `
                $Source_Dw_rg -ServerName $Source_Dw_svr -DatabaseName $Source_Dw_db))[-1]
}

# Generate creadentials for SQL 
$pwd = ConvertTo-SecureString $Target_Dw_pwd -AsPlainText -Force
$sqlcred = new-object System.Management.Automation.PSCredential ($Target_Dw_usr, $pwd)

# Create a new SQL DW in the SQL DW Geo Pair 
New-AzureRmSqlServer -ResourceGroupName $GeoPair_Source_Dw_rg `
                     -ServerName $Tagret_Dw_svr -Location uksouth `
                     -ServerVersion "12.0" -SqlAdministratorCredentials  $sqlcred -Verbose

# Restore the Geo backup to this new SQL DW instance
Restore-AzureRmSqlDatabase -FromGeoBackup -ResourceGroupName $GeoPair_Source_Dw_rg `
                            -ServerName $Tagret_Dw_svr `
                           -TargetDatabaseName ("vault_dw-restore_"+ `
                            $Source_Dw_svr_geoBackup.LastAvailableBackupDate.ToString("yyyyMMddHHmmss")) `
                           -ResourceId $Source_Dw_svr_geoBackup.ResourceId `
                           -Edition DataWarehouse -ServiceObjectiveName "DW100" -Verbose

# Get ResourceId of the SQL DW that needs to be moved to a new Azure subscription
$instanceToMigrate = Get-AzureRmResource `
                            -ResourceGroupName $GeoPair_Source_Dw_rg `
                            -ResourceName $Tagret_Dw_svr

# Move an Azure Rescource from one Azure subscription to another 
Move-AzureRmResource -DestinationResourceGroupName $Target_Dw_Rg `
                     -ResourceId $instanceToMigrate.ResourceId `
                     -DestinationSubscriptionId $Target_Azure_SubscriptionId -Force

```

Voila! Run the above script and you'll have a new SQL DW instance created and the last geo backup restored on this instance and this instance migrated from the origin Azure subscription to another Azure subscription. 

Now Azure SQL DW is an expensive service, so you probably don't want to have a lot of these instances running unless you need them. You can optionally use the snippet below to retrieve any other SQL DW instances running in the target Azure Subscription and tag them for deletion... 


``` PowerShell 

if($deleteExisting){
    $existingServers = Get-AzureRmSqlServer -ResourceGroupName $Target_Dw_Rg

    foreach($e in $existingServers){
        if($e.ServerName -ne $Tagret_Dw_svr){
            Set-AzureRmResource $e.ResourceId `
            -Tag @{"MarkedForDelete"="true";"MarkedForDeleteOn"=(Get-Date -Format "yyyyMMdd")} -Force
        }
    }
}

```


I hope you find this useful! Is this not working for you, reach out to me on [@arora_tarun](https://twitter.com/arora_tarun)

Tarun 