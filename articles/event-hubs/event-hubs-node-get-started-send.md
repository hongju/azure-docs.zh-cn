---
title: 发送和使用 Node.js 的 Azure 事件中心接收事件 |Microsoft Docs
description: 本文提供了一个演练，说明如何创建从 Azure 事件中心发送事件的 Node.js 应用程序。
services: event-hubs
author: spelluru
manager: kamalb
ms.service: event-hubs
ms.workload: core
ms.topic: article
ms.custom: seodec18
ms.date: 04/15/2019
ms.author: spelluru
ms.openlocfilehash: e67be59e0ed78b2080986acb73a33fc87599c9d3
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539341"
---
# <a name="send-events-to-or-receive-events-from-azure-event-hubs-using-nodejs"></a>事件发送到或从使用 Node.js 的 Azure 事件中心接收事件

Azure 事件中心是一个大数据流式处理平台和事件引入服务，每秒能够接收和处理数百万个事件。 事件中心可以处理和存储分布式软件和设备生成的事件、数据或遥测。 可以使用任何实时分析提供程序或批处理/存储适配器转换和存储发送到数据中心的数据。 有关事件中心的详细概述，请参阅[事件中心概述](event-hubs-about.md)和[事件中心功能](event-hubs-features.md)。

本教程介绍如何创建 Node.js 应用程序发送到事件或从事件中心接收事件。

