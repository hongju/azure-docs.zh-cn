---
title: 为 Azure Red Hat OpenShift 的 azure Active Directory 集成 |Microsoft Docs
description: 了解如何创建用于测试 Microsoft Azure Red Hat OpenShift 群集上的应用程序的 Azure AD 安全组和用户。
author: tylermsft
ms.author: twhitney
ms.service: openshift
manager: jeconnoc
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/13/2019
ms.openlocfilehash: f6b87748c33c1afd047ae25dfb7df3670a73e7c8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779681"
---
# <a name="azure-active-directory-integration-for-azure-red-hat-openshift"></a>为 Azure Red Hat OpenShift 的 azure Active Directory 集成

如果你尚未创建 Azure Active Directory (Azure AD) 租户，请按照中的说明操作[创建 Azure AD 租户的 Azure Red Hat OpenShift](howto-create-tenant.md)按照说明继续操作之前。

Microsoft Azure Red Hat OpenShift 需要有权代表你的群集执行任务。 如果你的组织还没有 Azure AD 用户、 Azure AD 安全组或 Azure AD 应用注册，以用作服务主体，请按照这些说明来创建它们。

## <a name="create-a-new-azure-active-directory-user"></a>创建新的 Azure Active Directory 用户

在[Azure 门户](https://portal.azure.com)，确保在你的用户名称将显示你的租户门户的权限：

![右上角中列出的与租户的门户屏幕截图](./media/howto-create-tenant/tenant-callout.png)如果显示了错误的租户，请单击右上方，您的用户名称，然后单击**切换目录**，然后选择从正确的租户**所有目录**列表。

创建一个新的 Azure Active Directory 全局管理员用户登录到 Azure Red Hat OpenShift 群集。

1. 转到[用户-所有用户](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers)边栏选项卡。
2. 单击 **+ 新用户**以打开**用户**窗格。
3. 输入**名称**为此用户。
4. 创建**用户名**根据你创建的租户的名称与`.onmicrosoft.com`附加在后面。 例如，`yourUserName@yourTenantName.onmicrosoft.com`。 记下此用户名。 你将需要它来登录到你的群集。
5. 单击**目录角色**以打开目录角色窗格中，选择**全局管理员**，然后单击**确定**窗格的底部。
6. 在中**用户**窗格中，单击**显示密码**并记录临时密码。 在第一次登录后，将提示您将其重置。
7. 在窗格的底部，单击**创建**以创建用户。

## <a name="create-an-azure-ad-security-group"></a>创建 Azure AD 安全组

若要授予群集管理员访问权限中的 Azure AD 安全组的成员身份会同步到 OpenShift 组"osa-客户的管理员"。 如果未指定，将不授予任何群集管理员访问权限。

1. 打开[Azure Active Directory 组](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups)边栏选项卡。
2. 单击 **+ 新建组**
3. 提供组名称和描述。
4. 设置**组类型**到**安全**。
5. 设置**成员身份类型**到**已分配**。

    将添加到此安全组在前面步骤中创建的 Azure AD 用户。

6. 单击**成员**以打开**选择成员**窗格。
7. 在成员列表中，选择上面创建的 Azure AD 用户。
8. 在门户的底部，单击**选择**，然后**创建**创建安全组。

    请记下的组 ID 值

9. 创建组后，您将看到它在所有组的列表。 单击新组。
10. 在显示页面上，复制**对象 ID**。 我们将把此值作为`GROUPID`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

## <a name="create-an-azure-ad-app-registration"></a>创建 Azure AD 应用注册

可以自动创建的 Azure Active Directory (Azure AD) 应用注册客户端通过省略创建群集的一部分`--aad-client-app-id`标记，用于`az openshift create`命令。 本教程演示如何创建 Azure AD 应用注册出于完整性的考虑。

如果你的组织还没有 Azure Active Directory (Azure AD) 应用注册要用作服务主体，请按照以下说明创建一个。

1. 打开[应用注册边栏选项卡](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview)然后单击 **+ 新注册**。
2. 在中**注册应用程序**窗格中，输入应用程序注册的名称。
3. 确保下**支持的帐户类型**的**此组织目录中的帐户**处于选中状态。 这是最安全的选择。
4. 一旦我们知道群集的 URI，我们将添加的重定向 URI 更高版本。 单击**注册**按钮以创建 Azure AD 应用程序注册。
5. 在显示页面上，复制**应用程序 （客户端） ID**。 我们将把此值作为`APPID`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

![应用程序对象页面的屏幕截图](./media/howto-create-tenant/get-app-id.png)

### <a name="create-a-client-secret"></a>创建客户端机密

生成您的应用程序到 Azure Active Directory 进行身份验证的客户端机密。

1. 在中**管理**部分中的应用程序注册页上，单击**证书和机密**。
2. 上**证书和机密**窗格中，单击 **+ 新客户端机密**。  **将客户端机密添加**窗格会显示。
3. 提供**说明**。
4. 设置**Expires**到您喜欢的持续时间**2 年内**。
5. 单击**外**密钥值将显示在**客户端机密**页部分。
6. 复制密钥值。 我们将把此值作为`SECRET`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。
 
![证书和机密窗格的屏幕截图](./media/howto-create-tenant/create-key.png)
 
Azure 应用程序对象的详细信息，请参阅[应用程序和 Azure Active Directory 中的服务主体对象](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)。

详细了解如何创建一个新的 Azure AD 应用程序，请参阅[使用 Azure Active Directory v1.0 终结点注册应用](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v1-add-azure-ad-app)。

## <a name="resources"></a>资源

* [应用程序和 Azure Active Directory 中的服务主体对象](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)  
* [快速入门：使用 Azure Active Directory v1.0 终结点注册应用](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v1-add-azure-ad-app)  

## <a name="next-steps"></a>后续步骤

如果你已满足所有[Azure Red Hat OpenShift 先决条件](howto-setup-environment.md)，已准备好创建第一个群集 ！

请尝试教程：
> [!div class="nextstepaction"]
> [创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)