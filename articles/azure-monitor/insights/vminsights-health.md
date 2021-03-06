---
title: 使用用于 VM 的 Azure Monitor（预览版）监视虚拟机运行状况 | Microsoft Docs
description: 本文介绍如何使用用于 VM 的 Azure Monitor 来了解虚拟机和基础操作系统的运行状况。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/12/2019
ms.author: magoedte
ms.openlocfilehash: f2a0d64da5a88e82c0ae1fd893af52f2070268f8
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60402059"
---
# <a name="understand-the-health-of-your-azure-virtual-machines"></a>了解 Azure 虚拟机的运行状况

Azure 包括多个单独执行特定角色或任务在监视空间，但还未提供 Azure 虚拟机上托管的操作系统的详细运行状况角度的服务。 虽然可以使用 Azure Monitor 的不同情况进行监视，它并不旨在模型和表示核心组件的运行状况或虚拟机的总体运行状况。 用于 VM 的 Azure Monitor 运行状况功能可以使用一个代表核心组件及其关系的模型、指定如何度量这些组件的运行状况的条件，来主动监视 Windows 或 Linux 来宾 OS 的可用性和性能，并在检测到不正常状况时发出警报。  

可以直接在虚拟机中或者某个资源组中的所有 VM 上，使用用于 VM 的 Azure Monitor 运行状况功能的两个透视图查看 Azure VM 和基础操作系统的总体运行状况。

本文将帮助你了解如何快速评估、调查和解决检测到的运行状况问题。

有关配置用于 VM 的 Azure Monitor 的信息，请参阅[启用用于 VM 的 Azure Monitor](vminsights-onboard.md)。

## <a name="monitoring-configuration-details"></a>监视配置详细信息

本部分概述为了监视 Azure Windows 和 Linux 虚拟机而定义的默认运行状况条件。 所有运行状况条件均预配置为在满足不正常条件时发出警报。 

### <a name="windows-vms"></a>Windows VM

- 可用内存 (MB) 
- 每次写入的平均磁盘秒数（逻辑磁盘）
- 每次写入的平均磁盘秒数（磁盘）
- 每次读取的平均逻辑磁盘秒数
- 每次传输的平均逻辑磁盘秒数
- 每次读取的平均磁盘秒数
- 每次传输的平均磁盘秒数
- 当前磁盘队列长度（逻辑磁盘）
- 当前磁盘队列长度（磁盘）
- 磁盘空闲时间百分比
- 文件系统错误或损坏
- 逻辑磁盘可用空间 (%) 不足
- 逻辑磁盘可用空间 (MB) 不足
- 逻辑磁盘空闲时间百分比
- 每秒内存页面数
- 读取使用的带宽百分比
- 使用的带宽百分比总计
- 写入使用的带宽百分比
- 使用的已提交内存百分比
- 磁盘空闲时间百分比
- DHCP 客户端服务运行状况
- DNS 客户端服务运行状况
- RPC 服务运行状况
- 服务器服务运行状况
- CPU 利用率百分比总计
- Windows 事件日志服务运行状况
- Windows 防火墙服务运行状况
- Windows 远程管理服务运行状况

### <a name="linux-vms"></a>Linux VM
- 磁盘平均值磁盘秒数/传输 
- 磁盘平均值磁盘秒数/读取 
- 磁盘平均值磁盘秒数/写入 
- 磁盘运行状况
- 逻辑磁盘可用空间
- 逻辑磁盘可用空间百分比
- 逻辑磁盘可用 Inode 百分比
- 网络适配器运行状况
- 处理器时间百分比总计
- 操作系统可用内存 (MB)

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com)。 

## <a name="introduction-to-health-experience"></a>运行状况体验简介

在深入探讨如何针对单个虚拟机或 VM 组使用运行状况功能之前，我们必须提供相关的简介，使读者了解信息的显示方式，以及要显示哪些可视化效果。  

## <a name="view-health-directly-from-a-virtual-machine"></a>直接从虚拟机查看运行状况 

若要查看 Azure VM 的运行状况，请在虚拟机的左窗格中选择“见解(预览版)”。 VM 见解页上默认会打开“运行状况”，其中显示了 VM 的运行状况视图。  

