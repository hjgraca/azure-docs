---
title: Azure Kubernetes Service (AKS) quotas and region availability
description: The default quotas and region availability of the Azure Kubernetes Service (AKS).
services: container-service
author: neilpeterson
manager: jeconnoc

ms.service: container-service
ms.topic: overview
ms.date: 04/26/2018
ms.author: nepeters
---
# Quotas and region availability for Azure Kubernetes Service (AKS)

All Azure services include certain default limits and quotas for resources and features. The following sections detail the default resource limits for several Azure Kubernetes Service (AKS) resources, as well as the availability of the AKS service in Azure regions.

## Service quotas and limits

[!INCLUDE [container-service-limits](../../includes/container-service-limits.md)]

## Provisioned infrastructure

All other network, compute, and storage limitations apply to the provisioned infrastructure. See [Azure subscription and service limits](../azure-subscription-service-limits.md) for the relevant limits.

## Region availability

Azure Kubernetes Service (AKS) is available for preview in the following regions:
- East US
- West Europe
- Central US
- Canada Central
- Canada East

## Next steps

Certain default limits and quotas can be increased. To request an increase of one or more resources that support such an increase, please submit an [Azure support request][azure-support] (select "Quota" for **Issue type**).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
