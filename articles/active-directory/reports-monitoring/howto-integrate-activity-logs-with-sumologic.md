---
title: Stream 到使用 Azure Monitor 的 SumoLogic 的 Azure Active Directory 日志 |Microsoft Docs
description: 了解如何使用 Azure Monitor 的 SumoLogic 与 Azure Active Directory 日志
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 2c3db9a8-50fa-475a-97d8-f31082af6593
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7ccf9cd550b3848be841f193f4ab8a6094cf671d
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60437843"
---
# <a name="integrate-azure-active-directory-logs-with-sumologic-using-azure-monitor"></a>使用 Azure Monitor 的 SumoLogic 与 Azure Active Directory 日志

本文介绍如何使用 Azure Monitor 将 Azure Active Directory (Azure AD) 日志与 SumoLogic 集成。 首先将日志路由到 Azure 事件中心，然后将事件中心与 SumoLogic 相集成。

## <a name="prerequisites"></a>必备组件

若要使用此功能，需满足以下条件：
* 包含 Azure AD 活动日志的 Azure 事件中心。 了解如何[将活动日志流式传输到事件中心](quickstart-azure-monitor-stream-logs-to-event-hub.md)。 
* 启用了 SumoLogic 单一登录的订阅。

## <a name="steps-to-integrate-azure-ad-logs-with-sumologic"></a>将 Azure AD 日志与 SumoLogic 相集成的步骤 

1. 首先，[将 Azure AD 日志流式传输到 Azure 事件中心](quickstart-azure-monitor-stream-logs-to-event-hub.md)。
2. 将 SumoLogic 实例配置为[收集 Azure Active Directory 的日志](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Collect_Logs_for_Azure_Active_Directory)。
3. [安装 Azure AD SumoLogic 应用](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards)以使用预配置的仪表板，这可对环境进行实时分析。

   ![仪表板](./media/howto-integrate-activity-logs-with-sumologic/overview-dashboard.png)

## <a name="next-steps"></a>后续步骤

* [解释 Azure Monitor 中的审核日志架构](reference-azure-monitor-audit-log-schema.md)
* [解释 Azure Monitor 中的登录日志架构](reference-azure-monitor-sign-ins-log-schema.md)
* [常见问题解答和已知的问题](concept-activity-logs-azure-monitor.md#frequently-asked-questions)