![用于 VM 的 Azure Monitor 针对所选 Azure 虚拟机显示的运行状况概述](./media/vminsights-health/vminsights-directvm-health.png)

在“运行状况”选项卡上，“来宾 VM 运行状况”部分下的表格显示了虚拟机的当前运行状况，以及不正常的组件引发的 VM 运行状况警报总数。 有关详细信息，请参阅[警报](#alerts)有关其他详细信息的警报功能体验。  

下表介绍了为 VM 定义的运行状况状态： 

|图标 |运行状况 |含义 |
|-----|-------------|------------|
| |Healthy |如果运行状况状态在定义的运行状况条件内，则为正常状态，表示没有检测到任何 VM 问题，它在按要求正常运行。 与父汇总监视器，运行状况汇总和它反映了子的最佳和最差状态。|
| |严重 |如果运行状况状态不在定义的运行状况条件内，则为严重状态，表示检测到一个或多个关键问题，需要解决这些问题才能恢复正常功能。 与父汇总监视器，运行状况汇总和它反映了子的最佳和最差状态。|
| |警告 |如果运行状况状态介于定义的运行状况条件的两个阈值之间，其中一个阈值指示“警告”状态，另一个指示“严重”状态（可以配置三个运行状况状态阈值），或者检测到非严重性问题，若不解决可能会导致严重问题，这种情况为警告状态。 父汇总监视器，如果一个或多个子级是处于警告状态，则父级将反映*警告*状态。 如果一个子级处于“严重”状态，另一个子级处于“警告”状态，则父汇总将显示“严重”运行状况。|
| |Unknown |如果出于多种原因（例如，无法收集数据、服务未初始化，等等）而无法计算的运行状况状态，则运行状况处于“未知”状态。此运行状况状态不可配置。| 

选择“查看运行状况诊断”会打开一个页面，其中显示了 VM 的所有组件、关联的运行状况条件、状态更改，以及与 VM 相关的监视组件遇到的其他重大问题。 有关详细信息，请参阅[运行状况诊断](#health-diagnostics)。 

“组件运行状况”部分下的表格显示了运行状况条件针对“CPU”、“内存”、“磁盘”和“网络”这几个具体方面监视的主要性能类别的总体运行状况。  选择其中一个组件会打开一个页面，其中列出了该组件的各个运行状况条件监视方面以及相应的运行状况。  

当从运行 Windows 操作系统的 Azure VM 访问运行状况中, 五种核心的 Windows 服务显示在部分顶部的运行状况状态**核心服务运行状况**。  选择任一服务会打开一个页面，其中列出了该组件的运行状况条件监视信息及其运行状况。  单击运行状况条件的名称会打开属性窗格，在其中可以查看配置详细信息，包括是否为运行状况条件定义了相应的 Azure Monitor 警报。 若要了解详细信息，请参阅[运行状况诊断和使用运行状况条件](#health-diagnostics)。  

## <a name="aggregate-virtual-machine-perspective"></a>聚合虚拟机透视图

若要查看资源组中所有虚拟机的运行状况集合，请在门户上的导航列表中，依次选择“Azure Monitor”、“虚拟机(预览)”。  

![Azure Monitor 中的 VM 见解监视视图](./media/vminsights-health/vminsights-aggregate-health.png)

从“订阅”和“资源组”下拉列表中，选择包含组相关 VM 的相应资源组，以查看其报告的运行状况。  你的选择仅适用于运行状况功能，并不会延续到性能或映射。

在“运行状况”选项卡上，可了解以下信息：

* 有多少个 VM 处于严重或不正常状态，有多少个 VM 处于正常状态或未提交数据（称为未知状态）？
* 哪些 VM 在报告不正常状态，这样的 VM 有多少个（同时列出操作系统 (OS)）？
* 有多少个 VM 由于检测到处理器、磁盘、内存或网络适配器的问题而显示为不正常状态（按运行状况分类）？  
* 有多少个 VM 由于检测到核心操作系统服务的问题而显示为不正常状态（按运行状况分类）？

在此处可以快速识别主动监视 VM 的运行状况条件检测到的最严重问题，以及查看 VM 运行状况警报详细信息和相关的知识库文章，以帮助诊断和修正问题。  选择任一严重性可打开按该严重性筛选的“[所有警报](../../azure-monitor/platform/alerts-overview.md#all-alerts-page)”页。

“按操作系统列出的 VM 分发版”列表根据 Windows 版本或 Linux 分发版及其版本显示 VM。 在每个操作系统类别中，VM 根据 VM 的运行状况进一步划分。 

![VM 见解：虚拟机分发版透视图](./media/vminsights-health/vminsights-vmdistribution-by-os.png)

可以单击任何一个列项（“VM 计数”、“严重”、“警告”、“正常”或“未知”）深入到“虚拟机”页，以查看与所选列匹配的筛选结果列表。 例如，如果我们想要查看所有运行“Red Hat Enterprise Linux 版本 7.5”的 VM，可单击该 OS 对应的“VM 计数”值，随后打开的页面会列出与该筛选器匹配的虚拟机及其当前已知的运行状况。  

![Red Hat Linux VM 的汇总信息示例](./media/vminsights-health/vminsights-rollup-vm-rehl-01.png)
 
在“虚拟机”页上，如果选择“VM 名称”列下的某个 VM 的名称，则会定向到 VM 实例页，其中包含了已识别到的、影响所选 VM 的警报和运行状况条件问题的更多详细信息。  在此页中，可以单击左上角的“运行状况”图标筛选运行状况详细信息，以查看哪些组件不正常，或者查看不正常组件引发的 VM 运行状况警报（已按警报严重性分类）。    

在 VM 列表视图中，单击某个 VM 的名称会打开所选 VM 的“运行状况”页，与直接在 VM 中选择“见解(预览版)”时所看到的页面相同。

![所选 Azure 虚拟机的 VM 见解](./media/vminsights-health/vminsights-directvm-health.png)

此页显示该虚拟机的“运行状况”汇总信息，以及按严重性分类的“警报”。这些警报显示了运行状况条件将运行状况从正常更改为不正常时引发的 VM 运行状况警报。  选择“出现严重状况的 VM”会打开一个页面，其中列出了处于严重运行状况的一个或多个 VM。  在该列表中单击某个 VM 的运行状况会显示该 VM 的“运行状况诊断”视图。  在此视图中可以找出哪个运行状况条件反映了运行状况问题。 当“运行状况诊断”页打开时，它会显示 VM 的所有组件，以及这些组件的关联运行状况条件和当前运行状况。 有关详细信息，请参阅[运行状况诊断](#health-diagnostics)。  

选择“查看所有运行状况条件”会打开一个页面，其中显示了适用于此功能的所有运行状况条件的列表。  可根据以下选项进一步筛选信息：

* **类型** – 有三种类型的运行状况条件可以评估状况，并汇总受监视 VM 的总体运行状况。  
    a. **单元** – 度量虚拟机的某个方面。 此运行状况条件类型可以检查性能计数器以确定组件的性能、运行脚本来执行综合事务，或者监视指示出错的事件。  默认情况下，筛选器设置为单元。  
    b. **依赖项** - 提供不同实体之间的运行状况汇总。 此运行状况条件允许一个实体的运行状况依赖于另一种实体的运行状况来成功完成操作。  
    c. **聚合** - 提供类似运行状况条件的合并运行状况。 单元和依赖项运行状况条件通常在聚合运行状况条件下配置。 除了针对面向某个实体的许多不同运行状况条件提供更好的一般性组织方式以外，聚合运行状况条件还为不同类别的实体提供唯一的运行状况。

* **类别** - 用于将类似类型的条件分组，以实现报告目的的运行状况条件类型。  类别为“可用性”或“性能”。

可以单击“不正常组件”列下的某个值，以进一步深入查看哪些实例不正常。  此页面上有一个表列出了处于严重运行状况的组件。    

## <a name="health-diagnostics"></a>运行状况诊断

在“运行状况诊断”页中，可以查看 VM 的运行状况模型，列出 VM 的所有组件、关联的运行状况条件、状态更改，以及与 VM 相关的监视对象识别的其他重大问题。

![VM 的“运行状况诊断”页示例](./media/vminsights-health/health-diagnostics-page-01.png)

可通过以下方式启动运行状况诊断。

* 通过 Azure Monitor 的聚合 VM 透视图中显示的所有 VM 的汇总运行状况。  在“运行状况”页上，单击“来宾 VM 运行状况”部分下的“严重”、“警告”、“正常”或“未知”运行状况对应的图标，以深入到列出了所有与该筛选类别匹配的 VM 的页面。  单击“运行状况”列中的值会打开限定为该特定 VM 的运行状况诊断。      

* 通过 Azure Monitor 的聚合 VM 透视图中的操作系统。 在“VM 分发版”下，选择任何一个列值会打开“虚拟机”页，并在表中返回与筛选的类别匹配的列表。  单击“运行状况状态”列下的值会打开所选 VM 的运行状况诊断。    
 
* 在 Azure Monitor 的 VM“运行状况”选项卡上的来宾 VM 中，选择“查看运行状况诊断” 

运行状况诊断将运行状况信息组织成以下类别： 

* 可用性
* 性能
 
所有运行状况条件定义特定的组件，如逻辑磁盘、 CPU、 等不筛选上的两个类别 （这是所有条件的全部视图），可以查看或选择时按这两种类别筛选结果**可用性**或**性能**页上的选项。 此外，可以看到条件的类别在其旁边**运行状况条件**列。 如果条件不符合所选的类别，它将显示消息**没有可用于所选类别的运行状况条件**中**运行状况条件**列。  

运行状况条件的状态按以下四种状态之一进行定义：“严重”、“警告”、“正常”和“未知”。 前三列是可配置，这意味着您可以修改直接从运行状况条件配置窗格的监视器的阈值或使用 Azure Monitor REST API[更新监视操作](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/monitors/update)。 “未知”不可配置，保留用于特定场景。  

“运行状况诊断”页包含三个主要部分：

* 组件模型 
* 运行状况条件
* 状态更改 

![“运行状况诊断”页包含的部分](./media/vminsights-health/health-diagnostics-page-02.png)

### <a name="component-model"></a>组件模型

“运行状况诊断”页的最左侧列是“组件模型”。 与 VM 关联的所有组件及其当前运行状况状态显示在此列中。 

在以下示例中，已发现的组件是磁盘、逻辑磁盘、处理器、内存和操作系统。 可在此列中发现并显示这些组件的多个实例。 例如，下图显示 VM 有两个逻辑磁盘实例（C: 和 D:），它们均处于正常状态。  

![运行状况诊断中显示的示例组件模型](./media/vminsights-health/health-diagnostics-page-component.png)

### <a name="health-criteria"></a>运行状况条件

“运行状况诊断”页的中间列是“运行状况条件”列。 为 VM 定义的运行状况模型在分层树中显示。 VM 的运行状况模型包括单元和聚合运行状况条件。  

![运行状况诊断中显示的示例运行状况条件](./media/vminsights-health/health-diagnostics-page-healthcriteria.png)

运行状况条件使用某种条件（可能是一个阈值、实体状态等）来度量受监视实例的运行状况。如前面所述，运行状况条件具有两个或三个可配置的运行状况状态阈值。 在任意给定时间，运行状况条件只能处于一种潜在状态。 

目标的总体运行状况取决于运行状况模型中为该目标定义的每个运行状况条件的运行状况。 它是直接针对目标运行状况条件以汇总到聚合运行状况条件通过目标组件为目标的运行状况条件的组合。 “运行状况诊断”页的“运行状况条件”部分演示了此层次结构。 运行状况汇总策略是聚合运行状况条件配置的一部分（默认值设置为“最差”）。 您可以找到一组默认的部分下的此功能的一部分运行的运行状况条件[监视配置的详细信息](#monitoring-configuration-details)，并且可以使用 Azure Monitor REST API[监视实例的资源的列表操作](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/monitorinstances/listbyresource)若要获取所有运行状况条件和针对 Azure VM 资源运行其详细的配置的列表。  

单击最右侧的椭圆形链接并选择“显示详细信息”打开配置窗格，可以修改运行状况条件“单元”类型的配置。 

![配置运行状况条件的示例](./media/vminsights-health/health-diagnostics-vm-example-02.png)

在所选的运行状况条件的配置窗格中，使用示例“每次写入的平均磁盘秒数”，可以使用不同数值来配置其阈值。 它是一个双状态监视器，意味着它只有正常和警告两种状态。 其他运行状况条件可能有三种状态，在这种情况下，可以配置警告和严重运行状况状态阈值的值。 您还可以修改使用 Azure Monitor REST API 的阈值[更新监视操作](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/monitors/update)。

>[!NOTE]
>将运行状况条件配置更改应用到一个实例，即可将其应用到所有受监视的实例。  例如，如果选择“磁盘 -1 D:”并修改“每次写入的平均磁盘秒数”阈值，则这项更改不仅会应用到该实例，而且会应用到在 VM 上发现和监视的所有其他磁盘实例。
>

![配置单元监视器的运行状况条件的示例](./media/vminsights-health/health-diagnostics-criteria-config-01.png)

如果想详细了解运行状况指示符，可以参阅知识文章，帮你查明问题、原因和解决方法。 单击页面上的“查看信息”链接，这将在浏览器中打开一个新的选项卡，其中显示了特定的知识文章。 在任何时候，都可以在[此处](https://docs.microsoft.com/azure/monitoring/infrastructure-health/)查看随用于 VM 的 Azure Monitor 运行状况功能提供的所有运行状况条件知识文章。
  
### <a name="state-changes"></a>状态更改

“运行状况诊断”页中最右侧的列是“状态更改”。 其中列出了与在“运行状况条件”部分选择的运行状况条件相关联的所有状态更改，或者从表的“组件模型”或“运行状况条件”列中选择的某个 VM 的状态更改。 

![运行状况诊断中显示的状态更改示例](./media/vminsights-health/health-diagnostics-page-statechanges.png)

此部分包含运行状况条件的状态，其顶部是关联的时间（按最新状态排序）。   

### <a name="association-of-component-model-health-criteria-and-state-change-columns"></a>关联的组件模型、 运行状况条件和状态更改的列 

这三个列彼此互相链接。 当你在“组件模型”部分中选择一个已发现的实例时，“运行状况条件”部分会根据该组件视图进行筛选，而“状态更改”部分会相应地根据所选运行状况条件进行更新。 

![选择受监视实例和结果的示例](./media/vminsights-health/health-diagnostics-vm-example-01.png)

在上面的示例中，当你选择“磁盘 - 1 D:”时，运行状况条件树会根据“磁盘 - 1 D:”进行筛选。 “状态更改”列根据“磁盘 - 1 D:”的可用性显示状态更改。 

若要查看更新的运行状况状态，可以单击“刷新”链接来刷新“运行状况诊断”页。  如果基于预定义的轮询间隔对运行状况条件的运行状况做了更新，则此任务可以避免等待显示最新运行状况。  **运行状况条件状态**是一个筛选器，用于根据所选运行状况状态（*正常*、*警告*、*严重*、*未知*和*所有*）来限定结果范围。  右上角的“上次更新时间”表示上次刷新“运行状况诊断”页的时间。  

## <a name="alerts"></a>警报

用于 VM 的 Azure Monitor 运行状况功能与 [Azure 警报](../../azure-monitor/platform/alerts-overview.md)相集成，当预定义的运行状况条件在检测到相应状况后从正常更改为不正常状态时，会引发警报。 警报按严重性分类 - 严重性 0 到 4，严重性 0 表示最高严重性级别。 

警报不与相关联的操作组，以触发警报时通知你。 订阅所有者需要配置通知的步骤[本部分后面](#configure-alerts)。   

“运行状况”仪表板的“警报”部分下显示了按严重性分类的 VM 运行状况警报的总数。 选择警报总数或者对应于某个严重性级别的编号时，“警报”页将会打开并列出与所选内容匹配的所有警报。  例如，如果选择了对应于“严重性级别 1”的行，则会看到以下视图：

![所有严重性级别 1 警报的示例](./media/vminsights-health/vminsights-sev1-alerts-01.png)

在“警报”页面上，不仅只显示与你的选择相符的警报，还按“资源类型”进行筛选以仅显示由虚拟机资源引发的运行状况警报。  它将反映在警报，列下的列表**目标资源**，其中显示了 Azure VM 已满足特定的运行状况条件的不正常条件时引发警报。  

中的其他资源类型或服务的警报不应包含在此视图中，如日志警报基于查询记录在日志或指标警报将通常从默认的 Azure Monitor 查看[的所有警报](../../azure-monitor/platform/alerts-overview.md#all-alerts-page)页。 

可以通过选择页面顶部的下拉菜单中的值，来对此视图进行筛选。

|列 |描述 | 
|-------|------------| 
|订阅 |选择 Azure 订阅。 只有选定订阅中的警报才会包含在视图中。 | 
|资源组 |选择单个资源组。 只有包含选定资源组中的目标的警报才会包含在视图中。 | 
|资源类型 |选择一个或多个资源类型。 默认情况下，仅选择目标虚拟机的警报并将其列在此视图中。 仅在指定资源组后，才显示此列。 | 
|资源 |选择资源。 只有包含该资源（作为目标）的警报才会包含在视图中。 仅在指定资源类型后，才显示此列。 | 
|严重性 |选择警报严重性，或选择“所有”以包含所有严重性的警报。 | 
|监视条件 |选择一个监视条件可以筛选系统激发的警报；如果该条件不再有效，则可以筛选系统已解决的警报。 选择“所有”会包含所有条件的警报。 | 
|警报状态 |选择一种警报状态（“新”、“确认”、“已关闭”），或选择“所有”以包括所有状态的警报。 | 
|监视服务 |选择一个服务，或选择“所有”以包含所有服务。 此功能仅支持来自 VM 见解的警报。| 
|时间范围| 只有在选定时间范围内触发的警报才会包含在该视图中。 支持的值为过去 1 小时、过去 24 小时、过去 7 天和过去 30 天。 | 

选择一个警报时会显示“警报详细信息”页，其中提供了该警报的详细信息，并可在其中更改警报的状态。 要详细了解如何管理警报，请参阅[使用 Azure Monitor 创建、查看和管理警报](../../azure-monitor/platform/alerts-metric.md)。  

>[!NOTE]
>目前，暂不支持根据运行状况条件创建新警报，或通过门户在 Azure Monitor 中修改现有运行状况警报规则。  
>

![所选警报的警报详细信息窗格](./media/vminsights-health/alert-details-pane-01.png)

还可以更改一个或多个警报的警报状态，方法是选择这些警报，然后在“所有警报”页的左上角选择“更改状态”。 在“更改警报状态”窗格中选择一种状态，在“注释”字段中添加更改操作的说明，然后单击“确定”提交更改。 在验证信息和应用更改期间，可在菜单中的“通知”下面跟踪操作进度。  

### <a name="configure-alerts"></a>配置警报
某些警报管理任务不能从 Azure 门户管理和执行使用，以使[Azure Monitor REST API](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/components)。 具体而言：

- 启用或禁用运行状况条件的警报 
- 设置运行状况条件警报通知 

每个示例中使用的方法使用[ARMClient](https://github.com/projectkudu/armclient)在 Windows 计算机上。 如果您不熟悉此方法，请参阅[使用 ARMClient](../platform/rest-api-walkthrough.md#use-armclient)。  

#### <a name="enable-or-disable-alert-rule"></a>启用或禁用警报规则

若要启用或禁用特定的运行状况条件，运行状况条件属性时的警报*alertGeneration*需要使用的值来修改**禁用**或**已启用**. 若要识别*monitorId*的特定运行状况条件，下面的示例将说明如何查询条件值**LogicalDisk\Avg Disk 秒 Per Transfer**。

1. 在终端窗口中，键入 **armclient.exe login**。 这样做会提示你登录到 Azure。

2. 键入以下命令检索所有活动的特定虚拟机上的运行状况条件并确定的值*monitorId*属性。 

    ```
    armclient GET "subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/monitors?api-version=2018-08-31-preview”
    ```

    下面的示例显示了该命令的输出。 记下的值*MonitorId*。 此值是必需的下一步需要指定运行状况条件的 ID，并修改其属性，以创建警报。

    ```
    "id": "/subscriptions/a7f23fdb-e626-4f95-89aa-3a360a90861e/resourcegroups/Lab/providers/Microsoft.Compute/virtualMachines/SVR01/providers/Microsoft.WorkloadMonitor/monitors/ComponentTypeId='LogicalDisk',MonitorId='Microsoft_LogicalDisk_AvgDiskSecPerRead'",
      "name": "ComponentTypeId='LogicalDisk',MonitorId='Microsoft_LogicalDisk_AvgDiskSecPerRead'",
      "type": "Microsoft.WorkloadMonitor/virtualMachines/monitors"
    },
    {
      "properties": {
        "description": "Monitor the performance counter LogicalDisk\\Avg Disk Sec Per Transfer",
        "monitorId": "Microsoft_LogicalDisk_AvgDiskSecPerTransfer",
        "monitorName": "Microsoft.LogicalDisk.AvgDiskSecPerTransfer",
        "monitorDisplayName": "Average Logical Disk Seconds Per Transfer",
        "parentMonitorName": null,
        "parentMonitorDisplayName": null,
        "monitorType": "Unit",
        "monitorCategory": "PerformanceHealth",
        "componentTypeId": "LogicalDisk",
        "componentTypeName": "LogicalDisk",
        "componentTypeDisplayName": "Logical Disk",
        "monitorState": "Enabled",
        "criteria": [
          {
            "healthState": "Warning",
            "comparisonOperator": "GreaterThan",
            "threshold": 0.1
          }
        ],
        "alertGeneration": "Enabled",
        "frequency": 1,
        "lookbackDuration": 17,
        "documentationURL": "https://aka.ms/Ahcs1r",
        "configurable": true,
        "signalType": "Metrics",
        "signalName": "VMHealth_Avg. Logical Disk sec/Transfer"
      },
      "etag": null,
    ```

3. 键入以下命令来修改*alertGeneration*属性。

    ```
    armclient patch subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/monitors/Microsoft_LogicalDisk_AvgDiskSecPerTransfer?api-version=2018-08-31-preview "{'properties':{'alertGeneration':'Disabled'}}"
    ```   

4. 键入在步骤 2 中用于验证属性的值设置为 GET 命令**禁用**。  

#### <a name="associate-action-group-with-health-criteria"></a>将操作组与运行状况条件相关联

Azure Vm 运行状况监视器支持短信和电子邮件通知时，将生成警报时运行状况条件将变为不正常。 若要配置通知，需要记下配置为发送短信或电子邮件通知的操作组的名称。 

>[!NOTE]
>此操作需要对受监视的每个 VM 执行你想要接收的通知。

1. 在终端窗口中，键入 **armclient.exe login**。 这样做会提示你登录到 Azure。

2. 键入以下命令以将操作组关联的警报规则。
 
    ```
    $payload = "{'properties':{'ActionGroupResourceIds':['/subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/microsoft.insights/actionGroups/actiongroupName']}}" 
    armclient PUT https://management.azure.com/subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/notificationSettings/default?api-version=2018-08-31-preview $payload
    ```

3. 若要验证的属性值**actionGroupResourceIds**已成功更新，请键入以下命令。

    ```
    armclient GET "subscriptions/subscriptionName/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/notificationSettings?api-version=2018-08-31-preview"
    ```

    输出应如下所示：
    
    ```
    {
      "value": [
        {
          "properties": {
            "actionGroupResourceIds": [
              "/subscriptions/a7f23fdb-e626-4f95-89aa-3a360a90861e/resourceGroups/Lab/providers/microsoft.insights/actionGroups/Lab-IT%20Ops%20Notify"
            ]
          },
          "etag": null,
          "id": "/subscriptions/a7f23fdb-e626-4f95-89aa-3a360a90861e/resourcegroups/Lab/providers/Microsoft.Compute/virtualMachines/SVR01/providers/Microsoft.WorkloadMonitor/notificationSettings/default",
          "name": "notificationSettings/default",
          "type": "Microsoft.WorkloadMonitor/virtualMachines/notificationSettings"
        }
      ],
      "nextLink": null
    }
    ```

## <a name="next-steps"></a>后续步骤

若要查明 VM 性能的瓶颈和整体利用率，请参阅[查看 Azure VM 性能](vminsights-performance.md)；若要查看已发现的应用程序依赖项，请参阅[查看用于 VM 的 Azure Monitor 映射](vminsights-maps.md)。 
