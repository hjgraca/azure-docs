---
title: Understanding resource access in Azure | Microsoft Docs
description: This topic explains concepts about using subscription administrators to control resource access in the full Azure portal
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman

ms.assetid: 174f1706-b959-4230-9a75-bf651227ebf6
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/06/2017
ms.author: curtand

ms.custom: it-pro;

---
# Understanding resource access in Azure

Access control in Azure starts from a billing perspective. The owner of an Azure account, accessed by visiting the  [Azure Account Center](https://account.azure.com), is the Account Administrator (AA). Subscriptions are a container for billing, but they also act as a security boundary: each subscription has a Service Administrator (SA) who can add, remove, and modify Azure resources in that subscription by using the [Azure portal](https://portal.azure.com/). The default SA of a new subscription is the AA, but the AA can change the SA in the Azure Account Center.

<br><br>![Azure Accounts][1]

Subscriptions also have an association with a directory. The directory defines a set of users. These can be users from the work or school that created the directory or they can be external guest users. Subscriptions are accessible by a subset of those directory users who have been assigned as either Service Administrator (SA) or Co-Administrator (CA); the only exception is that, for legacy reasons, Microsoft accounts (formerly Windows Live ID) can be assigned as SA or CA without being present in the directory.

<br><br>![Access Control in Azure][2]

Functionality within the Azure portal enables SAs that are signed in using a Microsoft Account to change the directory with which a subscription is associated. This operation has implications on the access control of that subscription.

<br><br>![Simple User Sign-in Flow][3]

In the simple case, an organization (such as Contoso) will enforce billing and access control across the same set of subscriptions. That is, the directory is associated to subscriptions that are owned by a single Azure Account. Upon successful login to the Azure portal, users see two collections of resources (depicted in orange in the previous illustration):

* Directories where their user account exists (sourced or added as a foreign principal). Note that the directory used for login isn’t relevant to this computation, so your directories will always be shown regardless of where you logged in.
* Resources that are part of subscriptions that are associated with the directory used for login AND which the user can access (where they are an SA or CA).

<br><br>![User with Multiple Subscriptions and Directories][4]

Users with subscriptions across multiple directories have the ability to switch the current context of the Azure portal by using the subscription filter. Under the covers, this results in a separate login to a different directory, but this is accomplished seamlessly using single sign-on (SSO).

Operations such as moving resources between subscriptions can be more difficult as a result of this single directory view of subscriptions. To perform the resource transfer, it may be necessary to first use the **Edit Directory** command on the Subscriptions page in **Settings** to associate the subscriptions to the same directory.

## Next Steps
* To learn more about how to change administrators for an Azure subscription, see [How to add or change Azure administrator roles](../billing/billing-add-change-azure-subscription-administrator.md)
* For more information on how Azure Active Directory relates to your Azure subscription, see [How Azure subscriptions are associated with Azure Active Directory](../active-directory/active-directory-how-subscriptions-associated-directory.md)
* For more information on how to assign roles in Azure AD, see [Assigning administrator roles in Azure Active Directory](../active-directory/active-directory-assign-admin-roles-azure-portal.md)

<!--Image references-->
[1]: ./media/rbac-and-directory-admin-roles/IC707931.png
[2]: ./media/rbac-and-directory-admin-roles/IC707932.png
[3]: ./media/rbac-and-directory-admin-roles/IC707933.png
[4]: ./media/rbac-and-directory-admin-roles/IC707934.png
