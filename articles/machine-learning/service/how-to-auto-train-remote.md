---
title: 自动 ML 远程计算目标
titleSuffix: Azure Machine Learning service
description: 了解如何构建与 Azure 机器学习服务的 Azure 机器学习远程计算目标上使用自动的机器学习模型
services: machine-learning
author: nacharya1
ms.author: nilesha
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/04/2018
ms.custom: seodec18
ms.openlocfilehash: 6a18bdf3a2a1ccd60ff20d21ebd99f4f6e15e38f
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/13/2019
ms.locfileid: "65551339"
---
# <a name="train-models-with-automated-machine-learning-in-the-cloud"></a>在云中使用自动化机器学习对模型进行训练

在 Azure 机器学习中，我们在所管理的不同类型的计算资源上训练模型。 计算目标可以是本地计算机，也可以是云中的计算机。

可以轻松地纵向扩展或通过添加额外的计算目标，例如 Azure 机器学习计算 (AmlCompute) 横向扩展机器学习实验。 AmlCompute 是可以轻松地创建单个或多节点计算的管理计算基础结构。

在本文中，您将学习如何生成使用 AmlCompute 自动化机器学习模型。

## <a name="how-does-remote-differ-from-local"></a>远程与本地有何区别？

教程“[使用自动化机器学习训练分类模型](tutorial-auto-train-models.md)”讲授了如何使用本地计算机通过自动化机器学习来训练模型。  本地培训的工作流同样适用于远程目标。 但是，使用远程计算，能够以异步方式执行自动化机器学习试验迭代。 此功能允许你取消特定迭代，观察执行状态，或继续在 Jupyter 笔记本的其他单元格上处理。 若要远程训练，首先创建一个远程计算目标，例如 AmlCompute。 然后，配置远程资源，并在那里提交代码。

本文介绍远程 AmlCompute 目标上运行自动化的机器学习试验所需的额外步骤。 本教程中的工作区对象 `ws` 将会在此处的整个代码中使用。

```python
ws = Workspace.from_config()
```

## <a name="create-resource"></a>创建资源

在你的工作区中创建 AmlCompute 目标 (`ws`) 如果它尚不存在。  

**时间估计**：AmlCompute 目标的创建需要大约 5 分钟。

```python
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget

amlcompute_cluster_name = "automlcl" #Name your cluster
provisioning_config = AmlCompute.provisioning_configuration(vm_size = "STANDARD_D2_V2", 
                                                            # for GPU, use "STANDARD_NC6"
                                                            #vm_priority = 'lowpriority', # optional
                                                            max_nodes = 6)

compute_target = ComputeTarget.create(ws, amlcompute_cluster_name, provisioning_config)
    
# Can poll for a minimum number of nodes and for a specific timeout.
# If no min_node_count is provided, it will use the scale settings for the cluster.
compute_target.wait_for_completion(show_output = True, min_node_count = None, timeout_in_minutes = 20)
```

现在，可以使用 `compute_target` 对象作为远程计算目标。

群集名称限制包括：
+ 必须小于 64 个字符。  
+ 不得包含以下任何字符：`\` ~ ! @ # $ % ^ & * ( ) = + _ [ ] { } \\\\ | ; : \' \\" , < > / ?.`

## <a name="access-data-using-getdata-file"></a>使用 get_data 文件访问数据

提供对定型数据的远程资源访问权限。 对于在远程计算上运行的自动化机器学习实验，需要使用 `get_data()` 函数来提取数据。  

若要提供访问权限，必须：
+ 创建一个包含 `get_data()` 函数的 get_data.py 文件 
+ 将该文件置于可以作为绝对路径访问的目录中 

你可以封装代码，以从 blob 存储或 get_data.py 文件中的本地磁盘读取数据。 在下面的代码示例中，数据来自 sklearn 包。