> [!NOTE]
> 可以从 [GitHub](https://github.com/Azure/azure-event-hubs-node/tree/master/client) 下载此用作示例的快速入门，将 `EventHubConnectionString` 和 `EventHubName` 字符串替换为事件中心值，并运行它。 或者，可以按照本教程中的步骤创建自己的解决方案。

## <a name="prerequisites"></a>必备组件

若要完成本教程，需要具备以下先决条件：

- 有效的 Azure 帐户。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- Node.js 版本 8.x 和更高版本。 从 [https://nodejs.org](https://nodejs.org) 下载最新的 LTS 版本。
- Visual Studio Code（推荐使用）或任何其他 IDE
- **创建事件中心命名空间和事件中心**。 第一步是使用 [Azure 门户](https://portal.azure.com)创建事件中心类型的命名空间，并获取应用程序与事件中心进行通信所需的管理凭据。 若要创建命名空间和事件中心，请按照[本文](event-hubs-create.md)中的程序进行操作，然后继续执行本教程的以下步骤。 然后，按照文章中的说明获取有关事件中心命名空间的连接字符串：[获取连接字符串](event-hubs-get-connection-string.md#get-connection-string-from-the-portal)。 本教程后面的步骤将使用此连接字符串。


### <a name="install-npm-package"></a>安装 npm 包
若要安装[事件中心的 npm 包](https://www.npmjs.com/package/@azure/event-hubs)，打开命令提示符具有`npm`在其路径中，将目录更改为你想要具有您的示例，然后运行此命令的文件夹

```shell
npm install @azure/event-hubs
```

若要安装[事件处理程序主机的 npm 包](https://www.npmjs.com/package/@azure/event-processor-host)，请运行以下命令改为

```shell
npm install @azure/event-processor-host
```

## <a name="send-events"></a>发送事件

本部分演示如何创建 Node.js 应用程序将事件发送到事件中心。 

1. 打开你喜爱的编辑器，如[Visual Studio Code](https://code.visualstudio.com)。 
2. 创建一个名为文件`send.js`并粘贴以下代码到它。
    ```javascript
    const { EventHubClient } = require("@azure/event-hubs");

    // Define connection string and the name of the Event Hub
    const connectionString = "";
    const eventHubsName = "";

    async function main() {
      const client = EventHubClient.createFromConnectionString(connectionString, eventHubsName);

      for (let i = 0; i < 100; i++) {
        const eventData = {body: `Event ${i}`};
        console.log(`Sending message: ${eventData.body}`);
        await client.send(eventData);
      }

      await client.close();
    }

    main().catch(err => {
      console.log("Error occurred: ", err);
    });
    ```
3. 在上面的代码中输入连接字符串和事件中心的名称
4. 然后运行命令`node send.js`在命令提示符中执行此文件。 这会将 100 个事件发送到事件中心

祝贺你！ 现在已向事件中心发送事件。


## <a name="receive-events"></a>接收事件

本部分演示如何创建 Node.js 应用程序从单个分区的默认使用者组中的事件中心接收事件。 

1. 打开你喜爱的编辑器，如[Visual Studio Code](https://code.visualstudio.com)。 
2. 创建一个名为文件`receive.js`并粘贴以下代码到它。
    ```javascript
    const { EventHubClient, delay } = require("@azure/event-hubs");

    // Define connection string and related Event Hubs entity name here
    const connectionString = "";
    const eventHubsName = "";

    async function main() {
      const client = EventHubClient.createFromConnectionString(connectionString, eventHubsName);
      const allPartitionIds = await client.getPartitionIds();
      const firstPartitionId = allPartitionIds[0];

      const receiveHandler = client.receive(firstPartitionId, eventData => {
        console.log(`Received message: ${eventData.body} from partition ${firstPartitionId}`);
      }, error => {
        console.log('Error when receiving message: ', error)
      });

      // Sleep for a while before stopping the receive operation.
      await delay(15000);
      await receiveHandler.stop();

      await client.close();
    }

    main().catch(err => {
      console.log("Error occurred: ", err);
    });
    ```
3. 在上述代码中输入连接字符串和事件中心的名称。
4. 然后运行命令`node receive.js`在命令提示符中执行此文件。 这将从事件中心中的默认使用者组的分区之一接收事件

祝贺你！ 现在已从事件中心接收事件。

## <a name="receive-events-using-event-processor-host"></a>使用事件处理程序主机接收事件

本部分演示如何使用 Azure 从事件中心接收事件[EventProcessorHost](event-hubs-event-processor-host.md) Node.js 应用程序中。 EventProcessorHost (EPH) 通过在事件中心的使用者组中的所有分区中创建接收器，帮助你高效地从事件中心接收事件。 它定期在 Azure 存储 Blob 中为收到的消息的元数据创建检查点。 使用此方式，可以很容易地在以后的某个时间从离开的位置继续接收消息。

1. 打开你喜爱的编辑器，如[Visual Studio Code](https://code.visualstudio.com)。 
2. 创建一个名为文件`receiveAll.js`并粘贴以下代码到它。
    ```javascript
    const { EventProcessorHost, delay } = require("@azure/event-processor-host");

    // Define connection string and related Event Hubs entity name here
    const eventHubConnectionString = "";
    const eventHubName = "";
    const storageConnectionString = "";

    async function main() {
      const eph = EventProcessorHost.createFromConnectionString(
        "my-eph",
        storageConnectionString,
        "my-storage-container-name",
        eventHubConnectionString,
        {
          eventHubPath: eventHubName,
          onEphError: (error) => {
            console.log("[%s] Error: %O", error);
          }
        }
      );


      eph.start((context, eventData) => {
        console.log(`Received message: ${eventData.body} from partition ${context.partitionId}`);
      }, error => {
        console.log('Error when receiving message: ', error)
      });

      // Sleep for a while before stopping the receive operation.
      await delay(15000);
      await eph.stop();
    }

    main().catch(err => {
      console.log("Error occurred: ", err);
    });

    ```
3. 为 Azure Blob 存储连接字符串以及在上述代码中输入连接字符串和事件中心的名称
4. 然后运行命令`node receiveAll.js`在命令提示符中执行此文件。

祝贺你！ 现在已从事件中心使用事件处理器主机接收事件。 这将从事件中心中的默认使用者组的所有分区接收事件

## <a name="next-steps"></a>后续步骤
请阅读以下文章：

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [功能和 Azure 事件中心内的术语](event-hubs-features.md)
- [事件中心常见问题解答](event-hubs-faq.md)
- 请查看有关其他 Node.js 示例[事件中心](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples)并[事件处理程序主机](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-processor-host/samples)GitHub 上
