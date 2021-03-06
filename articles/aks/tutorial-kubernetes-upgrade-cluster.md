---
title: "Azure 上的 Kubernertes 教學課程 - 更新叢集 | Microsoft Docs"
description: "Azure 上的 Kubernertes 教學課程 - 更新叢集"
services: container-service
documentationcenter: 
author: neilpeterson
manager: timlt
editor: 
tags: aks, azure-container-service
keywords: "Docker, 容器, 微服務, Kubernetes, DC/OS, Azure"
ms.assetid: 
ms.service: container-service
ms.devlang: aurecli
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/24/2017
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: f203e01e6aaecf04944ee830df4f6adeb1c74d2f
ms.sourcegitcommit: c5eeb0c950a0ba35d0b0953f5d88d3be57960180
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2017
---
# <a name="upgrade-kubernetes-in-azure-container-service-aks"></a>在 Azure Container Service (AKS) 中升級 Kubernetes

使用 Azure CLI 可以升級 Azure Container Service (AKS) 叢集。 在升級過程中，會仔細地[隔離並清空](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) \(英文\) Kubernetes 節點，將執行應用程式的中斷情況降到最低。

在本教學課程 (八個章節的第八部分) 中升級了 Kubernetes 叢集。 您完成的工作包括：

> [!div class="checklist"]
> * 識別目前和可用的 Kubernetes 版本
> * 升級 Kubernetes 節點
> * 驗證升級成功

## <a name="before-you-begin"></a>開始之前

在先前的教學課程中，已將應用程式封裝成容器映像、將此映像上傳至 Azure Container Registry，並已建立 Kubernetes 叢集。 該應用程式接著便在 Kubernetes 叢集上執行。 

如果您尚未完成這些步驟，而想要跟著做，請回到[教學課程 1 – 建立容器映像](./tutorial-kubernetes-prepare-app.md)。


## <a name="get-cluster-versions"></a>取得叢集版本

升級叢集之前，請使用 `az aks get-versions` 命令檢查哪些 Kubernetes 版本可進行升級。

```azurecli-interactive
az aks get-versions --name myK8sCluster --resource-group myResourceGroup --output table
```

您可以看到目前的節點版本是 `1.7.7` ，有版本 `1.8.1` 可用。

```
Name     ResourceGroup    MasterVersion    MasterUpgrades    AgentPoolVersion    AgentPoolUpgrades
-------  ---------------  ---------------  ----------------  ------------------  -------------------
default  myResourceGroup  1.7.7            1.8.1             1.7.7               1.8.1
```

## <a name="upgrade-cluster"></a>升級叢集

使用 `az aks upgrade` 命令升級叢集節點。 下列範例將叢集更新為版本 `1.8.1`。

```azurecli-interactive
az aks upgrade --name myK8sCluster --resource-group myResourceGroup --kubernetes-version 1.8.1
```

輸出：

```json
{
  "id": "/subscriptions/4f48eeae-9347-40c5-897b-46af1b8811ec/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myK8sCluster",
  "location": "westus2",
  "name": "myK8sCluster",
  "properties": {
    "accessProfiles": {
      "clusterAdmin": {
        "kubeConfig": "..."
      },
      "clusterUser": {
        "kubeConfig": "..."
      }
    },
    "agentPoolProfiles": [
      {
        "count": 1,
        "dnsPrefix": null,
        "fqdn": null,
        "name": "myK8sCluster",
        "osDiskSizeGb": null,
        "osType": "Linux",
        "ports": null,
        "storageProfile": "ManagedDisks",
        "vmSize": "Standard_D2_v2",
        "vnetSubnetId": null
      }
    ],
    "dnsPrefix": "myK8sClust-myResourceGroup-4f48ee",
    "fqdn": "myk8sclust-myresourcegroup-4f48ee-406cc140.hcp.westus2.azmk8s.io",
    "kubernetesVersion": "1.8.1",
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "..."
          }
        ]
      }
    },
    "provisioningState": "Succeeded",
    "servicePrincipalProfile": {
      "clientId": "e70c1c1c-0ca4-4e0a-be5e-aea5225af017",
      "keyVaultSecretRef": null,
      "secret": null
    }
  },
  "resourceGroup": "myResourceGroup",
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

## <a name="validate-upgrade"></a>驗證升級

您現在可以使用 `az aks show` 命令，確認升級是否成功。

```azurecli-interactive
az aks show --name myK8sCluster --resource-group myResourceGroup --output table
```

輸出：

```json
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------------
myK8sCluster  westus2     myResourceGroup  1.8.1                Succeeded            myk8sclust-myresourcegroup-3762d8-2f6ca801.hcp.westus2.azmk8s.io
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已在 AKS 叢集中升級 Kubernetes。 已完成下列工作：

> [!div class="checklist"]
> * 識別目前和可用的 Kubernetes 版本
> * 升級 Kubernetes 節點
> * 驗證升級成功

跟著此連結以深入了解 AKS。

> [!div class="nextstepaction"]
> [AKS 概觀](./intro-kubernetes.md)