>[!Warning]
>如果使用的是远程计算，则必须使用 `get_data()`，其中将执行数据转换。 如果需要为 get_data() 中的数据转换安装额外的库，则应当接着执行额外的步骤。 有关详细信息，请参阅 [auto-ml-dataprep sample notebook](https://aka.ms/aml-auto-ml-data-prep )。


```python
# Create a project_folder if it doesn't exist
if not os.path.exists(project_folder):
    os.makedirs(project_folder)

#Write the get_data file.
%%writefile $project_folder/get_data.py

from sklearn import datasets
from scipy import sparse
import numpy as np

def get_data():
    
    digits = datasets.load_digits()
    X_digits = digits.data[10:,:]
    y_digits = digits.target[10:]

    return { "X" : X_digits, "y" : y_digits }
```

## <a name="configure-experiment"></a>配置试验

为 `AutoMLConfig` 指定设置。  （请参阅[完整参数列表](how-to-configure-auto-train.md#configure-experiment)及其可能值。）

在设置中，`run_configuration` 设置为 `run_config` 对象，其中包含 DSVM 的设置和配置。  

```python
from azureml.train.automl import AutoMLConfig
import time
import logging

automl_settings = {
    "name": "AutoML_Demo_Experiment_{0}".format(time.time()),
    "iteration_timeout_minutes": 10,
    "iterations": 20,
    "n_cross_validations": 5,
    "primary_metric": 'AUC_weighted',
    "preprocess": False,
    "max_concurrent_iterations": 10,
    "verbosity": logging.INFO
}

automl_config = AutoMLConfig(task='classification',
                             debug_log='automl_errors.log',
                             path=project_folder,
                             compute_target = compute_target,
                             data_script=project_folder + "/get_data.py",
                             **automl_settings,
                            )
```

### <a name="enable-model-explanations"></a>启用模型说明

在 `AutoMLConfig` 构造函数中设置可选的 `model_explainability` 参数。 另外，若要使用模型说明功能，必须将验证 dataframe 对象作为参数 `X_valid` 进行传递。

```python
automl_config = AutoMLConfig(task='classification',
                             debug_log='automl_errors.log',
                             path=project_folder,
                             compute_target = compute_target,
                             data_script=project_folder + "/get_data.py",
                             **automl_settings,
                             model_explainability=True,
                             X_valid = X_test
                            )
```

## <a name="submit-training-experiment"></a>提交训练试验

现在，请提交配置，以自动选择算法、超参数并定型模型。

```python
from azureml.core.experiment import Experiment
experiment=Experiment(ws, 'automl_remote')
remote_run = experiment.submit(automl_config, show_output=True)
```

将会看到类似于以下示例的输出：

    Running on remote compute: mydsvmParent Run ID: AutoML_015ffe76-c331-406d-9bfd-0fd42d8ab7f6
    ***********************************************************************************************
    ITERATION: The iteration being evaluated.
    PIPELINE:  A summary description of the pipeline being evaluated.
    DURATION: Time taken for the current iteration.
    METRIC: The result of computing score on the fitted pipeline.
    BEST: The best observed score thus far.
    ***********************************************************************************************
    
     ITERATION     PIPELINE                               DURATION                METRIC      BEST
             2      Standardize SGD classifier            0:02:36                  0.954     0.954
             7      Normalizer DT                         0:02:22                  0.161     0.954
             0      Scale MaxAbs 1 extra trees            0:02:45                  0.936     0.954
             4      Robust Scaler SGD classifier          0:02:24                  0.867     0.954
             1      Normalizer kNN                        0:02:44                  0.984     0.984
             9      Normalizer extra trees                0:03:15                  0.834     0.984
             5      Robust Scaler DT                      0:02:18                  0.736     0.984
             8      Standardize kNN                       0:02:05                  0.981     0.984
             6      Standardize SVM                       0:02:18                  0.984     0.984
            10      Scale MaxAbs 1 DT                     0:02:18                  0.077     0.984
            11      Standardize SGD classifier            0:02:24                  0.863     0.984
             3      Standardize gradient boosting         0:03:03                  0.971     0.984
            12      Robust Scaler logistic regression     0:02:32                  0.955     0.984
            14      Scale MaxAbs 1 SVM                    0:02:15                  0.989     0.989
            13      Scale MaxAbs 1 gradient boosting      0:02:15                  0.971     0.989
            15      Robust Scaler kNN                     0:02:28                  0.904     0.989
            17      Standardize kNN                       0:02:22                  0.974     0.989
            16      Scale 0/1 gradient boosting           0:02:18                  0.968     0.989
            18      Scale 0/1 extra trees                 0:02:18                  0.828     0.989
            19      Robust Scaler kNN                     0:02:32                  0.983     0.989


## <a name="explore-results"></a>浏览结果

可以使用与[培训教程](tutorial-auto-train-models.md#explore-the-results)中相同的 Jupyter 小组件来查看图表和结果表格。

```python
from azureml.widgets import RunDetails
RunDetails(remote_run).show()
```
下面是小组件的静态图像。  在笔记本中，可以单击表格中的任意一行，查看运行属性和该运行的输出日志。   此外，还可以使用图表上方的下拉列表来查看每个迭代的每个可用指标的图表。

![小组件表](./media/how-to-auto-train-remote/table.png)
![小组件绘图](./media/how-to-auto-train-remote/plot.png)

小组件将显示可用于查看和浏览单个运行详细信息的 URL。
 
### <a name="view-logs"></a>查看日志

在 `/tmp/azureml_run/{iterationid}/azureml-logs` 下的 DSVM 上查找日志。

## <a name="best-model-explanation"></a>最佳模型说明

检索模型说明数据可以详细了解这些模型，更好地了解在后端运行的内容。 在此示例中，我们仅为最佳拟合模型运行模型说明。 如果为管道中的所有模型运行该说明，则会导致运行时间显著增加。 模型说明信息包括：

* shap_values：生成 shap lib 的说明信息。
* expected_values：适用于 X_train 数据集的模型的预期值。
* overall_summary：模型级别功能重要性值降序排序。
* overall_imp：功能名称如 overall_summary 中所示相同的顺序排序。
* per_class_summary：类级别功能重要性值，按降序排列。 仅适用于分类用例。
* per_class_imp：功能名称，排序方式与 per_class_summary 相同。 仅适用于分类用例。

使用以下代码，从迭代中选择最佳管道。 `get_output` 方法针对上次拟合调用返回最佳运行和拟合的模型。

```python
best_run, fitted_model = remote_run.get_output()
```

导入 `retrieve_model_explanation` 函数，在最佳模型上运行。

```python
from azureml.train.automl.automlexplainer import retrieve_model_explanation

shap_values, expected_values, overall_summary, overall_imp, per_class_summary, per_class_imp = \
    retrieve_model_explanation(best_run)
```

输出要查看的 `best_run` 说明变量的结果。

```python
print(overall_summary)
print(overall_imp)
print(per_class_summary)
print(per_class_imp)
```

在以下输出中输出 `best_run` 说明摘要变量结果。

![模型可说明性控制台输出](./media/how-to-auto-train-remote/expl-print.png)

也可通过工作区的 Azure 门户中的小组件 UI 和 Web UI 将功能重要性可视化。

![模型可说明性 UI](./media/how-to-auto-train-remote/model-exp.png)

## <a name="example"></a>示例

[How-to-use-azureml/automated-machine-learning/remote-amlcompute/auto-ml-remote-amlcompute.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/remote-amlcompute/auto-ml-remote-amlcompute.ipynb) notebook 演示了这篇文章中的概念。 

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>后续步骤

了解[如何为自动培训配置设置](how-to-configure-auto-train.md)。
