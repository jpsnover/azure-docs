---
title: Install and configure PowerShell for Azure Stack quickstart  | Microsoft Docs
description: Learn about installing and configuring PowerShell for Azure Stack.
services: azure-stack
documentationcenter: ''
author: SnehaGunda
manager: byronr
editor: ''

ms.assetid:
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/13/2017
ms.author: sngun

---

# Get up and running with PowerShell in Azure Stack

*Applies to: Azure Stack integrated systems and Azure Stack Development Kit*

This article is a quick start to install and configure Azure Stack environment with PowerShell. This script provided in this article is scoped to by the **Azure Stack operator** only.

This article is a condensed version of the steps described in the [Install PowerShell]( azure-stack-powershell-install.md), [Download tools]( azure-stack-powershell-download.md), [Configure the Azure Stack operator's PowerShell environment]( azure-stack-powershell-configure-admin.md) articles. By using the scripts in this topic, you can set up PowerShell for Azure Stack environments that are deployed with Azure Active Directory or Active Directory Federation Services.  


## Set up PowerShell for AAD based deployments

Sign in to your Azure Stack Development Kit, or a Windows-based external client if you are connected through VPN. Open an elevated PowerShell ISE session and run the following script (make sure to update the TenantName, ArmEndpoint, GraphAudience variables as per your environment configuration):

> [!IMPORTANT]
> The release of the AzureRM 1.2.11 PowerShell module comes with a list of breaking changes. To upgrade from the 1.2.10 version, see the migration guide at [https://aka.ms/azspowershellmigration](https://aka.ms/azspowershellmigration).

```powershell
# Specify Azure Active Directory tenant name
$TenantName = "<mydirectory>.onmicrosoft.com"

# Set the module repository and the execution policy
Set-PSRepository `
  -Name "PSGallery" `
  -InstallationPolicy Trusted

Set-ExecutionPolicy RemoteSigned `
  -force

# Uninstall any existing Azure PowerShell modules. To uninstall, close all the active PowerShell sessions and run the following command:
Get-Module -ListAvailable | `
  where-Object {$_.Name -like “Azure*”} | `
  Uninstall-Module

# Install PowerShell for Azure Stack
Install-Module `
  -Name AzureRm.BootStrapper `
  -Force

Use-AzureRmProfile `
  -Profile 2017-03-09-profile `
  -Force

Install-Module `
  -Name AzureStack `
  -RequiredVersion 1.2.11 `
  -Force 

# Download Azure Stack tools from GitHub and import the connect module
cd \

invoke-webrequest `
  https://github.com/Azure/AzureStack-Tools/archive/master.zip `
  -OutFile master.zip

expand-archive master.zip `
  -DestinationPath . `
  -Force

cd AzureStack-Tools-master

Import-Module `
  .\Connect\AzureStack.Connect.psm1

# For Azure Stack development kit, this value is set to https://adminmanagement.local.azurestack.external. To get this value for Azure Stack integrated systems, contact your service provider.
$ArmEndpoint = "<Resource Manager endpoint for your environment>"

# For Azure Stack development kit, this value is set to https://graph.windows.net/. To get this value for Azure Stack integrated systems, contact your service provider.
$GraphAudience = "<GraphAuidence endpoint for your environment>"

# Configure the Azure Stack operator’s PowerShell environment.
Add-AzureRMEnvironment `
  -Name "AzureStackAdmin" `
  -ArmEndpoint $ArmEndpoint

Set-AzureRmEnvironment `
  -Name "AzureStackAdmin" `
  -GraphAudience $GraphAudience

$TenantID = Get-AzsDirectoryTenantId `
  -AADTenantName $TenantName `
  -EnvironmentName AzureStackAdmin

# Sign-in to the operator's portal.
Login-AzureRmAccount `
  -EnvironmentName "AzureStackAdmin" `
  -TenantId $TenantID 

```

## Set up PowerShell for AD FS based deployments 

Sign in to your Azure Stack Development Kit, or a Windows-based external client if you are connected through VPN. Open an elevated PowerShell ISE session and run the following script (make sure to update the ArmEndpoint, GraphAudience variables as per your environment configuration):

```powershell

# Set the module repository and the execution policy
Set-PSRepository `
  -Name "PSGallery" `
  -InstallationPolicy Trusted

Set-ExecutionPolicy RemoteSigned `
  -force

# Uninstall any existing Azure PowerShell modules. To uninstall, close all the active PowerShell sessions and run the following command:
Get-Module -ListAvailable | `
  where-Object {$_.Name -like “Azure*”} | `
  Uninstall-Module

# Install PowerShell for Azure Stack
Install-Module `
  -Name AzureRm.BootStrapper `
  -Force

Use-AzureRmProfile `
  -Profile 2017-03-09-profile `
  -Force

Install-Module `
  -Name AzureStack `
  -RequiredVersion 1.2.11 `
  -Force 

# Download Azure Stack tools from GitHub and import the connect module
cd \

Invoke-WebRequest `
  https://github.com/Azure/AzureStack-Tools/archive/master.zip `
  -OutFile master.zip

Expand-Archive master.zip `
  -DestinationPath . `
  -Force

cd AzureStack-Tools-master

Import-Module `
  .\Connect\AzureStack.Connect.psm1

# For Azure Stack development kit, this value is set to https://adminmanagement.local.azurestack.external. To get this value for Azure Stack integrated systems, contact your service provider.
$ArmEndpoint = "<Resource Manager endpoint for your environment>"

# For Azure Stack development kit, this value is set to https://graph.local.azurestack.external/. To get this value for Azure Stack integrated systems, contact your service provider.
$GraphAudience = "<GraphAudience endpoint for your environment>"

# Configure the cloud administrator’s PowerShell environment.
Add-AzureRMEnvironment `
  -Name "AzureStackAdmin" `
  -ArmEndpoint $ArmEndpoint

Set-AzureRmEnvironment `
  -Name "AzureStackAdmin" `
  -GraphAudience $GraphAudience `
  -EnableAdfsAuthentication:$true

$TenantID = Get-AzsDirectoryTenantId `
  -ADFS `
  -EnvironmentName "AzureStackAdmin"

# Sign-in to the operator's portal.
Login-AzureRmAccount `
  -EnvironmentName "AzureStackAdmin" `
  -TenantId $TenantID 

```

## Test the connectivity

Now that you’ve configured PowerShell, you can test the configuration by creating a resource group:

```powershell
New-AzureRMResourceGroup -Name "ContosoVMRG" -Location Local
```

When the resource group is created, the cmdlet output has the Provisioning state property set to "Succeeded."

## Next steps

* [Install and configure CLI](azure-stack-connect-cli.md)

* [Develop templates](user/azure-stack-develop-templates.md)







