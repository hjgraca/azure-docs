---
title: Azure Mobile Engagement Android SDK Integration
description: Latest updates and procedures for Android SDK for Azure Mobile Engagement
services: mobile-engagement
documentationcenter: mobile
author: piyushjo
manager: erikre
editor: ''

ms.assetid: 585341f9-3f39-459a-af42-864e400a0128
ms.service: mobile-engagement
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: Java
ms.topic: article
ms.date: 07/17/2017
ms.author: piyushjo

---
# Release notes
> [!IMPORTANT]
> Azure Mobile Engagement retires on 3/31/2018. This page will be deleted shortly after.
> 


## 4.3.1 (07/17/2017)
* Fix a crash that could rarely happen when calling `EngagementAgentUtils.isInDedicatedEngagementProcess`, which is also used by the `EngagementApplication` class.

## 4.3.0 (06/27/2017)
* Android 8 support (previous versions of the SDK will not work on Android 8).
* No more dependency on support library.
* Remove `EngagementFragmentActivity` class.
* Due to [Background Execution Limits](https://developer.android.com/preview/features/background.html) on Android 8, logs in background might be delayed until the user interacts with the device, this will have an impact on Push Campaign **Delivered** and **System notification displayed** statistics being delayed if the device was sleeping (the notification will still be displayed, will ring and vibrate in real time without issues).
* Due to [Background Location Limits](https://developer.android.com/preview/features/background-location-limits.html), the real time location in background will not be updated frequently on Android 8.

## 4.2.4 (03/30/2017)
* Fix in-app notification text colors on Android 7 to be the same as older Android versions.

## 4.2.3 (08/10/2016)
* No more WIFI lock.
* Fix a deadlock when calling getDeviceId before init (bug introduced in 4.2.0).

## 4.2.2 (05/17/2016)
* Stability improvements.

## 4.2.1 (05/10/2016)
* Security: disable web view local file access.
* Security: remove `EngagementPreferenceActivity` class that extends obsolete and unsecure `PreferenceActivity` class.
* Security: reach activities are now documented to use `exported="false"`, this flag can also be used in previous SDK versions.

## 4.2.0 (03/11/2016)
* The SDK is now licensed under MIT.
* Allow specifying a custom device identifier at SDK initialization time.

## 4.1.5 (02/01/2016)
* Stability improvements.

## 4.1.4 (01/26/2016)
* Stability improvements.

## 4.1.3 (12/9/2015)
* Stability improvements.

## 4.1.2 (11/25/2015)
* Stability improvements.

## 4.1.1 (11/04/2015)
* Stability improvements.

## 4.1.0 (08/25/2015)
* Handle new permission model for Android M.
* Can now configure location features at runtime instead of using  `AndroidManifest.xml`.
* Fix a permission bug: if you use `ACCESS_FINE_LOCATION`, then `ACCESS_COARSE_LOCATION` is not needed anymore.
* Stability improvements.

## 4.0.0 (07/06/2015)
* Internal protocol changes to make analytics and push more reliable.
* Native push (GCM/ADM) is now also used for in app notifications so you must configure the native push credentials for any type of push campaign.
* Fix big picture notification: they were displayed only 10s after being pushed.
* Fix a bug in web view: clicking on a link was also executing the default action URL.
* Fix a rare crash related to local storage management.
* Fix dynamic configuration string management.
* Update EULA.

## 3.0.0 (02/17/2015)
* Initial Release of Azure Mobile Engagement
* appId configuration is replaced by a connection string configuration.
* Removed API to send and receive arbitrary XMPP messages from arbitrary XMPP entities.
* Removed API to send and receive messages between devices.
* Security improvements.
* Google Play and SmartAd tracking removed.

