---
title: Troubleshoot role-based access control Azure RBAC | Microsoft Docs
description: Get help with issues or questions about Role Based Access Control resources.
services: azure-portal
documentationcenter: na
author: rolyon
manager: mtillman

ms.assetid: df42cca2-02d6-4f3c-9d56-260e1eb7dc44
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2018
ms.author: rolyon
ms.reviewer: rqureshi
ms.custom: seohack1
---
# Troubleshooting Azure role-based access control 

This article answers common questions about the specific access rights that are granted with roles, so that you know what to expect when using the roles in the Azure portal and can troubleshoot access problems. These three roles cover all resource types:

* Owner  
* Contributor  
* Reader  

Owners and contributors both have full access to the management experience, but a contributor can’t give access to other users or groups. Things get a little more interesting with the reader role, so that’s where we'll spend some time. See the [Role-Based Access Control get-started article](role-assignments-portal.md) for details on how to grant access.

## App Service
### Write access capabilities
If you grant a user read-only access to a single web app, some features are disabled that you might not expect. The following management capabilities require **write** access to a web app (either Contributor or Owner), and aren’t available in any read-only scenario.

* Commands (like start, stop, etc.)
* Changing settings like general configuration, scale settings, backup settings, and monitoring settings
* Accessing publishing credentials and other secrets like app settings and connection strings
* Streaming logs
* Diagnostic logs configuration
* Console (command prompt)
* Active and recent deployments (for local git continuous deployment)
* Estimated spend
* Web tests
* Virtual network (only visible to a reader if a virtual network has previously been configured by a user with write access).

If you can't access any of these tiles, you need to ask your administrator for Contributor access to the web app.

### Dealing with related resources
Web apps are complicated by the presence of a few different resources that interplay. Here is a typical resource group with a couple websites:

![Web app resource group](./media/troubleshooting/website-resource-model.png)

As a result, if you grant someone access to just the web app, much of the functionality on the website blade in the Azure portal is disabled.

These items require **write** access to the **App Service plan** that corresponds to your website:  

* Viewing the web app’s pricing tier (Free or Standard)  
* Scale configuration (number of instances, virtual machine size, autoscale settings)  
* Quotas (storage, bandwidth, CPU)  

These items require **write** access to the whole **Resource group** that contains your website:  

* SSL Certificates and bindings (SSL certificates can be shared between sites in the same resource group and geo-location)  
* Alert rules  
* Autoscale settings  
* Application insights components  
* Web tests  

## Azure Functions
Some features of [Azure Functions](../azure-functions/functions-overview.md) require write access. For example, if a user is assigned the Reader role, they will not be able to view the functions within a function app. The portal will display **(No access)**.

![Function apps no access](./media/troubleshooting/functionapps-noaccess.png)

A reader can click the **Platform features** tab and then click **All settings** to view some settings related to a function app (similar to a web app), but they can't modify any of these settings.

## Virtual machine
Much like with web apps, some features on the virtual machine blade require write access to the virtual machine, or to other resources in the resource group.

Virtual machines are related to Domain names, virtual networks, storage accounts, and alert rules.

These items require **write** access to the **Virtual machine**:

* Endpoints  
* IP addresses  
* Disks  
* Extensions  

These require **write** access to both the **Virtual machine**, and the **Resource group** (along with the Domain name) that it is in:  

* Availability set  
* Load balanced set  
* Alert rules  

If you can't access any of these tiles, ask your administrator for Contributor access to the Resource group.

## See more
* [Role Based Access Control](role-assignments-portal.md): Get started with RBAC in the Azure portal.
* [Built-in roles](built-in-roles.md): Get details about the roles that come standard in RBAC.
* [Custom roles in Azure RBAC](custom-roles.md): Learn how to create custom roles to fit your access needs.
* [Create an access change history report](change-history-report.md): Keep track of changing role assignments in RBAC.

