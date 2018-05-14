---
title: Container workloads on Azure Batch | Microsoft Docs
description: Learn how to run applications from container images on Azure Batch.
services: batch
author: dlepow
manager: jeconnoc

ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.workload: na
ms.date: 05/07/2018
ms.author: danlep

---

# Run container applications on Azure Batch

Azure Batch lets you run and scale large numbers of batch computing jobs on Azure. Until now, Batch tasks have run directly on virtual machines (VMs) in a Batch pool, but now you can set up a Batch pool to run tasks in Docker containers. This article shows you how to use the Batch .NET SDK to create a pool of compute nodes that support running container tasks, and how to run container tasks on the pool.

Using containers provides an easy way to run Batch tasks without having to manage an environment and dependencies to run applications. Containers deploy applications as lightweight, portable, self-sufficient units that can run in a variety of environments. For example, you can build and test a container locally, then upload the container image to a registry in Azure or elsewhere. The container deployment model ensures that the runtime environment of your application is always correctly installed and configured, regardless of where you host the application. Container-based tasks in Batch can also take advantage of features of non-container tasks, including application packages and management of resource files and output files. 

This article assumes familiarity with Docker container concepts and how to create a Batch pool and job using the .NET SDK. The code snippets are meant to be used in a client application similar to the [DotNetTutorial sample](batch-dotnet-get-started.md), and are examples of code you would need to support container applications in Batch.


## Prerequisites

* **SDK versions**: The Batch SDKs support container images as of the following versions:
    * Batch REST API version 2017-09-01.6.0
    * Batch .NET SDK version 8.0.0
    * Batch Python SDK version 4.0
    * Batch Java SDK version 3.0
    * Batch Node.js SDK version 3.0

* **Accounts**: In your Azure subscription, you need to create a Batch account and optionally an Azure Storage account.

* **A supported VM image**: Containers are only supported in pools created with the Virtual Machine Configuration, from images detailed in the following section, "Supported virtual machine images." If you provide a custom image, your application must use Azure Active Directory [(Azure AD) authentication](batch-aad-auth.md) in order to run container-based workloads. 


## Supported virtual machine images

You need to use a supported Windows or Linux image to create a pool of VM compute nodes for container workloads.

### Windows images

For Windows container workloads, Batch currently supports custom images that you create from VMs running Docker on Windows, or you can use the Windows Server 2016 Datacenter with Containers image from the Azure Marketplace. This image is compatible with the `batch.node.windows amd64` node agent SKU ID. The type of container supported is currently limited to Docker.

### Linux images

For Linux container workloads, Batch currently supports only custom images that you create from VMs running Docker on the following Linux distributions: Ubuntu 16.04 LTS or CentOS 7.3. If you choose to provide your own custom Linux image, see the instructions in [Use a managed custom image to create a pool of virtual machines](batch-custom-images.md).

