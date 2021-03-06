---
title: 有关如何结合使用 Azure ExpressRoute 与 Azure Site Recovery 来实现灾难恢复和迁移 | Microsoft Docs
description: 介绍如何结合使用 Azure ExpressRoute 与 Azure Site Recovery 服务来实现灾难恢复和迁移。
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/18/2019
ms.author: mayg
ms.openlocfilehash: bf4cce8a224db81b8db7fae6a69b8b578bb3d47a
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60772293"
---
# <a name="azure-expressroute-with-azure-site-recovery"></a>将 Azure ExpressRoute 与 Azure Site Recovery 结合使用

使用 Microsoft Azure ExpressRoute 可通过连接服务提供商所提供的专用连接，将本地网络扩展到 Microsoft 云。 使用 ExpressRoute 可与 Microsoft Azure、Office 365 和 Dynamics 365 等 Microsoft 云服务建立连接。

本文介绍如何结合使用 Azure ExpressRoute 与 Azure Site Recovery 来实现灾难恢复和迁移。

## <a name="expressroute-circuits"></a>ExpressRoute 线路

ExpressRoute 线路表示通过连接提供商在本地基础结构与 Microsoft 云服务之间建立的逻辑连接。 可以订购多条 ExpressRoute 线路。 每条线路可以位于相同或不同的区域，且可以通过不同的连接提供程序连接到本地。 从[此处](../expressroute/expressroute-circuit-peerings.md)详细了解 ExpressRoute 线路。

## <a name="expressroute-routing-domains"></a>ExpressRoute 路由域

