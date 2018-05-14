---
title: Create a Virtual Machine Scale Set with an Azure template | Microsoft Docs
description: Learn how to quickly create a virtual machine scale with an Azure Resource Manager template
services: virtual-machine-scale-sets
documentationcenter: ''
author: iainfoulds
manager: jeconnoc
editor: ''
tags: azure-resource-manager

ms.assetid: 
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 11/16/2017
ms.author: iainfou

---

# Create a Virtual Machine Scale Set with the Azure CLI 2.0
A virtual machine scale set allows you to deploy and manage a set of identical, auto-scaling virtual machines. You can scale the number of VMs in the scale set manually, or define rules to autoscale based on resource usage such as CPU, memory demand, or network traffic. In this getting started article, you create a virtual machine scale set with an Azure Resource Manager template. You can also create a scale set with the [Azure CLI 2.0](quick-create-cli.md), [Azure PowerShell](quick-create-powershell.md), or the [Azure portal](quick-create-portal.md).


## Overview of templates
Azure Resource Manager templates let you deploy groups of related resources. Templates are written in JavaScript Object Notation (JSON) and define the entire Azure infrastructure environment for your application. In a single template, you can create the virtual machine scale set, install applications, and configure autoscale rules. With the use of variables and parameters, this template can be reused to update existing, or create additional, scale sets. You can deploy templates through the Azure portal, Azure CLI 2.0, or Azure PowerShell, as well as call them from continuous integration / continuous delivery (CI/CD) pipelines.

For more information on templates, see [Azure Resource Manager overview](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#template-deployment)


## Define a scale set
A template defines the configuration for each resource type. A virtual machine scale set resource type is similar to an individual VM. The core parts of the virtual machine scale set resource type are:

| Property                     | Description of property                                  | Example template value                    |
|------------------------------|----------------------------------------------------------|-------------------------------------------|
| type                         | Azure resource type to create                            | Microsoft.Compute/virtualMachineScaleSets |
| name                         | The scale set name                                       | myScaleSet                                |
| location                     | The location to create the scale set                     | East US                                   |
| sku.name                     | The VM size for each scale set instance                  | Standard_A1                               |
| sku.capacity                 | The number of VM instances to initially create           | 2                                         |
| upgradePolicy.mode           | VM instance upgrade mode when changes occur              | Automatic                                 |
| imageReference               | The platform or custom image to use for the VM instances | Canonical Ubuntu Server 16.04-LTS         |
| osProfile.computerNamePrefix | The name prefix for each VM instance                     | myvmss                                    |
| osProfile.adminUsername      | The username for each VM instance                        | azureuser                                 |
| osProfile.adminPassword      | The password for each VM instance                        | P@ssw0rd!                                 |

 The following snippet shows the core scale set resource definition in a template. To keep the sample short, the virtual network interface card (NIC) configuration is not shown. To customize a scale set template, you can change the VM size or initial capacity, or use a different platform or a custom image.

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US",
  "apiVersion": "2016-04-30-preview",
  "sku": {
    "name": "Standard_A1",
    "capacity": "2"
  },
  "properties": {
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
      "storageProfile": {
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage"
        },
        "imageReference":  {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "16.04-LTS",
          "version": "latest"
        }
      },
      "osProfile": {
        "computerNamePrefix": "myvmss",
        "adminUsername": "azureuser",
        "adminPassword": "P@ssw0rd!"
      }
    }
  }
}
```


## Install an application
When you deploy a scale set, VM extensions can provide post-deployment configuration and automation tasks, such as installing an app. Scripts can be downloaded from Azure storage or GitHub, or provided to the Azure portal at extension run-time. To apply an extension to your scale set, you add the *extensionProfile* section to the preceding resource example. The extension profile typically defines the following properties:

- Extension type
- Extension publisher
- Extension version
- Location of configuration or install scripts
- Commands to execute on the VM instances

Let's look at two ways to install an application with extensions - with the Custom Script Extension to install a Python app on Linux, or with the PowerShell DSC extension to install an ASP.NET app on Windows.

### Python HTTP server on Linux
The [Python HTTP server on Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-bottle-autoscale) uses the Custom Script Extension to install [Bottle](http://bottlepy.org/docs/dev/), a Python web framework, and a simple HTTP server. 

Two scripts are defined in *fileUris* - *installserver.sh*, and *workserver.py*. These files are downloaded from GitHub, then *commandToExecute* defines `bash installserver.sh` for the app to be installed and configured:

```json
"extensionProfile": {
  "extensions": [
    {
      "name": "AppInstall",
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "typeHandlerVersion": "2.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "fileUris": [
            "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-bottle-autoscale/installserver.sh",
            "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-bottle-autoscale/workserver.py"
          ],
          "commandToExecute": "bash installserver.sh"
        }
      }
    }
  ]
}
```

### ASP.NET application on Windows
The [ASP.NET application on Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-webapp-dsc-autoscale) sample template uses the PowerShell DSC extension to install an ASP.NET MVC app that runs in IIS. 

An install script is downloaded from GitHub, as defined in *url*. The extension then runs *InstallIIS* from the *IISInstall.ps1* script, as defined in *function* and *Script*. The ASP.NET app itself is provided as a Web Deploy package, which is also downloaded from GitHub, as defined in *WebDeployPackagePath*:

```json
"extensionProfile": {
  "extensions": [
    {
      "name": "Microsoft.Powershell.DSC",
      "properties": {
        "publisher": "Microsoft.Powershell",
        "type": "DSC",
        "typeHandlerVersion": "2.9",
        "autoUpgradeMinorVersion": true,
        "forceUpdateTag": "1.0",
        "settings": {
          "configuration": {
            "url": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/DSC/IISInstall.ps1.zip",
            "script": "IISInstall.ps1",
            "function": "InstallIIS"
          },
          "configurationArguments": {
            "nodeName": "localhost",
            "WebDeployPackagePath": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/WebDeploy/DefaultASPWebApp.v1.0.zip"
          }
        }
      }
    }
  ]
}
```

## Deploy the template
The simplest way to deploy the [Python HTTP server on Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-bottle-autoscale) or [ASP.NET MVC application on Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-webapp-dsc-autoscale) template is to use the **Deploy to Azure** button found in the readme files in GitHub.  You can also use PowerShell or Azure CLI to deploy the sample templates.

### Azure CLI 2.0
You can use the Azure CLI 2.0 to install the Python HTTP server on Linux as follows:

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location EastUS

# Deploy template into resource group
az group deployment create \
    --resource-group myResourceGroup \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-bottle-autoscale/azuredeploy.json
```

