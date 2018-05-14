---
title: Azure Search Service REST API Version 2016-09-01-Preview | Microsoft Docs
description: Azure Search Service REST API Version 2016-09-01-Preview includes experimental features such as moreLikeThis searches.
author: mhko
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: reference
ms.date: 04/18/2018
ms.author: nateko

---
# Azure Search Service REST API: Version 2016-09-01-Preview
This article is the reference documentation for `api-version=2016-09-01-Preview`. This preview extends the current generally available version, [api-version=2016-09-01](https://docs.microsoft.com/rest/api/searchservice), by providing the following experimental features:

* [`moreLikeThis` query parameter](search-more-like-this.md) to find documents that are relevant to a specific document.

Please ensure to target the preview API version `api-version=2016-09-01-Preview` to try these experimental features. The following example illustrates how the preview api version is specified in making a more-like-this query.

    GET https://[service name].search.windows.net/indexes/[index name]/docs?moreLikeThis=a1&api-version=2016-09-01-Preview

> [!NOTE]
> Preview features are available for testing and experimentation with the goal of gathering feedback and are subject to change. **We strongly advise against using preview APIs in production applications.**

Azure Search service is available in multiple versions. Please refer to [Search Service Versioning](https://docs.microsoft.com/azure/search/search-api-versions) for details.
