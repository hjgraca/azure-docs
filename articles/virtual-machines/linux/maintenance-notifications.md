---
title: Handling maintenance notifications for Linux VMs in Azure | Microsoft Docs
description: View maintenance notifications for Linux virtual machines running in Azure and start self-service maintenance.
services: virtual-machines-linux
documentationcenter: ''
author: zivraf
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager

ms.assetid: 
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 12/15/2017
ms.author: zivr

---


# Handling planned maintenance notifications for Linux virtual machines

Azure periodically performs updates to improve the reliability, performance, and security of the host infrastructure for virtual machines. Updates are changes like patching the hosting environment or upgrading and decommissioning hardware. A majority of these updates are performed without any impact to the hosted virtual machines. However, there are cases where updates do have an impact:

- If the maintenance does not require a reboot, Azure uses in-place migration to pause the VM while the host is updated.

- If maintenance requires a reboot, you get a notice of when the maintenance is planned. In these cases, you are given a time window where you can start the maintenance yourself, when it works for you.


Planned maintenance that requires a reboot is scheduled in waves. Each wave has different scope (regions).

- A wave starts with a notification to customers. By default, notification is sent to subscription owner and co-owners. You can add more recipients and messaging options like email, SMS, and webhooks, to the notifications using Azure [Activity Log Alerts](../../monitoring-and-diagnostics/monitoring-overview-activity-logs.md).  
- At the time of the notification, a *self-service window* is made available. During this window, you can find which of your virtual machines are included in this wave and proactively start maintenance according to your own scheduling needs.
- After the self-service window, a *scheduled maintenance window* begins. At some point during this window, Azure schedules and applies the required maintenance to your virtual machine. 

The goal in having two windows is to give you enough time to start maintenance and reboot your virtual machine while knowing when Azure will automatically start maintenance.


You can use the Azure portal, PowerShell, REST API, and CLI to query for the maintenance windows for your VMs and start self-service maintenance.

 > [!NOTE]
 > If you try to start maintenance and the request fails, Azure marks your VM as **skipped**. You will no longer be able to use the Customer Initiated Maintenance option. Your VM will have to be rebooted by Azure during the scheduled maintenance phase.


 
## Should you start maintenance using during the self-service window?  

The following guidelines should help you to decide whether you should use this capability and start maintenance at your own time.

> [!NOTE] 
> Self-service maintenance might not be available for all of your VMs. To determine if proactive redeploy is available for your VM, look for the **Start now** in the maintenance status. Self-service maintenance is currently not available for Cloud Services (Web/Worker Role), Service Fabric, and Virtual Machine Scale Sets.


Self-service maintenance is not recommended for deployments using **availability sets** since these are highly available setups, where only one update domain is impacted at any given time. 
	- Let Azure trigger the maintenance, but be aware that the order of update domains being impacted does not necessarily happen sequentially and that there is a 30-minute pause between update domains.
	- If a temporary loss of some of your capacity (1/update domain count) is a concern, it can easily be compensated for by allocating addition instances during the maintenance period. 

**Don't** use self-service maintenance in the following scenarios: 
	- If you shut down your VMs frequently, either manually, using DevTest labs, using auto-shutdown, or following a schedule, it could revert the maintenance status and therefore cause additional downtime.
	- On short-lived VMs which you know will be deleted before the end of the maintenance wave. 
	- For workloads with a large state stored in the local (ephemeral) disk that is desired to be maintained upon update. 
	- For cases where you resize your VM often, as it could revert the maintenance status. 
	- If you have adopted scheduled events which enable proactive failover or graceful shutdown of your workload, 15 minutes before start of maintenance shutdown

**Use** self-service maintenance, if you are planning to run your VM uninterrupted during the scheduled maintenance phase and none of the counter-indications mentioned above are applicable. 

It is best to use self-service maintenance in the following cases:
	- You need to communicate an exact maintenance window to your management or end-customer. 
	- You need to complete the maintenance by a given date. 
	- You need to control the sequence of maintenance, e.g., multi-tier application to guarantee safe recovery.
	- You need more than 30 minutes of VM recovery time between two update domains (UDs). To control the time between update domains, you must trigger maintenance on your VMs one update domain (UD) at a time.



## Find VMs scheduled for maintenance using CLI