To see your app in action, obtain the public IP address of the load balancer with [az network public-ip list](/cli/azure/network/public-ip#az_network_public_ip_show) as follows:

```azurecli-interactive
az network public-ip list \
    --resource-group myResourceGroup \
    --query [*].ipAddress -o tsv
```

Enter the public IP address of the load balancer in to a web browser in the format *http://<publicIpAddress>:9000/do_work*. The load balancer distributes traffic to one of your VM instances, as shown in the following example:

![Default web page in NGINX](media/virtual-machine-scale-sets-create-template/running-python-app.png)


### Azure PowerShell
You can use Azure PowerShell to install the ASP.NET application on Windows as follows:

```azurepowershell-interactive
# Create a resource group
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS

# Deploy template into resource group
New-AzureRmResourceGroupDeployment `
    -ResourceGroupName myResourceGroup `
    -TemplateFile https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/azuredeploy.json
```

To see your app in action, obtain the public IP address of your load balancer with [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) as follows:

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress
```

Enter the public IP address of the load balancer in to a web browser in the format *http://<publicIpAddress>/MyApp*. The load balancer distributes traffic to one of your VM instances, as shown in the following example:

![Running IIS site](./media/virtual-machine-scale-sets-create-powershell/running-iis-site.png)


## Clean up resources
When no longer needed, you can use [az group delete](/cli/azure/group#az_group_delete) to remove the resource group, scale set, and all related resources as follows:

```azurecli-interactive 
az group delete --name myResourceGroup
```


## Next steps