For Docker support, install [Docker Community Edition (CE)](https://www.docker.com/community-edition) or [Docker Enterprise Edition (EE)](https://www.docker.com/enterprise-edition).

If you want to take advantage of the GPU performance of Azure NC or NV VM sizes, you need to install NVIDIA drivers on the image. Also, you need to install and run the Docker Engine Utility for NVIDIA GPUs, [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker).

To access the Azure RDMA network, use RDMA-capable VM sizes, such as A8, A9, H16r, H16mr, or NC24r. Necessary RDMA drivers are installed in the CentOS 7.3 HPC and Ubuntu 16.04 LTS images from the Azure Marketplace. Additional configuration may be needed to run MPI workloads. See [Use RDMA-capable or GPU-enabled instances in Batch pool](batch-pool-compute-intensive-sizes.md).


## Limitations

* Batch provides RDMA support only for containers running on Linux pools.


## Authenticate using Azure Active Directory

If you use a custom VM image to create the Batch pool, your client application must authenticate using Azure AD integrated authentication (shared key authentication does not work). Before running the application, make sure you register it in Azure AD to establish an identity for it and to specify its permissions to other applications.

Also, when you use a custom VM image, you need to grant IAM access control to the application to access the VM image. In the Azure portal, click **All resources**, select the container image, and from the **Access control (IAM)** section of the image page, click **Add**. In the **Add permissions** page, specify a **Role**, in **Assign access to**, select **Azure AD user, group, or application**, then in **Select** enter the application name.

In your application, pass an Azure AD authentication token when you create the Batch client. If you are developing using the Batch .NET SDK, use [BatchClient.Open](/dotnet/api/microsoft.azure.batch.batchclient.open#Microsoft_Azure_Batch_BatchClient_Open_Microsoft_Azure_Batch_Auth_BatchTokenCredentials_), as described in [Authenticate Batch service solutions with Active Directory](batch-aad-auth.md).


## Reference a VM image for pool creation

In your application code, provide a reference to the VM image to use in creating the compute nodes of the pool. You do this by creating an [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) object. You can specify the image to use in one of the following ways:

* If you are using a custom image, provide an Azure Resource Manager resource identifier for the virtual machine image. The image identifier has a path format as shown in the following example:

  ```csharp
  // Provide a reference to a custom image using an image ID
  ImageReference imageReference = new ImageReference("/subscriptions/<subscription-ID>/resourceGroups/<resource-group>/providers/Microsoft.Compute/images/<imageName>");
  ```

    To obtain this image ID from the Azure portal, open **All resources**, select the custom image, and from the **Overview** section of the image page, copy the path in **Resource ID**.

* If you are using an [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute?page=1&subcategories=windows-based) image, provide a group of parameters describing the image: the publisher, the offer type, SKU, and version of the image, as listed in [List of virtual machine images](batch-linux-nodes.md#list-of-virtual-machine-images):

  ```csharp
  // Provide a reference to an Azure Marketplace image for
  // "Windows Server 2016 Datacenter with Containers"
  ImageReference imageReference = new ImageReference(
    publisher: "MicrosoftWindowsServer",
    offer: "WindowsServer",
    sku: "2016-Datacenter-with-Containers",
    version: "latest");
  ```


## Container configuration for Batch pool

To enable a Batch pool to run container workloads, you must specify [ContainerConfiguration](/dotnet/api/microsoft.azure.batch.containerconfiguration) settings in the pool's [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) object.

You can create a container-enabled pool with or without prefetched container images, as shown in the following examples. The pull (or prefetch) process lets you pre-load container images either from Docker Hub or another container registry on the Internet. The advantage of prefetching container images is that when tasks first start running they don't have to wait for the container image to download. The container configuration pulls container images to the VMs when the pool is created. Tasks that run on the pool can then reference the list of container images and container run options.



### Pool without prefetched container images

To configure a container-enabled pool without prefetched container images, define `ContainerConfiguration` and `VirtualMachineConfiguration` objects as shown in the following example. This and the following examples assume that you are using a custom Ubuntu 16.04 LTS image with Docker Engine installed.

```csharp
// Specify container configuration. This is required even though there are no prefetched images.
ContainerConfiguration containerConfig = new ContainerConfiguration();

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    containerConfiguration: containerConfig,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);

// Commit pool creation
pool.Commit();
```


### Prefetch images for container configuration

To prefetch container images on the pool, add the list of container images (`containerImageNames`) to the `ContainerConfiguration`, and give the image list a name. The following example assumes that you are using a custom Ubuntu 16.04 LTS image, and prefetch a TensorFlow image from [Docker Hub](https://hub.docker.com). This example includes a start task that runs in the VM host on the pool nodes. You might run a start task in the host, for example, to mount a file server that can be accessed from the containers.

```csharp
// Specify container configuration, prefetching Docker images
ContainerConfiguration containerConfig = new ContainerConfiguration(
    containerImageNames: new List<string> { "tensorflow/tensorflow:latest-gpu" } );

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    containerConfiguration: containerConfig,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");

// Set a native host command line start task
StartTask startTaskNative = new StartTask( CommandLine: "<native-host-command-line>" );

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration, startTaskContainer);

// Commit pool creation
pool.Commit();
```


### Prefetch images from a private container registry

You can also prefetch container images by authenticating to a private container registry server. In the following example, the `ContainerConfiguration` and `VirtualMachineConfiguration` objects use a custom Ubuntu 16.04 LTS image and prefetch a private TensorFlow image from a private Azure container registry.

```csharp
// Specify a container registry
ContainerRegistry containerRegistry = new ContainerRegistry (
	registryServer: "myContainerRegistry.azurecr.io",
    username: "myUserName",
    password: "myPassword");

// Create container configuration, prefetching Docker images from the container registry
ContainerConfiguration containerConfig = new ContainerConfiguration(
    containerImageNames: new List<string> {
        "myContainerRegistry.azurecr.io/tensorflow/tensorflow:latest-gpu" },
    containerRegistries: new List<ContainerRegistry> { containerRegistry } );

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    containerConfiguration: containerConfig,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);

// Commit pool creation
pool.Commit();
```


## Container settings for the task

To run container tasks on the compute nodes, you must specify container-specific settings such as task run options, images to use, and registry.

Use the `ContainerSettings` property of the task classes to configure container-specific settings. These settings are defined by the [TaskContainerSettings](/dotnet/api/microsoft.azure.batch.taskcontainersettings) class.

If you run tasks on container images, the [cloud task](/dotnet/api/microsoft.azure.batch.cloudtask) and [job manager task](/dotnet/api/microsoft.azure.batch.cloudjob.jobmanagertask) require container settings. However, the [start task](/dotnet/api/microsoft.azure.batch.starttask), [job preparation task](/dotnet/api/microsoft.azure.batch.cloudjob.jobpreparationtask), and [job release task](/dotnet/api/microsoft.azure.batch.cloudjob.jobreleasetask) do not require container settings (that is, they can run within a container context or directly on the node).

When you configure the container settings, all directories recursively below the `AZ_BATCH_NODE_ROOT_DIR` (the root of Azure Batch directories on the node) are mapped into the container, all task environment variables are mapped into the container, and the task command line is executed in the container.

The code example in [Prefetch images for container configuration](#prefetch-images-for-container-configuration) showed how you specify a container configuration for a start task. The following code example shows how you specify container configuration for a cloud task:

```csharp
// Simple container task command

string cmdLine = "<my-command-line>";

TaskContainerSettings cmdContainerSettings = new TaskContainerSettings (
    imageName: "tensorflow/tensorflow:latest-gpu",
    containerRunOptions: "--rm --read-only"
    );

CloudTask containerTask = new CloudTask (
    id: "Task1",
    containerSettings: cmdContainerSettings,
    commandLine: cmdLine); 
```


## Next steps

* Also see the [Batch Shipyard](https://github.com/Azure/batch-shipyard) toolkit for easy deployment of container workloads on Azure Batch through [Shipyard recipes](https://github.com/Azure/batch-shipyard/tree/master/recipes).

* For more information on installing and using Docker CE on Linux, see the [Docker](https://docs.docker.com/engine/installation/) documentation.

* For more information on using custom images, see [Use a managed custom image to create a pool of virtual machines ](batch-custom-images.md).