一条 ExpressRoute 线路有多个关联的路由域：
-   [Azure 专用对等互连](../expressroute/expressroute-circuit-peerings.md#privatepeering)：可以通过专用对等互连域来连接虚拟网络内部署的 Azure 计算服务（即虚拟机 (IaaS) 和云服务 (PaaS)）。 专用对等互连域被视为进入 Microsoft Azure 的核心网络的受信任扩展。
-   [Azure 公共对等互连](../expressroute/expressroute-circuit-peerings.md#publicpeering)：Azure 存储、SQL 数据库和网站等服务是使用公共 IP 地址提供的。 可以通过公共对等路由域私下连接到公共 IP 地址（包括云服务的 VIP）上托管的服务。 对于新创建的服务，不应再使用公共对等互连；对于 Azure PaaS 服务，应改用 Microsoft 对等互连。
-   [Microsoft 对等互连](../expressroute/expressroute-circuit-peerings.md#microsoftpeering)：与 Microsoft 联机服务（Office 365、Dynamics 365 和 Azure PaaS 服务）的连接通过 Microsoft 对等互连建立。 在连接到 Azure PaaS 服务时，建议使用 Microsoft 对等互连作为路由域。

从[此处](../expressroute/expressroute-circuit-peerings.md#peeringcompare)详细了解并比较 ExpressRoute 路由域。

## <a name="on-premises-to-azure-replication-with-expressroute"></a>使用 ExpressRoute 进行本地到 Azure 的复制

Azure Site Recovery 支持从本地 [Hyper-V 虚拟机](hyper-v-azure-architecture.md)、[VMware 虚拟机](vmware-azure-architecture.md)和[物理服务器](physical-azure-architecture.md)向 Azure 进行灾难恢复和迁移。 对于所有本地到 Azure 的方案，复制数据都发送到 Azure 存储帐户并存储在其中。 在复制期间，无需支付任何虚拟机费用。 故障转移到 Azure 时，Site Recovery 会自动创建 Azure IaaS 虚拟机。

Site Recovery 通过公共终结点将数据复制到 Azure 存储帐户。 若要将 ExpressRoute 用于 Site Recovery 复制，可以利用[公共对等互连](../expressroute/expressroute-circuit-peerings.md#publicpeering)（新的创建已弃用） 或[Microsoft 对等互连](../expressroute/expressroute-circuit-peerings.md#microsoftpeering)。 在复制时，建议使用 Microsoft 对等互连作为路由域。 此外，复制时还应确保满足[网络要求](vmware-azure-configuration-server-requirements.md#network-requirements)。 在将虚拟机或服务器故障转移到 Azure 虚拟网络后，可以使用[专用对等互连](../expressroute/expressroute-circuit-peerings.md#privatepeering)访问它们。 私有对等互连不支持复制。

在本地使用代理服务器并且想要对复制流量使用 ExpressRoute，你需要配置服务器和进程服务器上配置代理跳过列表。 请遵循以下步骤进行配置：

- 下载 PsExec 工具从[此处](https://aka.ms/PsExec)以访问系统用户上下文。
- 在系统用户上下文中打开 Internet Explorer，通过运行以下命令行 psexec-s-i"%programfiles%\Internet Explorer\iexplore.exe"
- 在 IE 中添加代理设置
- 在忽略列表中，添加 Azure 存储 URL *。 blob.core.windows.net

这将确保仅复制流量流经 ExpressRoute，通信可以通过代理转而。

下图描绘了这种组合方案：![使用 ExpressRoute 进行本地到 Azure 的复制](./media/concepts-expressroute-with-site-recovery/site-recovery-with-expressroute.png)

## <a name="azure-to-azure-replication-with-expressroute"></a>使用 ExpressRoute 进行 Azure 到 Azure 的复制

Azure Site Recovery 支持对 [Azure 虚拟机](azure-to-azure-architecture.md)进行灾难恢复。 根据 Azure 虚拟机是否使用 [Azure 托管磁盘](../virtual-machines/windows/managed-disks-overview.md)，复制数据将被发送到目标 Azure 区域中的 Azure 存储帐户或副本托管磁盘。 虽然复制终结点是公共终结点，但在默认情况下，无论源虚拟网络位于哪个 Azure 区域，Azure VM 复制的复制流量都不会遍历 Internet。 对于 0.0.0.0/0 地址前缀，可使用[自定义路由](../virtual-network/virtual-networks-udr-overview.md#custom-routes)覆盖 Azure 的默认系统路由，并将 VM 流量转移到本地网络虚拟设备 (NVA)，但不建议对 Site Recovery 复制使用此配置。 如果使用自定义路由，则应在虚拟网络中为“存储”[创建一个虚拟网络服务终结点](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage)，这样复制流量就不会离开 Azure 边界。

对于 Azure VM 灾难恢复，默认情况下，复制时不需要 ExpressRoute。 在将虚拟机故障转移到目标 Azure 区域后，可以使用[专用对等互连](../expressroute/expressroute-circuit-peerings.md#privatepeering)访问它们。

如果已经使用 ExpressRoute 从本地数据中心连接到源区域中的 Azure VM，则可以计划在故障转移目标区域重新建立 ExpressRoute 连接。 进行灾难恢复时，可以使用相同的 ExpressRoute 线路通过新的虚拟网络连接连接到目标区域，也可以使用不同的 ExpressRoute 线路和连接。 [此处](azure-vm-disaster-recovery-with-expressroute.md#fail-over-azure-vms-when-using-expressroute)介绍了各种可能的方案。

可以将 Azure 虚拟机复制到同一地理群集内的任何 Azure 区域，详细信息如[此处](../site-recovery/azure-to-azure-support-matrix.md#region-support)所述。 如果所选目标 Azure 区域与源区域不在同一地缘政治区域内，则可能需要启用 ExpressRoute Premium。 有关更多详细信息，请查看 [ExpressRoute 位置](../expressroute/expressroute-locations.md#azure-regions-to-expressroute-locations-within-a-geopolitical-region)和 [ExpressRoute 定价](https://azure.microsoft.com/pricing/details/expressroute/)。

## <a name="next-steps"></a>后续步骤
- 详细了解 [ExpressRoute 线路](../expressroute/expressroute-circuit-peerings.md)。
- 详细了解 [ExpressRoute 路由域](../expressroute/expressroute-circuit-peerings.md#peeringcompare)。
- 详细了解 [ExpressRoute 位置](../expressroute/expressroute-locations.md)。
- 了解有关灾难恢复的详细信息[使用 ExpressRoute 的 Azure 虚拟机](azure-vm-disaster-recovery-with-expressroute.md)。
