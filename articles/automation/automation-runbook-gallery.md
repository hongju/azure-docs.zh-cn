---
title: Azure 自动化的 Runbook 和模块库
description: 可以安装并在 Azure 自动化环境中使用 Microsoft 和社区提供的 Runbook 与模块。  本文介绍如何访问这些资源，以及在库中补充 Runbook。
services: automation
ms.service: automation
ms.subservice: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/20/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 20aafc117ad8b6bd625894180fdfe79bd86192bd
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60737339"
---
# <a name="runbook-and-module-galleries-for-azure-automation"></a>Azure 自动化的 Runbook 和模块库

无需在 Azure 自动化中创建自己的 Runbook 和模块，即可访问 Microsoft 和社区构建的方案。

可以获取 PowerShell runbook 和[模块](#modules-in-powershell-gallery)从 PowerShell 库和[Python runbook](#python-runbooks)从脚本中心库。 此外可以通过共享开发时，方案将 runbook 添加到库，请参阅参与到社区

## <a name="runbooks-in-powershell-gallery"></a>PowerShell 库中的 Runbook

[PowerShell 库](https://www.powershellgallery.com/packages)提供来自 Microsoft 和社区，您可以导入 Azure 自动化的 runbook。 若要使用其中一个，从库中，下载 runbook，或从库中，或从 Azure 门户中的自动化帐户可以直接导入 runbook。

您可以只导入直接从 PowerShell 库使用 Azure 门户。 无法执行此函数使用 PowerShell。

> [!NOTE]
> 您应验证您从 PowerShell 库获取，并使用在安装和在生产环境中运行它们时要特别注意的任何 runbook 的内容。

### <a name="to-import-a-powershell-runbook-from-the-runbook-gallery-with-the-azure-portal"></a>从 Runbook 库使用 Azure 门户中导入 PowerShell runbook

1. 在 Azure 门户中，打开自动化帐户。
2. 在“流程自动化”下，单击“Runbook 库”
3. 选择**源：PowerShell 库**。
4. 找到所需的库项，选择它以查看其详细信息。 在左侧可以输入发布者和类型的其他搜索参数。

   ![浏览库](media/automation-runbook-gallery/browse-gallery.png)

5. 单击“查看源项目”以在 [TechNet 脚本中心](https://gallery.technet.microsoft.com/)查看该项。
6. 要导入项，请单击它以查看其详细信息，并单击“导入”按钮。

   ![“导入”按钮](media/automation-runbook-gallery/gallery-item-detail.png)

7. 可选择更改 Runbook 的名称，并单击“确定”导入该 Runbook。
8. Runbook 将出现在自动化帐户的“Runbook”选项卡中。

### <a name="adding-a-powershell-runbook-to-the-gallery"></a>将 PowerShell runbook 添加到库

Microsoft 建议你将 runbook 添加到您认为会对其他客户有用的 PowerShell 库。 PowerShell 库接受 PowerShell 模块和 PowerShell 脚本。 可以添加的 runbook[将数据上载到 PowerShell 库](/powershell/gallery/how-to/publishing-packages/publishing-a-package)。

> [!NOTE]
> PowerShell 库中不支持图形 runbook。

## <a name="modules-in-powershell-gallery"></a>PowerShell 库中的模块

PowerShell 模块包含可以在 Runbook 中使用的 cmdlet，并可以在 [PowerShell 库](https://www.powershellgallery.com)中找到可在 Azure 自动化中安装的现有模块。 可以从 Azure 门户启动此库，并将这些模块直接安装到 Azure 自动化中。 也可以下载并手动安装这些模块。  

### <a name="to-import-a-module-from-the-automation-module-gallery-with-the-azure-portal"></a>使用 Azure 门户从自动化模块库导入模块

1. 在 Azure 门户中，打开自动化帐户。
2. 在“共享资源”中选择“模块”，打开模块列表。
3. 请单击页面顶部的“浏览全部”。

   ![模块库](media/automation-runbook-gallery/modules-blade.png)

4. 在“浏览库”页，可以按以下字段进行搜索：

   * 模块名称
   * 标记
   * 作者
   * Cmdlet/DSC 资源名称

5. 找到感兴趣的模块并选择它以查看其详细信息。  

   深入了解特定模块时，可以查看更多信息。 此信息包括返回 PowerShell 库的链接、任何所需的依赖项以及该模块包含的所有 cmdlet 或 DSC 资源。

   ![PowerShell 模块详细信息](media/automation-runbook-gallery/gallery-item-details-blade.png)

6. 要直接将模块安装到 Azure 自动化中，请单击“导入”按钮。
7. 在“导入”页中单击“导入”按钮时，会看到将要导入的模块名称。 如果安装了所有依赖项，“确定”按钮将处于活动状态。 如果缺少依赖项，则需要在导入此模块前导入这些依赖项。
8. 在“导入”页上，单击“确定”导入模块。 Azure 自动化将模块导入帐户时，它提取有关该模块和 cmdlet 的元数据。 此操作可能需要几分钟才能完成，因为需要提取每个活动。
9. 将收到正在部署该模块的初始通知；完成此过程后，还会收到另一通知。
10. 导入模块后，可以看到可用的活动。 可以在 Runbook 和所需状态配置中使用其资源。

> [!NOTE]
> 仅支持 PowerShell 核心的模块将在 Azure 自动化中不受支持，无法在 Azure 门户中导入，或直接从 PowerShell 库部署。

## <a name="python-runbooks"></a>Python Runbook

可在[脚本中心库](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&f%5B0%5D.Value=WindowsAzure&f%5B1%5D.Type=ProgrammingLanguage&f%5B1%5D.Value=Python&f%5B1%5D.Text=Python&sortBy=Date&username=)中找到 Python Runbook。 你可以通过单击参与到脚本中心库的 Python runbook**上载发布内容**。 执行此操作时，请确保在上传内容时添加标签“Python”。

> [!NOTE]
> 若要将内容上载到[脚本中心](https://gallery.technet.microsoft.com/scriptcenter)100 个点的最小值是必需的。 

## <a name="requesting-a-runbook-or-module"></a>请求 Runbook 或模块

可以将请求发送到[用户之声](https://feedback.azure.com/forums/246290-azure-automation/)。  如果需要 Runbook 编写帮助，或对 PowerShell 存有疑问，请将问题发布到我们的[论坛](https://social.msdn.microsoft.com/Forums/windowsazure/home?forum=azureautomation&filter=alltypes&sort=lastpostdesc)。

## <a name="next-steps"></a>后续步骤

* 若要开始使用 Runbook，请参阅[在 Azure 自动化中管理 Runbook](manage-runbooks.md)
* 若要了解 Runbook 的 PowerShell 和 PowerShell 工作流之间的差异，请参阅[了解 PowerShell 工作流](automation-powershell-workflow.md)