Planned maintenance information can be seen using [azure vm get-instance-view](/cli/azure/vm?view=azure-cli-latest#az_vm_get_instance_view).
 
Maintenance information is returned only if there is maintenance planned. If there is no maintenance scheduled that impacts the VM, the command does not return any maintenance information. 

```azure-cli
az vm get-instance-view -g rgName -n vmName
```

The following values are returned under MaintenanceRedeployStatus: 

| Value	| Description	|
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indicates whether you can start maintenance on the VM at this time ||
| PreMaintenanceWindowStartTime         | The beginning of the maintenance self-service window when you can initiate maintenance on your VM ||
| PreMaintenanceWindowEndTime           | The end of the maintenance self-service window when you can initiate maintenance on your VM ||
| MaintenanceWindowStartTime            | The beginning of the maintenance scheduled window in which Azure initiates maintenance on your VM ||
| MaintenanceWindowEndTime              | The end of the maintenance scheduled window in which Azure initiates maintenance on your VM ||
| LastOperationResultCode               | The result of the last attempt to initiate maintenance on the VM ||




## Start maintenance on your VM using CLI

The following call will initiate maintenance on a VM if `IsCustomerInitiatedMaintenanceAllowed` is set to true.

```azure-cli
az vm perform-maintenance rgName vmName 
```

[!INCLUDE [virtual-machines-common-maintenance-notifications](../../../includes/virtual-machines-common-maintenance-notifications.md)]

## Classic deployments

If you still have legacy VMs that were deployed using the classic deployment model, you can use CLI 1.0 to query for VMs and initiate maintenance.

Make sure you are in the correct mode to work with classic VM by typing:

```
azure config mode asm
```

To get the maintenance status of a VM named *myVM*, type:

```
azure vm show myVM 
``` 

To start maintenance on your classic VM named *myVM* in the *myService* service and *myDeployment* deployment, type:

```
azure compute virtual-machine initiate-maintenance --service-name myService --name myDeployment --virtual-machine-name myVM
```


## FAQ


**Q: Why do you need to reboot my virtual machines now?**

**A:** While the majority of updates and upgrades to the Azure platform do not impact virtual machine's availability, there are cases where we can't avoid rebooting virtual machines hosted in Azure. We have accumulated several changes which require us to restart our servers which will result in virtual machines reboot.

**Q: If I follow your recommendations for High Availability by using an Availability Set, am I safe?**

**A:** Virtual machines deployed in an availability set or virtual machine scale sets have the notion of Update Domains (UD). When performing maintenance, Azure honors the UD constraint and will not reboot virtual machines from different UD (within the same availability set).  Azure also waits for at least 30 minutes before moving to the next group of virtual machines. 

For more information about high availability, see [Regions and availability for virtual machines in Azure](regions-and-availability.MD).

**Q: How do I get notified about planned maintenance?**

**A:** A planned maintenance wave starts by setting a schedule to one or more Azure regions. Soon after, an email notification is sent to the subscription owners (one email per subscription). Additional channels and recipients for this notification could be configured using Activity Log Alerts. In case you deploy a virtual machine to a region where planned maintenance is already scheduled, you will not receive the notification but rather need to check the maintenance state of the VM.

**Q: I don't see any indication of planned maintenance in the portal, Powershell, or CLI, What is wrong?**

**A:** Information related to planned maintenance is available during a planned maintenance wave only for the VMs which are going to be impacted by it. In other words, if you see not data, it could be that the maintenance wave has already completed (or not started) or that your virtual machine is already hosted in an updated server.

**Q: Is there a way to know exactly when my virtual machine will be impacted?**

**A:** When setting the schedule, we define a time window of several days. However, the exact sequencing of servers (and VMs) within this window is unknown. Customers who would like to know the exact time for their VMs can use [scheduled events](scheduled-events.md) and query from within the virtual machine and receive a 15 minute notification before a VM reboot.

**Q: How long will it take you to reboot my virtual machine?**

**A:**  Depending on the size of your VM, reboot may take up to several minutes during the self-service maintenance window. During the Azure initiated reboots in the scheduled maintenance window, the reboot will typically take about 25 minutes. Note that in case you use Cloud Services (Web/Worker Role), Virtual Machine Scale Sets, or availability sets, you will be given 30 minutes between each group of VMs (UD) during the scheduled maintenance window.

**Q: What is the experience in the case of Cloud Services (Web/Worker Role), Service Fabric, and Virtual Machine Scale Sets?**

**A:** While these platforms are impacted by planned maintenance, customers using these platforms are considered safe given that only VMs in a single Upgrade Domain (UD) will be impacted at any given time. Self-service maintenance is currently not available for Cloud Services (Web/Worker Role), Service Fabric, and Virtual Machine Scale Sets.

**Q: I have received an email about hardware decommissioning, is this the same as planned maintenance?**

**A:** While hardware decommissioning is a planned maintenance event, we have not yet onboarded this use case to the new experience.  

**Q: I don’t see any maintenance information on my VMs. What went wrong?**

**A:** There are several reasons why you’re not seeing any maintenance information on your VMs:
1.	You are using a subscription marked as Microsoft internal.
2.	Your VMs are not scheduled for maintenance. It could be that the maintenance wave has ended, canceled or modified so that your VMs are no longer impacted by it.
3.	You don’t have the **Maintenance** column added to your VM list view. While we have added this column to the default view, customers who configured to see non-default columns must manually add the **Maintenance** column to their VM list view.

**Q: My VM is scheduled for maintenance for the second time. Why?**

**A:** There are several use cases where you will see your VM scheduled for maintenance after you have already completed your maintenance-redeploy:
1.	We have canceled the maintenance wave and restarted it with a different payload. It could be that we've detected faulted payload and we simply need to deploy an additional payload.
2.	Your VM was *service healed* to another node due to a hardware fault
3.	You have selected to stop (deallocate) and restart the  VM
4.	You have **auto shutdown** turned on for the VM


**Q: Maintenance of my availability set takes a long time, and I now see “skipped” status on some of my availability set instances. Why?** 

**A:** If you have clicked to update multiple instances in an availability set in short succession, Azure will queue these requests and starts to update only the VMs in one update domain (UD) at a time. However, since there might be a pause between update domains, the update might appear to take longer. If the update queue takes longer than 60 minutes, some instances will show the **skipped** state even if they have been updated successfully. To avoid this incorrect status, update your availability sets by clicking only on instance within one availability set and wait for the update on that VM to complete before clicking on the next VM in a different update domain.


## Next Steps

Learn how you can register for maintenance events from within the VM using [Scheduled Events](scheduled-events.md).
