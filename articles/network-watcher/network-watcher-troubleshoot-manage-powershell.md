---
title: Troubleshoot Azure Virtual Network Gateway and Connections - PowerShell | Microsoft Docs
description: This page explains how to use the Azure Network Watcher troubleshoot PowerShell cmdlet
services: network-watcher
documentationcenter: na
author: georgewallace
manager: timlt
editor: 

ms.assetid: e4d5f195-b839-4394-94ef-a04192766e55
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload:  infrastructure-services
ms.date: 02/22/2017
ms.author: gwallace

---

# Troubleshoot Virtual Network Gateway and Connections using Azure Network Watcher PowerShell

Network Watcher provides many capabilities as it relates to understanding your network resources in Azure. One of these capabilities is resource troubleshooting. Resource troubleshooting can be called by PowerShell, CLI, or REST API. When called, Network Watcher inspects the health of a Virtual Network Gateway or a Connection and returns its findings.

## Before you begin

This scenario assumes you have already followed the steps in [Create a Network Watcher](network-watcher-create.md) to create a Network Watcher.

## Overview

Resource troubleshooting provides the ability troubleshoot issues that arise with Virtual Network Gateways and Connections. When a request is made to resource troubleshooting, logs are being queried and inspected. When inspection is complete, the results are returned. Resource troubleshooting requests are long running requests, which could take multiple minutes to return a result. The logs from troubleshooting are stored in a container on a storage account that is specified.

## Retrieve Network Watcher

The first step is to retrieve the Network Watcher instance. The `$networkWatcher` variable is passed to the `Start-AzureRmNetworkWatcherResourceTroubleshooting` cmdlet in step 4.

```powershell
$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" } 
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName 
```

## Retrieve a Virtual Network Gateway Connection

In this example, resource troubleshooting is being ran on a Connection. You can also pass it a Virtual Network Gateway.

```powershell
$connection = Get-AzureRmVirtualNetworkGatewayConnection -Name "2to3" -ResourceGroupName "testrg"
```

## Create a storage account

Resource troubleshooting returns data about the health of the resource, it also saves logs to a storage account to be reviewed. In this step, we create a storage account, if an existing storage account exists you can use it.

```powershell
$sa = New-AzureRmStorageAccount -Name "contosoexamplesa" -SKU "Standard_LRS" -ResourceGroupName "testrg" -Location "WestCentralUS"
```

## Run Network Watcher resource troubleshooting

You troubleshoot resources with the `Start-AzureRmNetworkWatcherResourceTroubleshooting` cmdlet. We pass the cmdlet the Network Watcher object, the Id of the Connection or Virtual Network Gateway, the storage account id, and the path to store the results.

> [!NOTE]
> The `Start-AzureRmNetworkWatcherResourceTroubleshooting` cmdlet is long running and may take a few minutes to complete.

```powershell
Start-AzureRmNetworkWatcherResourceTroubleshooting -NetworkWatcher $networkWatcher -TargetResourceId $connection.Id -StorageId $sa.Id -StoragePath "$($sa.PrimaryEndpoints.Blob)logs"
```

Once you run the cmdlet, Network Watcher reviews the resource to verify the health. It returns the results to the shell and stores logs of the results in the storage account specified.

## Understanding the results

The action text provides general guidance on how to resolve the issue. If an action can be taken for the issue, a link is provided with additional guidance. In the case where there is no additional guidance, the response provides the url to open a support case.  For more information about the properties of the response and what is included, visit [Network Watcher Troubleshoot overview](network-watcher-troubleshoot-overview.md)

For instructions on downloading files from azure storage accounts, refer to [Get started with Azure Blob storage using .NET](../storage/storage-dotnet-how-to-use-blobs.md). Another tool that can be used is Storage Explorer. More information about Storage Explorer can be found here at the following link: [Storage Explorer](http://storageexplorer.com/)

## Next steps

If settings have been changed that stop VPN connectivity, see [Manage Network Security Groups](../virtual-network/virtual-network-manage-nsg-arm-portal.md) to track down the network security group and security rules that may be in question.
