---
title: "變更 Azure Service Fabric 叢集設定 | Microsoft Docs"
description: "本文說明您可以自訂的網狀架構設定和網狀架構升級原則。"
services: service-fabric
documentationcenter: .net
author: chackdan
manager: timlt
editor: 
ms.assetid: 7ced36bf-bd3f-474f-a03a-6ebdbc9677e2
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/15/2017
ms.author: chackdan
ms.openlocfilehash: bfbfc5b46985d3413510ba69cbdfc688f46f0f1b
ms.sourcegitcommit: e5355615d11d69fc8d3101ca97067b3ebb3a45ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/31/2017
---
# <a name="customize-service-fabric-cluster-settings-and-fabric-upgrade-policy"></a>自訂 Service Fabric 叢集設定和網狀架構升級原則
本文件將告訴您如何為 Service Fabric 叢集自訂各種網狀架構設定和網狀架構升級原則。 您可以透過 [Azure 入口網站](https://portal.azure.com)或使用 Azure Resource Manager 範本來進行自訂。

> [!NOTE]
> 並非所有設定都可以在入口網站中使用。 若下列設定無法透過入口網站使用，請使用 Azure Resource Manager 範本自訂它。
> 

## <a name="customize-cluster-settings-using-resource-manager-templates"></a>使用 Resource Manager 範本自訂叢集設定
下列步驟說明如何將新設定 *MaxDiskQuotaInMB* 新增至 *Diagnostics* 區段。

1. 移至 https://resources.azure.com
2. 透過展開 [訂用帳戶] -> [資源群組] -> [Microsoft.ServiceFabric] -> \<您的叢集名稱>，來瀏覽至您的訂用帳戶
3. 選取右上角的 [讀取/寫入]。
4. 選取 [編輯]，並更新 `fabricSettings` JSON 元素，然後新增元素：

```
      {
        "name": "Diagnostics",
        "parameters": [
          {
            "name": "MaxDiskQuotaInMB",
            "value": "65536"
          }
        ]
      }
```

下列是您可自訂的網狀架構設定清單，並依區段分組。

### <a name="section-name-diagnostics"></a>區段名稱：Diagnostics
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ConsumerInstances |String | 動態 |DCA 取用者執行個體的清單。 |
| ProducerInstances |String | 動態 |DCA 產生者執行個體的清單。 |
| AppEtwTraceDeletionAgeInDays |整數，預設值為 3 | 動態 |天數，在此時間過後便會刪除含有應用程式 ETW 追蹤的舊有 ETL 檔案。 |
| AppDiagnosticStoreAccessRequiresImpersonation |布林值，預設值為 true | 動態 |在代表應用程式存取診斷存放區時是否需要模擬。 |
| MaxDiskQuotaInMB |整數，預設值為 65536 | 動態 |Windows Fabric 記錄檔的磁碟配額 (MB)。 |
| DiskFullSafetySpaceInMB |整數，預設值為 1024 | 動態 |要防止 DCA 使用的剩餘磁碟空間 (MB)。 |
| ApplicationLogsFormatVersion |整數，預設值為 0 | 動態 |應用程式記錄格式的版本。 支援的值為 0 和 1。 版本 1 所包含的 ETW 事件記錄欄位比版本 0 多。 |
| ClusterId |String | 動態 |叢集的唯一識別碼。 此參數會在建立叢集時產生。 |
| EnableTelemetry |布林值，預設值為 true | 動態 |這會用來啟用或停用遙測。 |
| EnableCircularTraceSession |布林值，預設值為 false | 靜態 |旗標會指出是否應使用循環追蹤工作階段。 |

### <a name="section-name-traceetw"></a>區段名稱︰Trace/Etw
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| Level |整數，預設值為 4 | 動態 |追蹤 etw 層級可以接受值 1、2、3、4。 您必須保持在追蹤層級 4，才可以支援 |

### <a name="section-name-performancecounterlocalstore"></a>區段名稱︰PerformanceCounterLocalStore
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| IsEnabled |布林值，預設值為 true | 動態 |旗標會指出是否啟用本機節點的效能計數器收集。 |
| SamplingIntervalInSeconds |整數，預設值為 60 | 動態 |所要收集之效能計數器的取樣間隔。 |
| Counters |String | 動態 |要收集之效能計數器的逗號分隔清單。 |
| MaxCounterBinaryFileSizeInMB |整數，預設值為 1 | 動態 |每個效能計數器之二進位檔案的大小上限 (MB)。 |
| NewCounterBinaryFileCreationIntervalInMinutes |整數，預設值為 10 | 動態 |間隔上限 (秒)，在此時間過後便會建立新的效能計數器二進位檔案。 |

### <a name="section-name-setup"></a>區段名稱︰Setup
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| FabricDataRoot |String | 不允許 |Service Fabric 資料的根目錄。 預設為 Azure d:\svcfab |
| FabricLogRoot |String | 不允許 |Service Fabric 記錄的根目錄。 這是放置 SF 記錄和追蹤的位置。 |
| ServiceRunAsAccountName |String | 不允許 |用來執行網狀架構主機服務的帳戶名稱。 |
| SkipFirewallConfiguration |布林值，預設值為 false | 不允許 |指定是否需要由系統設定防火牆設定。 這只有當您使用 Windows 防火牆時才適用。 如果您使用協力廠商防火牆，則您必須開啟要供系統和應用程式使用的連接埠 |
|NodesToBeRemoved|字串，預設值為 ""| 動態 |應在設定升級過程中移除的節點。 (僅適用於獨立部署)|
|ContainerNetworkSetup|布林值，預設值為 FALSE| 靜態 |是否要設定容器網路。|
|ContainerNetworkName|字串，預設值為 L""| 靜態 |要在設定容器網路時使用的網路名稱。|

### <a name="section-name-transactionalreplicator"></a>區段名稱︰TransactionalReplicator
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| MaxCopyQueueSize |單位，預設值為 16384 | 靜態 |這是用來定義保有複寫作業之佇列初始大小的最大值。 請注意，此值必須是 2 的乘冪。 如果在執行階段期間，佇列增加到此大小，則會將主要和次要複寫器之間的作業節流。 |
| BatchAcknowledgementInterval | 時間 (秒)，預設值為 0.015 | 靜態 | 以秒為單位指定時間範圍。 決定複寫器在收到作業後，要等待多久才回傳通知。 在此期間所收到的其他作業會透過同一則訊息回傳其通知，以降低網路流量，但可能也會降低複寫器的輸送量。 |
| MaxReplicationMessageSize |單位，預設值為 52428800 | 靜態 | 複寫作業的訊息大小上限。 預設值為 50 MB。 |
| ReplicatorAddress |字串，預設值為 "localhost:0" | 靜態 | -'IP:Port' 字串形式的端點，Windows Fabric 複寫器使用此端點來建立與其他複本的連線，以便傳送/接收作業。 |
| InitialPrimaryReplicationQueueSize |單位，預設值為 64 | 靜態 |此值會定義主要複寫器上保有複寫作業之佇列的初始大小。 請注意，此值必須是 2 的乘冪。|
| MaxPrimaryReplicationQueueSize |單位，預設值為 8192 | 靜態 |這是主要複寫佇列中可存在的作業數目上限。 請注意，此值必須是 2 的乘冪。 |
| MaxPrimaryReplicationQueueMemorySize |單位，預設值為 0 | 靜態 |這是主要複寫佇列的最大值 (位元組)。 |
| InitialSecondaryReplicationQueueSize |單位，預設值為 64 | 靜態 |此值會定義次要複寫器上保有複寫作業之佇列的初始大小。 請注意，此值必須是 2 的乘冪。 |
| MaxSecondaryReplicationQueueSize |單位，預設值為 16384 | 靜態 |這是次要複寫佇列中可存在的作業數目上限。 請注意，此值必須是 2 的乘冪。 |
| MaxSecondaryReplicationQueueMemorySize |單位，預設值為 0 | 靜態 |這是次要複寫佇列的最大值 (位元組)。 |
| SecondaryClearAcknowledgedOperations |布林值，預設值為 false | 靜態 |布林值，用來控制當次要複寫器上的作業已認可至主要複寫器 (已排清至磁碟) 時是否要予以清除。 將此參數設為 TRUE，會導致在容錯移轉之後快取複本時，在新的主要複寫器上產生額外的磁碟讀取。 |
| MaxMetadataSizeInKB |整數，預設值為 4 |不允許|記錄資料流中繼資料的大小上限。 |
| MaxRecordSizeInKB |單位，預設值為 1024 |不允許| 記錄資料流記錄的大小上限。 |
| CheckpointThresholdInMB |整數，預設值為 50 |靜態|記錄使用量超過此值時，便會起始檢查點。 |
| MaxAccumulatedBackupLogSizeInMB |整數，預設值為 800 |靜態|指定備份記錄鏈中備份記錄檔的最大累積大小 (MB)。 如果增量備份會產生導致累積備份記錄檔的備份記錄檔，增量備份要求將會失敗，因為相關完整備份會大於此大小。 在這個情況下，使用者需要進行完整備份。 |
| MaxWriteQueueDepthInKB |整數，預設值為 0 |不允許| 核心記錄器可以針對與此複本相關聯之記錄使用的寫入佇列深度上限，此值為整數值，並且會以 KB 為單位來指定。 此值是核心記錄器更新期間可處於待處理狀態的位元組數上限。 此值可為 0 (讓核心記錄器計算出適當值) 或 4 的倍數。 |
| SharedLogId |String |不允許|共用記錄識別碼。 此參數是 GUID，每個共用記錄的這個參數都應該是唯一的。 |
| SharedLogPath |String |不允許|共用記錄的路徑。 如果此值空白，則會使用預設共用記錄。 |
| SlowApiMonitoringDuration |時間 (秒)，預設值為 300 |靜態| 指定 API 的持續時間，在此時間過後便會引發警告健康狀態事件。|
| MinLogSizeInMB |整數，預設值為 0 |靜態|交易記錄檔大小下限。 系統將不會允許把記錄檔截斷為低於此設定的大小。 0 表示複寫器將會根據其他設定決定記錄大小下限。 提高這個值會提高執行部分複本和增量備份的可能性，因為這會降低將相關記錄檔記錄截斷的可能性。 |

### <a name="section-name-fabricclient"></a>區段名稱︰FabricClient
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| NodeAddresses |字串，預設值為 "" |靜態|不同節點上可用於與命名服務通訊的位址 (連接字串) 集合。 一開始用戶端會隨機選取其中一個位址來連接。 如果提供了多個連接字串，而且因為通訊或逾時錯誤而導致連接失敗，用戶端便會循序改用下一個位址。 如需重試語意的詳細資料，請參閱命名服務位址的重試區段。 |
| ConnectionInitializationTimeout |時間 (秒)，預設值為 2 |動態|以秒為單位指定時間範圍。 用戶端每次嘗試開啟閘道連線的連線逾時間隔。 |
| PartitionLocationCacheLimit |整數，預設值為 100000 |靜態|針對服務解析所快取的資料分割數目 (設為 0 表示沒有限制)。 |
| ServiceChangePollInterval |時間 (秒)，預設值為 120 |動態|以秒為單位指定時間範圍。 針對已登錄服務之變更通知回呼，輪詢從用戶端變為閘道之服務變更時，連續兩次輪詢之間所應有的間隔時間。 |
| KeepAliveIntervalInSeconds |整數，預設值為 20 |靜態|FabricClient 傳輸將 Keep-Alive 訊息傳送至閘道的間隔時間。 若為 0，keepAlive 會停用。 必須是正值。 |
| HealthOperationTimeout |時間 (秒)，預設值為 120 |動態|以秒為單位指定時間範圍。 傳送至健康狀態管理員之報告訊息的逾時值。 |
| HealthReportSendInterval |時間 (秒)，預設值為 30 |動態|以秒為單位指定時間範圍。 報告元件傳送累積的健康狀態報告至健康狀態管理員的間隔時間。 |
| HealthReportRetrySendInterval |時間 (秒)，預設值為 30 |動態|以秒為單位指定時間範圍。 報告元件重新傳送累積的健康狀態報告至健康狀態管理員的間隔時間。 |
| RetryBackoffInterval |時間 (秒)，預設值為 3 |動態|以秒為單位指定時間範圍。 輪詢間隔時間，在此時間過後便會重試作業。 |
| MaxFileSenderThreads |單位，預設值為 10 |靜態|以平行方式傳輸的檔案數上限。 |

### <a name="section-name-common"></a>區段名稱︰Common
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| PerfMonitorInterval |時間 (秒)，預設值為 1 |動態|以秒為單位指定時間範圍。 效能監視間隔。 設為 0 或負值會停用監視。 |

### <a name="section-name-healthmanager"></a>區段名稱︰HealthManager
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| EnableApplicationTypeHealthEvaluation |布林值，預設值為 false |靜態|叢集健康狀態評估原則︰啟用每一應用程式類型的健康狀態評估。 |

### <a name="section-name-nodedomainids"></a>區段名稱︰NodeDomainIds
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| UpgradeDomainId |字串，預設值為 "" |靜態|說明節點所屬的升級網域。 |
| PropertyGroup |NodeFaultDomainIdCollection |靜態|說明節點所屬的容錯網域。 容錯網域會透過可描述節點在資料中心內之位置的 URI 來定義。  容錯網域 URI 的格式為 fd:/fd/ 再後接 URI 路徑區段。|

### <a name="section-name-nodeproperties"></a>區段名稱︰NodeProperties
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| PropertyGroup |NodePropertyCollectionMap |靜態|節點屬性的字串索引鍵/值組集合。 |

### <a name="section-name-nodecapacities"></a>區段名稱︰NodeCapacities
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| PropertyGroup |NodeCapacityCollectionMap |靜態|不同計量的節點容量集合。 |

### <a name="section-name-fabricnode"></a>區段名稱︰FabricNode
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| StateTraceInterval |時間 (秒)，預設值為 300 |靜態|以秒為單位指定時間範圍。 在每個節點追蹤節點狀態以及在 FM/FMM 追蹤執行中節點的間隔。 |
| StartApplicationPortRange |整數，預設值為 0 |靜態|主控子系統所管理之應用程式連接埠的開頭。 若主控內的 EndpointFilteringEnabled 為 true，則需要此參數。 |
| EndApplicationPortRange |整數，預設值為 0 |靜態|主控子系統所管理之應用程式連接埠的結尾 (不含)。 若主控內的 EndpointFilteringEnabled 為 true，則需要此參數。 |
| ClusterX509StoreName |字串，預設值為 "My" |動態|包含用來保護叢集間通訊之叢集憑證的 X.509 憑證存放區名稱。 |
| ClusterX509FindType |字串，預設值為 "FindByThumbprint" |動態|指出如何在以下列 ClusterX509StoreName 支援值所指定的存放區中搜尋叢集憑證："FindByThumbprint"、"FindBySubjectName" 和 "FindBySubjectName"。當有多個相符項目時，會使用到期日最遠者。 |
| ClusterX509FindValue |字串，預設值為 "" |動態|搜尋用來找出叢集憑證的篩選值。 |
| ClusterX509FindValueSecondary |字串，預設值為 "" |動態|搜尋用來找出叢集憑證的篩選值。 |
| ServerAuthX509StoreName |字串，預設值為 "My" |動態|包含入場服務之伺服器憑證的 X.509 憑證存放區名稱。 |
| ServerAuthX509FindType |字串，預設值為 "FindByThumbprint" |動態|指出如何搜尋以下列 ServerAuthX509StoreName 支援值指定之存放區中的伺服器憑證︰FindByThumbprint、FindBySubjectName。 |
| ServerAuthX509FindValue |字串，預設值為 "" |動態|搜尋用來找出伺服器憑證的篩選值。 |
| ServerAuthX509FindValueSecondary |字串，預設值為 "" |動態|搜尋用來找出伺服器憑證的篩選值。 |
| ClientAuthX509StoreName |字串，預設值為 "My" |動態|包含預設管理員角色 FabricClient 之憑證的 X.509 憑證存放區名稱。 |
| ClientAuthX509FindType |字串，預設值為 "FindByThumbprint" |動態|指出如何搜尋以下列 ClientAuthX509StoreName 支援值指定之存放區中的憑證︰FindByThumbprint、FindBySubjectName。 |
| ClientAuthX509FindValue |字串，預設值為 "" | 動態|搜尋用來找出預設管理員角色 FabricClient 之憑證的篩選值。 |
| ClientAuthX509FindValueSecondary |字串，預設值為 "" |動態|搜尋用來找出預設管理員角色 FabricClient 之憑證的篩選值。 |
| UserRoleClientX509StoreName |字串，預設值為 "My" |動態|包含預設使用者角色 FabricClient 之憑證的 X.509 憑證存放區名稱。 |
| UserRoleClientX509FindType |字串，預設值為 "FindByThumbprint" |動態|指出如何搜尋以下列 UserRoleClientX509StoreName 支援值指定之存放區中的憑證︰FindByThumbprint、FindBySubjectName。 |
| UserRoleClientX509FindValue |字串，預設值為 "" |動態|搜尋用來找出預設使用者角色 FabricClient 之憑證的篩選值。 |
| UserRoleClientX509FindValueSecondary |字串，預設值為 "" |動態|搜尋用來找出預設使用者角色 FabricClient 之憑證的篩選值。 |

### <a name="section-name-paas"></a>區段名稱：Paas
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ClusterId |字串，預設值為 "" |不允許|網狀架構用來保護組態的 X509 憑證存放區。 |

### <a name="section-name-fabrichost"></a>區段名稱︰FabricHost
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| StopTimeout |時間 (秒)，預設值為 300 |動態|以秒為單位指定時間範圍。 託管服務的啟用、停用和升級逾時。 |
| StartTimeout |時間 (秒)，預設值為 300 |動態|以秒為單位指定時間範圍。 fabricactivationmanager 的啟動逾時。 |
| ActivationRetryBackoffInterval |時間 (秒)，預設值為 5 |動態|以秒為單位指定時間範圍。 每次啟用失敗的輪詢間隔；在每次連續啟用失敗之後，系統會重試啟用最多 MaxActivationFailureCount 次。 每次嘗試的重試間隔是連續啟用失敗與啟用輪詢間隔的乘積。 |
| ActivationMaxRetryInterval |時間 (秒)，預設值為 300 |動態|以秒為單位指定時間範圍。 啟用的重試間隔上限。 在每次連續失敗之後，重試間隔的計算方式為 Min( ActivationMaxRetryInterval; Continuous Failure Count * ActivationRetryBackoffInterval)。 |
| ActivationMaxFailureCount |整數，預設值為 10 |動態|這是系統在放棄之前會重試失敗之啟用的計數上限。 |
| EnableServiceFabricAutomaticUpdates |布林值，預設值為 false |動態|這是為了讓網狀架構能夠透過 Windows Update 自動更新。 |
| EnableServiceFabricBaseUpgrade |布林值，預設值為 false |動態|這是為了讓伺服器進行基底更新。 |
| EnableRestartManagement |布林值，預設值為 false |動態|這是為了讓伺服器重新啟動。 |


### <a name="section-name-failovermanager"></a>區段名稱︰FailoverManager
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| UserReplicaRestartWaitDuration |時間 (秒)，預設值為 60.0 * 30 |動態|以秒為單位指定時間範圍。 當保存的複本停止運作時，Windows Fabric 會先在這段持續時間內等候複本恢復運作，不行的話再建立新的取代複本 (這需要狀態複本)。 |
| QuorumLossWaitDuration |時間 (秒)，預設值為 MaxValue |動態|以秒為單位指定時間範圍。 這是資料分割可處於仲裁遺失狀態的持續時間上限。 如果此持續時間過後，資料分割仍處於仲裁遺失狀態，則會以將停止運作的複本視為遺失的方式，讓資料分割從仲裁遺失狀態復原。 請注意，這可能會造成資料遺失。 |
| UserStandByReplicaKeepDuration |時間 (秒)，預設值為 3600.0 * 24 * 7 |動態|以秒為單位指定時間範圍。 當保存的複本從停止運作狀態恢復過來，它可能已被取代。 此計時器會決定 FM 會將待命複本保留多久才予以捨棄。 |
| UserMaxStandByReplicaCount |整數，預設值為 1 |動態|系統針對使用者服務所保留之待命複本的預設數目上限。 |
| ExpectedClusterSize|整數，預設值為 1|動態|當叢集一開始啟動時，FM 會等候這諸多節點回報其已啟動後，再放置其他服務，包括命名之類的系統服務。  增加此值會讓叢集所需的啟動時間增加，但可避免較早啟動的節點負載過度，並避免因為有更多節點上線而需要進行其他動作。  一般來說，這個值應該設為初始叢集大小的一小部分。 |
|ClusterPauseThreshold|整數，預設值為 1|動態|如果系統中的節點數目低於此值，則進行放置、負載平衡並停止容錯移轉。 |
|TargetReplicaSetSize|整數，預設值為 7|不允許|這是 Windows Fabric 會維護的 FM 複本目標數目。  數字越高，FM 資料的可靠性越高，而交換條件則是損失少許效能。 |
|MinReplicaSetSize|整數，預設值為 3|不允許|這是 FM 的最小複本集大小。  如果作用中 FM 複本數目低於此值，FM 會拒絕對叢集所做的變更，直到複本數目至少恢復到最小值 |
|ReplicaRestartWaitDuration|時間範圍，預設值為 Common::TimeSpan::FromSeconds(60.0 * 30)|不允許|以秒為單位指定時間範圍。 這是 FMService 的 ReplicaRestartWaitDuration |
|StandByReplicaKeepDuration|時間範圍，預設值為 Common::TimeSpan::FromSeconds(3600.0 * 24 * 7)|不允許|以秒為單位指定時間範圍。 這是 FMService 的 StandByReplicaKeepDuration |
|PlacementConstraints|字串，預設值為 L""|不允許|容錯移轉管理員複本的任何放置條件約束 |
|ExpectedNodeFabricUpgradeDuration|時間範圍，預設值為 Common::TimeSpan::FromSeconds(60.0 * 30)|動態|以秒為單位指定時間範圍。 這是在 Windows Fabric 升級期間，用來升級節點的預期持續時間。 |
|ExpectedReplicaUpgradeDuration|時間範圍，預設值為 Common::TimeSpan::FromSeconds(60.0 * 30)|動態|以秒為單位指定時間範圍。 這是在應用程式升級期間，用來升級節點上所有複本的預期持續時間。 |
|ExpectedNodeDeactivationDuration|時間範圍，預設值為 Common::TimeSpan::FromSeconds(60.0 * 30)|動態|以秒為單位指定時間範圍。 這是節點用來完成停用的預期持續時間。 |
|IsSingletonReplicaMoveAllowedDuringUpgrade|布林值，預設值為 TRUE|動態|如果設為 true，則會允許目標複本集大小為 1 的複本在升級期間移動。 |
|ReconfigurationTimeLimit|時間範圍，預設值為 Common::TimeSpan::FromSeconds(300)|動態|以秒為單位指定時間範圍。 重新設定的時間限制，此時間過後，便會起始警告健康情況報告 |
|BuildReplicaTimeLimit|時間範圍，預設值為 Common::TimeSpan::FromSeconds(3600)|動態|以秒為單位指定時間範圍。 用來建置具狀態複本的時間限制，此時間過後，便會起始警告健康情況報告 |
|CreateInstanceTimeLimit|時間範圍，預設值為 Common::TimeSpan::FromSeconds(300)|動態|以秒為單位指定時間範圍。 用來建立無狀態執行個體的時間限制，此時間過後，便會起始警告健康情況報告 |
|PlacementTimeLimit|時間範圍，預設值為 Common::TimeSpan::FromSeconds(600)|動態|以秒為單位指定時間範圍。 用來觸達目標複本計數的時間限制，此時間過後，便會起始警告健康情況報告 |

### <a name="section-name-namingservice"></a>區段名稱︰NamingService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| TargetReplicaSetSize |整數，預設值為 7 |不允許|命名服務存放區之每個資料分割的複本集數目。 增加複本集數目可增加命名服務存放區資訊的可靠性層級，減少因為節點失敗所會遺失之資訊的變更，代價為 Windows Fabric 的負載以及對命名資料執行更新所需的時間量會增加。|
|MinReplicaSetSize | 整數，預設值為 3 |不允許| 要完成更新所需寫入到之命名服務複本的數目下限。 如果系統中的作用中複本少於此數目，可靠性系統便會拒絕更新命名服務存放區，直到複本還原為止。 此值不應超過 TargetReplicaSetSize。 |
|ReplicaRestartWaitDuration | 時間 (秒)，預設值為 (60.0 * 30)|不允許| 以秒為單位指定時間範圍。 當命名服務複本停止運作時，此計時器就會啟動。  當它到期時，FM 就會開始取代停止運作的複本 (還不會將它們視為遺失)。 |
|QuorumLossWaitDuration | 時間 (秒)，預設值為 MaxValue |不允許| 以秒為單位指定時間範圍。 當命名服務進入仲裁遺失狀態時，此計時器就會啟動。  當計時器到期時，FM 會將停止運作的複本視為遺失，並嘗試復原仲裁。 請注意，這可能會導致資料遺失。 |
|StandByReplicaKeepDuration | 時間 (秒)，預設值為 3600.0 * 2 |不允許| 以秒為單位指定時間範圍。 當命名服務複本從停止運作狀態恢復過來，它可能已被取代。  此計時器會決定 FM 會將待命複本保留多久才予以捨棄。 |
|PlacementConstraints | 字串，預設值為 "" |不允許| 命名服務的放置條件約束。 |
|ServiceDescriptionCacheLimit | 整數，預設值為 0 |靜態| 命名存放區服務之 LRU 服務說明快取中保有的項目數上限 (設為 0 表示沒有限制)。 |
|RepairInterval | 時間 (秒)，預設值為 5 |靜態| 以秒為單位指定時間範圍。 會開始修復授權單位擁有者和名稱擁有者間命名不一致問題的間隔時間。 |
|MaxNamingServiceHealthReports | 整數，預設值為 10 |動態|命名存放區服務會一次報告為狀況不良之緩慢作業的數目上限。 若為 0，則會傳送所有緩慢作業。 |
| MaxMessageSize |整數，預設值為 4\*1024\*1024 |靜態|用戶端節點通訊在使用命名時的訊息大小上限。 DOS 攻擊減輕，預設值為 4MB。 |
| MaxFileOperationTimeout |時間 (秒)，預設值為 30 |動態|以秒為單位指定時間範圍。 檔案存放區服務作業所允許的逾時值上限。 指定較大逾時值的要求會遭到拒絕。 |
| MaxOperationTimeout |時間 (秒)，預設值為 600 |動態|以秒為單位指定時間範圍。 用戶端作業所允許的逾時值上限。 指定較大逾時值的要求會遭到拒絕。 |
| MaxClientConnections |整數，預設值為 1000 |動態|每個閘道所允許的用戶端連線數目上限。 |
| ServiceNotificationTimeout |時間 (秒)，預設值為 30 |動態|以秒為單位指定時間範圍。 將服務通知傳遞給用戶端時所使用的逾時值。 |
| MaxOutstandingNotificationsPerClient |整數，預設值為 1000 |動態|閘道在強制關閉用戶端註冊前可處於待處理狀態的通知數目上限。 |
| MaxIndexedEmptyPartitions |整數，預設值為 1000 |動態|通知快取中會保持已編製索引狀態以便同步處理重新連線用戶端的空白資料分割數上限。 任何超過此數目的空白資料分割，都將會以遞增查閱版本順序從索引中移除。 重新連線用戶端仍可同步處理和接收遺失的空白資料分割更新，但同步處理通訊協定會變得更耗費資源。 |
| GatewayServiceDescriptionCacheLimit |整數，預設值為 0 |靜態|命名閘道之 LRU 服務說明快取中保有的項目數上限 (設為 0 表示沒有限制)。 |
| PartitionCount |整數，預設值為 3 |不允許|要建立之命名服務存放區的資料分割數目。 每個資料分割都擁有一個與其索引對應的資料分割索引鍵，因此存在資料分割索引鍵 [0; PartitionCount)。 增加命名服務資料分割數目會增加命名服務可執行的級別，其方法是減少任何備用複本集所保有的平均資料量，代價則是資源的使用率會增加 (因為必須維持 PartitionCount*ReplicaSetSize 個服務複本)。|

### <a name="section-name-runas"></a>區段名稱：RunAs
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| RunAsAccountName |字串，預設值為 "" |動態|指出 RunAs 帳戶名稱。 "DomainUser" 或 "ManagedServiceAccount" 帳戶類型才需要此參數。 有效值為 "domain\user" 或 "user@domain"。 |
|RunAsAccountType|字串，預設值為 "" |動態|指出 RunAs 帳戶類型。 任何 RunAs 區段皆需要此參數。有效值為 "DomainUser/NetworkService/ManagedServiceAccount/LocalSystem"。|
|RunAsPassword|字串，預設值為 "" |動態|指出 RunAs 帳戶密碼。 "DomainUser" 帳戶類型才需要此參數。 |

### <a name="section-name-runasfabric"></a>區段名稱︰RunAs_Fabric
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| RunAsAccountName |字串，預設值為 "" |動態|指出 RunAs 帳戶名稱。 "DomainUser" 或 "ManagedServiceAccount" 帳戶類型才需要此參數。 有效值為 "domain\user" 或 "user@domain"。 |
|RunAsAccountType|字串，預設值為 "" |動態|指出 RunAs 帳戶類型。 任何 RunAs 區段皆需要此參數。有效值為 "LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem"。 |
|RunAsPassword|字串，預設值為 "" |動態|指出 RunAs 帳戶密碼。 "DomainUser" 帳戶類型才需要此參數。 |

### <a name="section-name-runashttpgateway"></a>區段名稱：RunAs_HttpGateway
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| RunAsAccountName |字串，預設值為 "" |動態|指出 RunAs 帳戶名稱。 "DomainUser" 或 "ManagedServiceAccount" 帳戶類型才需要此參數。 有效值為 "domain\user" 或 "user@domain"。 |
|RunAsAccountType|字串，預設值為 "" |動態|指出 RunAs 帳戶類型。 任何 RunAs 區段皆需要此參數。有效值為 "LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem"。 |
|RunAsPassword|字串，預設值為 "" |動態|指出 RunAs 帳戶密碼。 "DomainUser" 帳戶類型才需要此參數。 |

### <a name="section-name-runasdca"></a>區段名稱：RunAs_DCA
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| RunAsAccountName |字串，預設值為 "" |動態|指出 RunAs 帳戶名稱。 "DomainUser" 或 "ManagedServiceAccount" 帳戶類型才需要此參數。 有效值為 "domain\user" 或 "user@domain"。 |
|RunAsAccountType|字串，預設值為 "" |動態|指出 RunAs 帳戶類型。 任何 RunAs 區段皆需要此參數。有效值為 "LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem"。 |
|RunAsPassword|字串，預設值為 "" |動態|指出 RunAs 帳戶密碼。 "DomainUser" 帳戶類型才需要此參數。 |

### <a name="section-name-httpgateway"></a>區段名稱：HttpGateway
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
|IsEnabled|布林值，預設值為 false |靜態| 啟用/停用 HttpGateway。 預設會停用 HttpGateway。 |
|ActiveListeners |單位，預設值為 50 |靜態| 要張貼到 http 伺服器佇列的讀取數。 這會控制 HttpGateway 滿意的並行要求數目。 |
|MaxEntityBodySize |單位，預設值為 4194304 |動態|提供 http 要求預期會有的主體大小上限。 預設值為 4 MB。 若要求的主體大小大於此值，Httpgateway 會將要求判為失敗。 讀取區塊大小下限為 4096 個位元組。 因此，此值必須 >= 4096。 |
|HttpGatewayHealthReportSendInterval |時間 (秒)，預設值為 30 |靜態|以秒為單位指定時間範圍。 Http 閘道傳送累積的健康狀態報告至健康情況管理員的間隔時間。 |

### <a name="section-name-ktllogger"></a>區段名稱︰KtlLogger
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
|AutomaticMemoryConfiguration |整數，預設值為 1 |動態|表示是否應自動動態設定記憶體設定的旗標。 若為零，則會直接使用記憶體組態設定，而不會根據系統條件加以變更。 若為一，則會自動設定記憶體設定，而且可能會根據系統條件加以變更。 |
|WriteBufferMemoryPoolMinimumInKB |整數，預設值為 8388608 |動態|一開始要為寫入緩衝區記憶體集區配置的 KB 數目。 使用 0 表示無限制。預設值應與下面的 SharedLogSizeInMB 一致。 |
|WriteBufferMemoryPoolMaximumInKB | 整數，預設值為 0 |動態|允許寫入緩衝區記憶體集區成長達到的 KB 數目。 使用 0 表示無限制。 |
|MaximumDestagingWriteOutstandingInKB | 整數，預設值為 0 |動態|允許共用記錄比專用記錄多出的 KB 數目。 使用 0 表示無限制。
|SharedLogPath |字串，預設值為 "" |靜態|用以放置共用記錄容器之位置的路徑和檔案名稱。 使用 "" 可使用位於網狀架構資料根目錄下的預設路徑。 |
|SharedLogId |字串，預設值為 "" |靜態|共用記錄容器的唯一 GUID。 若使用位於網狀架構資料根目錄下的預設路徑，則使用 ""。 |
|SharedLogSizeInMB |整數，預設值為 8192 |靜態|要在共用記錄容器中配置的 MB 數。 |

### <a name="section-name-applicationgatewayhttp"></a>區段名稱︰ApplicationGateway/Http
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
|IsEnabled |布林值，預設值為 false |靜態| 啟用/停用 HttpApplicationGateway。 預設為停用 HttpApplicationGateway，必須設定此組態才能加以啟用。 |
|NumberOfParallelOperations | 單位，預設值為 5000 |靜態|要張貼到 http 伺服器佇列的讀取數。 這會控制 HttpGateway 滿意的並行要求數目。 |
|DefaultHttpRequestTimeout |以秒為單位的時間。 預設值為 120 |動態|以秒為單位指定時間範圍。  針對在 http 應用程式閘道中所處理的 http 要求提供預設要求逾時。 |
|ResolveServiceBackoffInterval |時間 (秒)，預設值為 5 |動態|以秒為單位指定時間範圍。  提供在重試失敗的解析服務作業前的預設輪詢間隔。 |
|BodyChunkSize |單位，預設值為 16384 |動態| 提供用來讀取主體的區塊大小 (位元組)。 |
|GatewayAuthCredentialType |字串，預設值為 "None" |靜態| 表示要在 http 應用程式閘道端點使用的安全性認證類型。有效值為 "None/X509。 |
|GatewayX509CertificateStoreName |字串，預設值為 "My" |動態| 包含 http 應用程式閘道之憑證的 X.509 憑證存放區名稱。 |
|GatewayX509CertificateFindType |字串，預設值為 "FindByThumbprint" |動態| 指出如何搜尋以下列 GatewayX509CertificateStoreName 支援值指定之存放區中的憑證︰FindByThumbprint、FindBySubjectName。 |
|GatewayX509CertificateFindValue | 字串，預設值為 "" |動態| 搜尋用來找出 http 應用程式閘道憑證的篩選值。 此憑證設定於 https 端點上，如果服務需要，也可用來驗證應用程式的身分識別。 首先會查閱 FindValue，如果不存在，則會查閱 FindValueSecondary。 |
|GatewayX509CertificateFindValueSecondary | 字串，預設值為 "" |動態|搜尋用來找出 http 應用程式閘道憑證的篩選值。 此憑證設定於 https 端點上，如果服務需要，也可用來驗證應用程式的身分識別。 首先會查閱 FindValue，如果不存在，則會查閱 FindValueSecondary。|
|HttpRequestConnectTimeout|時間範圍，預設值為 Common::TimeSpan::FromSeconds(5)|動態|以秒為單位指定時間範圍。  針對從 http 應用程式閘道所傳送的 http 要求提供連線逾時。  |
|RemoveServiceResponseHeaders|字串，預設值為 L"Date; Server"|靜態|會在轉送到用戶端之前，從服務回應中移除之回應標頭的分號/逗號分隔清單。 如果此參數設定為空字串，則會傳遞由服務依原狀傳回的所有標頭。 亦即 不會覆寫日期和伺服器 |
|ApplicationCertificateValidationPolicy|字串，預設值為 L"None"|靜態| ApplicationCertificateValidationPolicy：None：不驗證伺服器憑證，使要求成功。 ServiceCertificateThumbprints：請參閱 ServiceCertificateThumbprints 組態以取得反向 Proxy 可以信任的逗號分隔遠端憑證指紋清單。 ServiceCommonNameAndIssuer：請參閱 ServiceCommonNameAndIssuer 組態以取得反向 Proxy 可以信任之遠端憑證的主體名稱和簽發者指紋。 |
|ServiceCertificateThumbprints|字串，預設值為 L""|動態| |
|CrlCheckingFlag|單位，預設值為 0x40000000 |動態| 應用程式/服務憑證鏈結驗證旗標，例如 CRL 檢查 0x10000000 CERT_CHAIN_REVOCATION_CHECK_END_CERT 0x20000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN 0x40000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN_EXCLUDE_ROOT 0x80000000 CERT_CHAIN_REVOCATION_CHECK_CACHE_ONLY 設定為 0 會停用 CRL 檢查。CertGetCertificateChain 的 dwFlags 會記載完整的支援值清單：http://msdn.microsoft.com/library/windows/desktop/aa376078(v=vs.85).aspx  |
|IgnoreCrlOfflineError|布林值，預設值為 TRUE|動態|是否要忽略應用程式/服務憑證驗證的 CRL 離線錯誤。 |
|SecureOnlyMode|布林值，預設值為 FALSE|動態| SecureOnlyMode：true：反向 Proxy 只會轉送至發行安全端點的服務。 false：反向 Proxy 可以將要求轉送至安全/不安全的端點。  |
|ForwardClientCertificate|布林值，預設值為 FALSE|動態| |

### <a name="section-name-applicationgatewayhttpservicecommonnameandissuer"></a>區段名稱：ApplicationGateway/Http/ServiceCommonNameAndIssuer
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap，預設值為 None|動態|  |

### <a name="section-name-management"></a>區段名稱︰Management
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ImageStoreConnectionString |SecureString |靜態|ImageStore 根目錄的連接字串。 |
| ImageStoreMinimumTransferBPS | 整數，預設值為 1024 |動態|叢集與 ImageStore 之間的傳輸速率下限。 此值可用來決定外部 ImageStore 的存取逾時。 只有在叢集和 ImageStore 之間的延遲偏高時才需變更此值，以允許叢集有更多時間可從外部 ImageStore 進行下載。 |
|AzureStorageMaxWorkerThreads | 整數，預設值為 25 |動態|平行背景工作執行緒的數目上限。 |
|AzureStorageMaxConnections | 整數，預設值為 5000 |動態|Azure 儲存體的並行連線數目上限。 |
|AzureStorageOperationTimeout | 時間 (秒)，預設值為 6000 |動態|以秒為單位指定時間範圍。 可供 xstore 作業完成的逾時值。 |
|ImageCachingEnabled | 布林值，預設值為 true |靜態|此組態可讓我們啟用或停用快取。 |
|DisableChecksumValidation | 布林值，預設值為 false |靜態| 此組態可讓我們在應用程式佈建期間啟用或停用總和檢查碼驗證。 |
|DisableServerSideCopy | 布林值，預設值為 false |靜態|此組態會在應用程式佈建期間，啟用或停用 ImageStore 上應用程式套件的伺服器端複製作業。 |

### <a name="section-name-healthmanagerclusterhealthpolicy"></a>區段名稱︰HealthManager/ClusterHealthPolicy
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ConsiderWarningAsError |布林值，預設值為 false |靜態|叢集健康狀態評估原則︰將警告視為錯誤處理。 |
| MaxPercentUnhealthyNodes | 整數，預設值為 0 |靜態|叢集健康狀態評估原則︰要讓叢集被評估為狀況良好所允許的狀況不良節點百分比上限。 |
| MaxPercentUnhealthyApplications | 整數，預設值為 0 |靜態|叢集健康狀態評估原則︰要讓叢集被評估為狀況良好所允許的狀況不良應用程式百分比上限。 |

### <a name="section-name-healthmanagerclusterupgradehealthpolicy"></a>區段名稱︰HealthManager/ClusterUpgradeHealthPolicy
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
|MaxPercentDeltaUnhealthyNodes|整數，預設值為 10|靜態|叢集升級健康狀態評估原則︰要讓叢集被評估為狀況良好所允許的差異狀況不良節點百分比上限 |
|MaxPercentUpgradeDomainDeltaUnhealthyNodes|整數，預設值為 15|靜態|叢集升級健康狀態評估原則︰要讓叢集被評估為狀況良好所允許的升級網域中狀況不良節點差異百分比上限 |

### <a name="section-name-faultanalysisservice"></a>區段名稱︰FaultAnalysisService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| TargetReplicaSetSize |整數，預設值為 0 |不允許|NOT_PLATFORM_UNIX_START。FaultAnalysisService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize |整數，預設值為 0 |不允許|FaultAnalysisService 的 MinReplicaSetSize。 |
| ReplicaRestartWaitDuration |時間 (秒)，預設值為 60 分鐘|靜態|以秒為單位指定時間範圍。 FaultAnalysisService 的 ReplicaRestartWaitDuration。 |
| QuorumLossWaitDuration | 時間 (秒)，預設值為 MaxValue |靜態|以秒為單位指定時間範圍。 FaultAnalysisService 的 QuorumLossWaitDuration。 |
| StandByReplicaKeepDuration| 時間 (秒)，預設值為 (60*24*7) 分鐘 |靜態|以秒為單位指定時間範圍。 FaultAnalysisService 的 StandByReplicaKeepDuration。 |
| PlacementConstraints | 字串，預設值為 ""|靜態| FaultAnalysisService 的 PlacementConstraints。 |
| StoredActionCleanupIntervalInSeconds | 整數，預設值為 3600 |靜態|這是存放區的清除頻率。  只有處於終止狀態且在至少 CompletedActionKeepDurationInSeconds 秒前完成的動作會遭到移除。 |
| CompletedActionKeepDurationInSeconds | 整數，預設值為 604800 |靜態| 這大約是處於終止狀態之動作的保留時間長度。  此值也取決於 StoredActionCleanupIntervalInSeconds，因為要清除的工作只會在該間隔內完成。 604800 為 7 天。 |
| StoredChaosEventCleanupIntervalInSeconds | 整數，預設值為 3600 |靜態|這是會稽核存放區是否要清除的頻率，如果事件數目超過 30000，就會開始清除作業。 |
|DataLossCheckWaitDurationInSeconds|整數，預設值為 25|靜態|系統會等候資料發生遺失的總時間長度 (以秒為單位)。  呼叫了 StartPartitionDataLossAsync() API 時，便會於內部使用此參數。 |
|DataLossCheckPollIntervalInSeconds|整數，預設值為 5|靜態|這是系統在等候資料發生遺失時所執行之檢查的間隔時間。  系統會對每一內部反覆項目來檢查資料遺失數目的次數為 DataLossCheckWaitDurationInSeconds/此值。 |
|ReplicaDropWaitDurationInSeconds|整數，預設值為 600|靜態|呼叫了資料遺失 API 時，便會使用此參數。  此參數會控制在系統內部叫用移除複本之後，系統要等候多久才會捨棄複本。 |

### <a name="section-name-filestoreservice"></a>區段名稱︰FileStoreService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| NamingOperationTimeout |時間 (秒)，預設值為 60 |動態|以秒為單位指定時間範圍。 命名作業的執行逾時值。 |
| QueryOperationTimeout | 時間 (秒)，預設值為 60 |動態|以秒為單位指定時間範圍。 查詢作業的執行逾時值。 |
| MaxCopyOperationThreads | 單位，預設值為 0 |動態| 次要複寫器可從主要複寫器複製的平行檔案數上限。 '0' == 核心數目。 |
| MaxFileOperationThreads | 單位，預設值為 100 |靜態| 允許在主要複寫器執行 FileOperations (複製/移動) 的平行執行緒數目上限。 '0' == 核心數目。 |
| MaxStoreOperations | 單位，預設值為 4096 |靜態|主要複寫器上允許的平行存放區交易作業數上限。 '0' == 核心數目。 |
| MaxRequestProcessingThreads | 單位，預設值為 200 |靜態|允許在主要複寫器處理要求的平行執行緒數目上限。 '0' == 核心數目。 |
| MaxSecondaryFileCopyFailureThreshold | 單位，預設值為 25|動態|在放棄前要在次要複寫器上重試檔案複製的次數上限。 |
| AnonymousAccessEnabled | 布林值，預設值為 true |靜態|啟用/停用 FileStoreService 共用的匿名存取。 |
| PrimaryAccountType | 字串，預設值為 "" |靜態|要對 FileStoreService 共用進行 ACL 操作之主體的主要 AccountType。 |
| PrimaryAccountUserName | 字串，預設值為 "" |靜態|要對 FileStoreService 共用進行 ACL 操作之主體的主要帳戶使用者名稱。 |
| PrimaryAccountUserPassword | SecureString，預設值為空白 |靜態|要對 FileStoreService 共用進行 ACL 操作之主體的主要帳戶密碼。 |
| PrimaryAccountNTLMPasswordSecret | SecureString，預設值為空白 |靜態| 使用 NTLM 驗證時，用來做為種子以產生相同密碼的密碼祕密。 |
| PrimaryAccountNTLMX509StoreLocation | 字串，預設值為 "LocalMachine"|靜態| 使用 NTLM 驗證時，用來在 PrimaryAccountNTLMPasswordSecret 上產生 HMAC 之 X509 憑證的存放區位置。 |
| PrimaryAccountNTLMX509StoreName | 字串，預設值為 "MY"|靜態| 使用 NTLM 驗證時，用來在 PrimaryAccountNTLMPasswordSecret 上產生 HMAC 之 X509 憑證的存放區名稱。 |
| PrimaryAccountNTLMX509Thumbprint | 字串，預設值為 ""|靜態|使用 NTLM 驗證時，用來在 PrimaryAccountNTLMPasswordSecret 上產生 HMAC 之 X509 憑證的指紋。 |
| SecondaryAccountType | 字串，預設值為 ""|靜態| 要對 FileStoreService 共用進行 ACL 操作之主體的次要 AccountType。 |
| SecondaryAccountUserName | 字串，預設值為 ""| 靜態|要對 FileStoreService 共用進行 ACL 操作之主體的次要帳戶使用者名稱。 |
| SecondaryAccountUserPassword | SecureString，預設值為空白 |靜態|要對 FileStoreService 共用進行 ACL 操作之主體的次要帳戶密碼。  |
| SecondaryAccountNTLMPasswordSecret | SecureString，預設值為空白 |靜態| 使用 NTLM 驗證時，用來做為種子以產生相同密碼的密碼祕密。 |
| SecondaryAccountNTLMX509StoreLocation | 字串，預設值為 "LocalMachine" |靜態|使用 NTLM 驗證時，用來在 SecondaryAccountNTLMPasswordSecret 上產生 HMAC 之 X509 憑證的存放區位置。 |
| SecondaryAccountNTLMX509StoreName | 字串，預設值為 "MY" |靜態|使用 NTLM 驗證時，用來在 SecondaryAccountNTLMPasswordSecret 上產生 HMAC 之 X509 憑證的存放區名稱。 |
| SecondaryAccountNTLMX509Thumbprint | 字串，預設值為 ""| 靜態|使用 NTLM 驗證時，用來在 SecondaryAccountNTLMPasswordSecret 上產生 HMAC 之 X509 憑證的指紋。 |
|CommonNameNtlmPasswordSecret|SecureString，預設值為 Common::SecureString(L"")| 靜態|使用 NTLM 驗證時，用來做為種子以產生相同密碼的密碼祕密 |
|CommonName1Ntlmx509StoreLocation|字串，預設值為 L"LocalMachine"|靜態|使用 NTLM 驗證時，用來在 CommonName1NtlmPasswordSecret 上產生 HMAC 之 X509 憑證的存放區位置 |
|CommonName1Ntlmx509StoreName|字串，預設值為 L"MY"| 靜態|使用 NTLM 驗證時，用來在 CommonName1NtlmPasswordSecret 上產生 HMAC 之 X509 憑證的存放區名稱 |
|CommonName1Ntlmx509CommonName|字串，預設值為 L""|靜態| 使用 NTLM 驗證時，用來在 CommonName1NtlmPasswordSecret 上產生 HMAC 的 X509 憑證通用名稱 |
|CommonName2Ntlmx509StoreLocation|字串，預設值為 L"LocalMachine"| 靜態|使用 NTLM 驗證時，用來在 CommonName2NtlmPasswordSecret 上產生 HMAC 之 X509 憑證的存放區位置 |
|CommonName2Ntlmx509StoreName|字串，預設值為 L"MY"|靜態| 使用 NTLM 驗證時，用來在 CommonName2NtlmPasswordSecret 上產生 HMAC 之 X509 憑證的存放區名稱 |
|CommonName2Ntlmx509CommonName|字串，預設值為 L""|靜態|使用 NTLM 驗證時，用來在 CommonName2NtlmPasswordSecret 上產生 HMAC 的 X509 憑證通用名稱 |

### <a name="section-name-imagestoreservice"></a>區段名稱︰ImageStoreService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| 已啟用 |布林值，預設值為 false |靜態|ImageStoreService 的「啟用」旗標。 預設值：false |
| TargetReplicaSetSize | 整數，預設值為 7 |不允許|ImageStoreService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize | 整數，預設值為 3 |不允許|ImageStoreService 的 MinReplicaSetSize。 |
| ReplicaRestartWaitDuration | 時間 (秒)，預設值為 60.0 * 30 |靜態|以秒為單位指定時間範圍。 ImageStoreService 的 ReplicaRestartWaitDuration。 |
| QuorumLossWaitDuration | 時間 (秒)，預設值為 MaxValue |靜態| 以秒為單位指定時間範圍。 ImageStoreService 的 QuorumLossWaitDuration。 |
| StandByReplicaKeepDuration | 時間 (秒)，預設值為 3600.0 * 2 |靜態| 以秒為單位指定時間範圍。 ImageStoreService 的 StandByReplicaKeepDuration。 |
| PlacementConstraints | 字串，預設值為 "" |靜態| ImageStoreService 的 PlacementConstraints。 |

### <a name="section-name-imagestoreclient"></a>區段名稱︰ImageStoreClient
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ClientUploadTimeout |時間 (秒)，預設值為 1800 |動態|以秒為單位指定時間範圍。 目的地為映像存放區服務之頂層上傳要求的逾時值。 |
| ClientCopyTimeout | 時間 (秒)，預設值為 1800 |動態| 以秒為單位指定時間範圍。 目的地為映像存放區服務之頂層複製要求的逾時值。 |
|ClientDownloadTimeout | 時間 (秒)，預設值為 1800 |動態| 以秒為單位指定時間範圍。 目的地為映像存放區服務之頂層下載要求的逾時值。 |
|ClientListTimeout | 時間 (秒)，預設值為 600 |動態|以秒為單位指定時間範圍。 目的地為映像存放區服務之頂層列出要求的逾時值。 |
|ClientDefaultTimeout | 時間 (秒)，預設值為 180 |動態| 以秒為單位指定時間範圍。 目的地為映像存放區服務之所有非上傳/非下載要求 (例如，存在、刪除) 的逾時值。 |

### <a name="section-name-tokenvalidationservice"></a>區段名稱︰TokenValidationService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| 提供者 |字串，預設值為 "DSTS" |靜態|要啟用之權杖驗證提供者的逗號分隔清單 (有效提供者為︰DSTS、AAD)。 目前只能隨時啟用單一提供者。 |

### <a name="section-name-upgradeorchestrationservice"></a>區段名稱︰UpgradeOrchestrationService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| TargetReplicaSetSize |整數，預設值為 0 |不允許|UpgradeOrchestrationService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize |整數，預設值為 0 |不允許|UpgradeOrchestrationService 的 MinReplicaSetSize。
| ReplicaRestartWaitDuration | 時間 (秒)，預設值為 60 分鐘|靜態| 以秒為單位指定時間範圍。 UpgradeOrchestrationService 的 ReplicaRestartWaitDuration。 |
| QuorumLossWaitDuration | 時間 (秒)，預設值為 MaxValue |靜態| 以秒為單位指定時間範圍。 UpgradeOrchestrationService 的 QuorumLossWaitDuration。 |
| StandByReplicaKeepDuration | 時間 (秒)，預設值為 60*24*7 分鐘 |靜態| 以秒為單位指定時間範圍。 UpgradeOrchestrationService 的 StandByReplicaKeepDuration。 |
| PlacementConstraints | 字串，預設值為 "" |靜態| UpgradeOrchestrationService 的 PlacementConstraints。 |
| AutoupgradeEnabled | 布林值，預設值為 true |靜態| 以目標狀態檔案為基礎的自動輪詢和升級動作。 |
| UpgradeApprovalRequired | 布林值，預設值為 false | 靜態|讓程式碼升級作業需要系統管理員核准後才能繼續的設定。 |

### <a name="section-name-upgradeservice"></a>區段名稱︰UpgradeService
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| PlacementConstraints |字串，預設值為 "" |不允許|升級服務的 PlacementConstraints。 |
| TargetReplicaSetSize | 整數，預設值為 3 |不允許| UpgradeService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize | 整數，預設值為 2 |不允許| UpgradeService 的 MinReplicaSetSize。 |
| CoordinatorType | 字串，預設值為 "WUTest"|不允許|UpgradeService 的 CoordinatorType。 |
| BaseUrl | 字串，預設值為 "" |靜態|UpgradeService 的 BaseUrl。 |
| ClusterId | 字串，預設值為 "" |靜態|UpgradeService 的 ClusterId。 |
| X509StoreName | 字串，預設值為 "My"|動態|UpgradeService 的 X509StoreName。 |
| X509StoreLocation | 字串，預設值為 "" |動態| UpgradeService 的 X509StoreLocation。 |
| X509FindType | 字串，預設值為 ""|動態| UpgradeService 的 X509FindType。 |
| X509FindValue | 字串，預設值為 "" |動態| UpgradeService 的 X509FindValue。 |
| X509SecondaryFindValue | 字串，預設值為 "" |動態| UpgradeService 的 X509SecondaryFindValue。 |
| OnlyBaseUpgrade | 布林值，預設值為 false |動態|UpgradeService 的 OnlyBaseUpgrade。 |
| TestCabFolder | 字串，預設值為 "" |靜態| UpgradeService 的 TestCabFolder。 |

### <a name="section-name-security"></a>區段名稱︰Security
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|ClusterCredentialType|字串，預設值為 L"None"|不允許|指出為了保護叢集而要使用的安全性認證類型。 有效值為"None/X509/Windows" |
|ServerAuthCredentialType|字串，預設值為 L"None"|靜態|指出為了保護 FabricClient 和叢集間通訊而要使用的安全性認證類型。 有效值為"None/X509/Windows" |
|ClientRoleEnabled|布林值，預設值為 FALSE|靜態|指出是否啟用用戶端角色，在設定為 true 時，會根據用戶端的身分識別對其指派角色。 對於 V2，啟用此參數表示不在 AdminClientCommonNames/AdminClientIdentities 內的用戶端只能執行唯讀作業。 |
|ClusterCertThumbprints|字串，預設值為 L""|動態|允許加入到叢集的憑證指紋，這是以逗號分隔的名稱清單。 |
|ServerCertThumbprints|字串，預設值為 L""|動態|叢集用來與用戶端通訊的伺服器憑證指紋，用戶端會使用此指紋來驗證叢集。 這是以逗號分隔的名稱清單。 |
|ClientCertThumbprints|字串，預設值為 L""|動態|用戶端用來與叢集通訊的憑證指紋，叢集會使用此指紋來授權連入連線。 這是以逗號分隔的名稱清單。 |
|AdminClientCertThumbprints|字串，預設值為 L""|動態|系統管理員角色的用戶端所使用的憑證指紋。 這是以逗號分隔的名稱清單。 |
|CrlCheckingFlag|單位，預設值為 0x40000000|動態|預設憑證鏈結驗證旗標，可能會由元件特定的旗標加以覆寫，例如，同盟/X509CertChainFlags 0x10000000 CERT_CHAIN_REVOCATION_CHECK_END_CERT 0x20000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN 0x40000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN_EXCLUDE_ROOT 0x80000000 CERT_CHAIN_REVOCATION_CHECK_CACHE_ONLY 設定為 0 會停用 CRL 檢查。CertGetCertificateChain 的 dwFlags 會記載完整的支援值清單：http://msdn.microsoft.com/library/windows/desktop/aa376078(v=vs.85).aspx |
|IgnoreCrlOfflineError|布林值，預設值為 FALSE|動態|是否要在伺服器端驗證內送用戶端憑證時忽略 CRL 離線錯誤 |
|IgnoreSvrCrlOfflineError|布林值，預設值為 TRUE|動態|是否要在用戶端驗證內送伺服器憑證時忽略 CRL 離線錯誤，預設為 true。 使用已撤銷的伺服器憑證來進行攻擊需要入侵 DNS，其難度比使用已撤銷的用戶端憑證更高。 |
|CrlDisablePeriod|時間範圍，預設值為 Common::TimeSpan::FromMinutes(15)|動態|以秒為單位指定時間範圍。 如果 CRL 離線錯誤是可忽略的，則在遇到離線錯誤之後，要將給定憑證的 CRL 檢查停用多久。 |
|CrlOfflineHealthReportTtl|時間範圍，預設值為 Common::TimeSpan::FromMinutes(1440)|動態|以秒為單位指定時間範圍。 |
|CertificateHealthReportingInterval|時間範圍，預設值為 Common::TimeSpan::FromSeconds(3600 * 8)|靜態|以秒為單位指定時間範圍。 指定憑證健康情況報告的間隔，預設為 8 個小時，設定為 0 會停用憑證健康情況報告 |
|CertificateExpirySafetyMargin|時間範圍，預設值為 Common::TimeSpan::FromMinutes(43200)|靜態|以秒為單位指定時間範圍。 憑證到期日的安全邊際，當到期日接近此邊際值時，憑證健康情況報告的狀態會從「正常」變更為「警告」。 預設值為 30 天。 |
|ClientClaimAuthEnabled|布林值，預設值為 FALSE|靜態|指出是否在用戶端啟用宣告型驗證，將此參數設定為 true 會隱含設定 ClientRoleEnabled。 |
|ClientClaims|字串，預設值為 L""|動態|用戶端預期會發出之用來連線至閘道的所有可能宣告。 這是 'OR' 清單：ClaimsEntry || ClaimsEntry || ClaimsEntry ... 每個 ClaimsEntry 都是一個 "AND" 清單：ClaimType=ClaimValue && ClaimType=ClaimValue && ClaimType=ClaimValue ... |
|AdminClientClaims|字串，預設值為 L""|動態|系統管理員用戶端預期會發出的所有可能宣告，其格式與 ClientClaims 相同，此清單會於內部新增至 ClientClaims，因此不必再將相同的項目新增至 ClientClaims。 |
|ClusterSpn|字串，預設值為 L""|不允許|當網狀架構以單一網域使用者 (gMSA/網域使用者帳戶) 的身分執行時，叢集的服務主體名稱。 它是租用接聽程式和 fabric.exe 中的接聽程式的 SPN：同盟接聽程式、內部複寫接聽程式、執行階段服務接聽程式和命名閘道接聽程式。 當網狀架構以機器帳戶的身分執行時，此參數應該保持空白，在此情況下，來自接聽程式傳輸位址的連線端計算接聽程式 SPN。 |
|ClusterIdentities|字串，預設值為 L""|動態|用於叢集成員資格授權的叢集節點 Windows 身分識別。 它是以逗號分隔的清單，每個項目都是網域帳戶名稱或群組名稱 |
|ClientIdentities|字串，預設值為 L""|動態|FabricClient 的 Windows 身分識別，命名閘道會使用此參數來授權連入連線。 它是以逗號分隔的清單，每個項目都是網域帳戶名稱或群組名稱。 為了方便，系統會自動允許執行 fabric.exe 的帳戶，ServiceFabricAllowedUsers 和 ServiceFabricAdministrators 群組也是如此。 |
|AdminClientIdentities|字串，預設值為 L""|動態|系統管理員角色之網狀架構用戶端的 Windows 身分識別，可用來授權特殊權限的網狀架構作業。 它是以逗號分隔的清單，每個項目都是網域帳戶名稱或群組名稱。 為了方便，系統會自動對執行 fabric.exe 的帳戶指派系統管理員角色，ServiceFabricAdministrators 群組也是如此。 |
|AADTenantId|字串，預設值為 L""|靜態|租用戶識別碼 (GUID) |
|AADClusterApplication|字串，預設值為 L""|靜態|代表叢集的 Web API 應用程式名稱或識別碼 |
|AADClientApplication|字串，預設值為 L""|靜態|代表網狀架構用戶端的原生用戶端應用程式名稱或識別碼 |
|X509Folder|字串，預設值為 /var/lib/waagent|靜態|X509 憑證和私密金鑰的所在資料夾 |
|FabricHostSpn| 字串，預設值為 L"" |靜態| 當網狀架構以單一網域使用者 (gMSA/網域使用者帳戶) 的身分執行，而且 FabricHost 是在機器帳戶下執行時，FabricHost 的服務主體名稱。 它是 FabricHost 的 IPC 接聽程式 SPN，因為 FabricHost 是在機器帳戶下執行，因此依預設應該保持空白 |
|DisableFirewallRuleForPublicProfile| 布林值，預設值為 TRUE | 靜態|指出是否不應該對公用設定檔啟用防火牆規則 |
|DisableFirewallRuleForPrivateProfile| 布林值，預設值為 TRUE |靜態| 指出是否不應該對私人設定檔啟用防火牆規則 | 
|DisableFirewallRuleForDomainProfile| 布林值，預設值為 TRUE |靜態| 指出是否不應該對網域設定檔啟用防火牆規則 |
|SettingsX509StoreName| 字串，預設值為 L"MY"| 動態|網狀架構用來保護組態的 X509 憑證存放區 |

### <a name="section-name-securityadminclientx509names"></a>區段名稱：Security/AdminClientX509Names
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap，預設值為 None|動態| |

### <a name="section-name-securityclientx509names"></a>區段名稱：Security/ClientX509Names
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
PropertyGroup|X509NameMap，預設值為 None|動態| |

### <a name="section-name-securityclusterx509names"></a>區段名稱：Security/ClusterX509Names
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
PropertyGroup|X509NameMap，預設值為 None|動態| |

### <a name="section-name-securityserverx509names"></a>區段名稱：Security/ServerX509Names
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
PropertyGroup|X509NameMap，預設值為 None|動態| |

### <a name="section-name-securityclientaccess"></a>區段名稱︰Security/ClientAccess
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| CreateName |字串，預設值為 "Admin" |動態|命名 URI 建立的安全性組態。 |
| DeleteName |字串，預設值為 "Admin" |動態|命名 URI 刪除的安全性組態。 |
| PropertyWriteBatch |字串，預設值為 "Admin" |動態|命名屬性寫入作業的安全性組態。 |
| CreateService |字串，預設值為 "Admin" |動態| 服務建立的安全性組態。 |
| CreateServiceFromTemplate |字串，預設值為 "Admin" |動態|從範本建立服務的安全性組態。 |
| UpdateService |字串，預設值為 "Admin" |動態|服務更新的安全性組態。 |
| DeleteService  |字串，預設值為 "Admin" |動態|服務刪除的安全性組態。 |
| ProvisionApplicationType |字串，預設值為 "Admin" |動態| 應用程式類型佈建的安全性組態。 |
| CreateApplication |字串，預設值為 "Admin" | 動態|應用程式建立的安全性組態。 |
| DeleteApplication |字串，預設值為 "Admin" |動態| 應用程式刪除的安全性組態。 |
| UpgradeApplication |字串，預設值為 "Admin" |動態| 用於啟動或中斷應用程式升級的安全性組態。 |
| RollbackApplicationUpgrade |字串，預設值為 "Admin" |動態| 用於復原應用程式升級的安全性組態。 |
| UnprovisionApplicationType |字串，預設值為 "Admin" |動態| 應用程式類型取消佈建的安全性組態。 |
| MoveNextUpgradeDomain |字串，預設值為 "Admin" |動態| 用於以明確的「升級網域」繼續進行應用程式升級的安全性組態。 |
| ReportUpgradeHealth |字串，預設值為 "Admin" |動態| 用於以目前的升級進度繼續進行應用程式升級的安全性組態。 |
| ReportHealth |字串，預設值為 "Admin" |動態| 用於報告健康狀態的安全性組態。 |
| ProvisionFabric |字串，預設值為 "Admin" |動態| MSI 和/或叢集資訊清單佈建的安全性組態。 |
| UpgradeFabric |字串，預設值為 "Admin" |動態| 用於啟動叢集升級的安全性組態。 |
| RollbackFabricUpgrade |字串，預設值為 "Admin" |動態| 用於復原叢集升級的安全性組態。 |
| UnprovisionFabric |字串，預設值為 "Admin" |動態| MSI 和/或叢集資訊清單取消佈建的安全性組態。 |
| MoveNextFabricUpgradeDomain |字串，預設值為 "Admin" |動態| 用於以明確的「升級網域」繼續進行叢集升級的安全性組態。 |
| ReportFabricUpgradeHealth |字串，預設值為 "Admin" |動態| 用於以目前的升級進度繼續進行叢集升級的安全性組態。 |
| StartInfrastructureTask |字串，預設值為 "Admin" | 動態|用於啟動基礎結構工作的安全性組態。 |
| FinishInfrastructureTask |字串，預設值為 "Admin" |動態| 用於完成基礎結構工作的安全性組態。 |
| ActivateNode |字串，預設值為 "Admin" |動態| 用於啟用節點的安全性組態。 |
| DeactivateNode |字串，預設值為 "Admin" |動態| 用於停用節點的安全性組態。 |
| DeactivateNodesBatch |字串，預設值為 "Admin" |動態| 用於停用多個節點的安全性組態。 |
| RemoveNodeDeactivations |字串，預設值為 "Admin" |動態| 用於將停用多個節點之作業還原的安全性組態。 |
| GetNodeDeactivationStatus |字串，預設值為 "Admin" |動態| 用於檢查停用狀態的安全性組態。 |
| NodeStateRemoved |字串，預設值為 "Admin" |動態| 用於報告節點狀態已遭移除的安全性組態。 |
| RecoverPartition |字串，預設值為 "Admin" | 動態|用於復原資料分割的安全性組態。 |
| RecoverPartitions |字串，預設值為 "Admin" | 動態|用於復原資料分割的安全性組態。 |
| RecoverServicePartitions |字串，預設值為 "Admin" |動態| 用於復原服務資料分割的安全性組態。 |
| RecoverSystemPartitions |字串，預設值為 "Admin" |動態| 用於復原系統服務資料分割的安全性組態。 |
| ReportFault |字串，預設值為 "Admin" |動態| 用於報告錯誤的安全性組態。 |
| InvokeInfrastructureCommand |字串，預設值為 "Admin" |動態| 基礎結構工作管理命令的安全性組態。 |
| FileContent |字串，預設值為 "Admin" |動態| 映像存放區用戶端檔案傳輸 (叢集外部) 的安全性組態。 |
| FileDownload |字串，預設值為 "Admin" |動態| 映像存放區用戶端檔案下載起始 (叢集外部) 的安全性組態。 |
| InternalList |字串，預設值為 "Admin" | 動態|映像存放區用戶端檔案列出作業 (內部) 的安全性組態。 |
| 刪除 |字串，預設值為 "Admin" |動態| 映像存放區用戶端刪除作業的安全性組態。 |
| 上傳 |字串，預設值為 "Admin" | 動態|映像存放區用戶端上傳作業的安全性組態。 |
| GetStagingLocation |字串，預設值為 "Admin" |動態| 映像存放區用戶端預備位置擷取的安全性組態。 |
| GetStoreLocation |字串，預設值為 "Admin" |動態| 映像存放區用戶端存放區位置擷取的安全性組態。 |
| NodeControl |字串，預設值為 "Admin" |動態| 用於啟動、停止和重新啟動節點的安全性組態。 |
| CodePackageControl |字串，預設值為 "Admin" |動態| 用於重新啟動程式碼套件的安全性組態。 |
| UnreliableTransportControl |字串，預設值為 "Admin" |動態| 用於新增和移除行為的不可靠傳輸。 |
| MoveReplicaControl |字串，預設值為 "Admin" | 動態|移動複本。 |
| PredeployPackageToNode |字串，預設值為 "Admin" |動態| 預先部署 API。 |
| StartPartitionDataLoss |字串，預設值為 "Admin" |動態| 會在資料分割上引發資料遺失。 |
| StartPartitionQuorumLoss |字串，預設值為 "Admin" |動態| 會在資料分割上引發仲裁遺失。 |
| StartPartitionRestart |字串，預設值為 "Admin" |動態| 同時重新啟動部分或所有的資料分割複本。 |
| CancelTestCommand |字串，預設值為 "Admin" |動態| 取消傳輸中的特定 TestCommand。 |
| StartChaos |字串，預設值為 "Admin" |動態| 啟動尚未啟動的混亂。 |
| StopChaos |字串，預設值為 "Admin" |動態| 停止已啟動的混亂。 |
| StartNodeTransition |字串，預設值為 "Admin" |動態| 用於啟動節點轉換的安全性組態。 |
| StartClusterConfigurationUpgrade |字串，預設值為 "Admin" |動態| 在資料分割上引發 StartClusterConfigurationUpgrade。 |
| GetUpgradesPendingApproval |字串，預設值為 "Admin" |動態| 在資料分割上引發 GetUpgradesPendingApproval。 |
| StartApprovedUpgrades |字串，預設值為 "Admin" |動態| 在資料分割上引發 StartApprovedUpgrades。 |
| Ping |字串，預設值為 "Admin\|\|User" |動態| 用於 Ping 用戶端的安全性組態。 |
| 查詢 |字串，預設值為 "Admin\|\|User" |動態| 查詢的安全性組態。 |
| NameExists |字串，預設值為 "Admin\|\|User" | 動態|命名 URI 存在檢查的安全性組態。 |
| EnumerateSubnames |字串，預設值為 "Admin\|\|User" |動態| 命名 URI 列舉的安全性組態。 |
| EnumerateProperties |字串，預設值為 "Admin\|\|User" | 動態|命名屬性列舉的安全性組態。 |
| PropertyReadBatch |字串，預設值為 "Admin\|\|User" |動態| 命名屬性讀取作業的安全性組態。 |
| GetServiceDescription |字串，預設值為 "Admin\|\|User" |動態| 用於長時間輪詢服務通知和讀取服務描述的安全性組態。 |
| ResolveService |字串，預設值為 "Admin\|\|User" |動態| 抱怨型服務解析的安全性組態。 |
| ResolveNameOwner |字串，預設值為 "Admin\|\|User" | 動態|用於解析命名 URI 擁有者的安全性組態。 |
| ResolvePartition |字串，預設值為 "Admin\|\|User" | 動態|用於解析系統服務的安全性組態。 |
| ServiceNotifications |字串，預設值為 "Admin\|\|User" |動態| 事件型服務通知的安全性組態。 |
| PrefixResolveService |字串，預設值為 "Admin\|\|User" |動態| 抱怨型服務前置詞解析的安全性組態。 |
| GetUpgradeStatus |字串，預設值為 "Admin\|\|User" |動態| 用於輪詢應用程式升級狀態的安全性組態。 |
| GetFabricUpgradeStatus |字串，預設值為 "Admin\|\|User" |動態| 用於輪詢叢集升級狀態的安全性組態。 |
| InvokeInfrastructureQuery |字串，預設值為 "Admin\|\|User" | 動態|用於查詢基礎結構工作的安全性組態。 |
| 列出 |字串，預設值為 "Admin\|\|User" | 動態|映像存放區用戶端檔案列出作業的安全性組態。 |
| ResetPartitionLoad |字串，預設值為 "Admin\|\|User" |動態| 用於重設 failoverUnit 負載的安全性組態。 |
| ToggleVerboseServicePlacementHealthReporting | 字串，預設值為 "Admin\|\|User" |動態| 用於切換詳細 ServicePlacement HealthReporting 的安全性組態。 |
| GetPartitionDataLossProgress | 字串，預設值為 "Admin\|\|User" | 動態|擷取叫用資料遺失 API 呼叫的進度。 |
| GetPartitionQuorumLossProgress | 字串，預設值為 "Admin\|\|User" |動態| 擷取叫用仲裁遺失 API 呼叫的進度。 |
| GetPartitionRestartProgress | 字串，預設值為 "Admin\|\|User" |動態| 擷取重新啟動資料分割 API 呼叫的進度。 |
| GetChaosReport | 字串，預設值為 "Admin\|\|User" |動態| 擷取指定時間範圍內的混亂狀態。 |
| GetNodeTransitionProgress | 字串，預設值為 "Admin\|\|User" |動態| 用於取得節點轉換命令進度的安全性組態。 |
| GetClusterConfigurationUpgradeStatus | 字串，預設值為 "Admin\|\|User" |動態| 在資料分割上引發 GetClusterConfigurationUpgradeStatus。 |
| GetClusterConfiguration | 字串，預設值為 "Admin\|\|User" | 動態|在資料分割上引發 GetClusterConfiguration。 |
|CreateComposeDeployment|字串，預設值為 L"Admin"| 動態|建立撰寫檔案所描述的撰寫部署 |
|DeleteComposeDeployment|字串，預設值為 L"Admin"| 動態|刪除撰寫部署 |
|UpgradeComposeDeployment|字串，預設值為 L"Admin"| 動態|升級撰寫部署 |
|ResolveSystemService|字串，預設值為 L"Admin\|\|User"|動態| 用於解析系統服務的安全性組態 |
|GetUpgradeOrchestrationServiceState|字串，預設值為 L"Admin"| 動態|在資料分割上引發 GetUpgradeOrchestrationServiceState |
|SetUpgradeOrchestrationServiceState|字串，預設值為 L"Admin"| 動態|在資料分割上引發 SetUpgradeOrchestrationServiceState |

### <a name="section-name-reconfigurationagent"></a>區段名稱：ReconfigurationAgent
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ApplicationUpgradeMaxReplicaCloseDuration | 時間 (秒)，預設值為 900 |動態|以秒為單位指定時間範圍。 應用程式升級期間，系統在終止有複本卡在關閉作業的服務主機之前會等候的持續時間。|
| ServiceApiHealthDuration | 時間 (秒)，預設值為 30 分鐘 |動態| 以秒為單位指定時間範圍。 ServiceApiHealthDuration 會定義我們在報告服務 API 狀況不良前，會等候多久時間來讓它執行。 |
| ServiceReconfigurationApiHealthDuration | 時間 (秒)，預設值為 30 |動態| 以秒為單位指定時間範圍。 ServiceReconfigurationApiHealthDuration 會定義我們在報告服務 API 狀況不良前，會等候多久時間來讓它執行。 這適用於會影響可用性的 API 呼叫。|
| PeriodicApiSlowTraceInterval | 時間 (秒)，預設值為 5 分鐘 |動態| 以秒為單位指定時間範圍。 PeriodicApiSlowTraceInterval 會定義 API 監視器重新追蹤慢速 API 呼叫的間隔時間。 |
| NodeDeactivationMaxReplicaCloseDuration | 時間 (秒)，預設值為 900 |動態|以秒為單位指定時間範圍。 節點停用期間，系統在終止有複本卡在關閉作業的服務主機之前會等候的持續時間。 |
| FabricUpgradeMaxReplicaCloseDuration | 時間 (秒)，預設值為 900 |動態| 以秒為單位指定時間範圍。 網狀架構升級期間，系統在終止有複本卡在關閉作業的服務主機之前會等候的持續時間。 |
|GracefulReplicaShutdownMaxDuration|時間範圍，預設值為 Common::TimeSpan::FromSeconds(120)|動態|以秒為單位指定時間範圍。 系統在終止有複本卡在關閉作業的服務主機之前會等候的持續時間。|
|ReplicaChangeRoleFailureRestartThreshold|整數，預設值為 10|動態| 整數。 指定主要升階期間的 API 失敗次數，超過此次數後，系統便會套用自動補救動作 (複本重新啟動)。 |
|ReplicaChangeRoleFailureWarningReportThreshold|整數，預設值為 2147483647|動態| 整數。 指定主要升階期間的 API 失敗次數，超過此次數後，系統便會引發警告健康情況報告。|

### <a name="section-name-placementandloadbalancing"></a>區段名稱︰PlacementAndLoadBalancing
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| TraceCRMReasons |布林值，預設值為 true |動態|指定是否要追蹤 CRM 所發出移動至作業事件通道的原因。 |
| ValidatePlacementConstraint | 布林值，預設值為 true |動態| 指定在服務的 ServiceDescription 更新時，是否要驗證服務的 PlacementConstraint 運算式。 |
| PlacementConstraintValidationCacheSize | 整數，預設值為 10000 |動態| 用來進行快速驗證和快取放置條件約束運算式的資料表大小限制。 |
|VerboseHealthReportLimit | 整數，預設值為 20 | 動態|定義在針對複本報告健康狀態警告之前，複本必須未放置的次數 (如果已啟用詳細健康狀態報告)。 |
|ConstraintViolationHealthReportLimit | 整數，預設值為 50 |動態| 定義在執行診斷和發出健康狀態報告之前，違反條件約束的複本必須持續未修正的次數。 |
|DetailedConstraintViolationHealthReportLimit | 整數，預設值為 200 |動態| 定義在執行診斷和發出詳細的健康狀態報告之前，違反條件約束的複本必須持續未修正的次數。 |
|DetailedVerboseHealthReportLimit | 整數，預設值為 200 | 動態|定義在發出詳細的健康狀態報告之前，未放置的複本必須持續未放置的次數。 |
|ConsecutiveDroppedMovementsHealthReportLimit | 整數，預設值為 20 | 動態|定義在執行診斷和發出健康狀態警告之前，ResourceBalancer 所發出之移動遭到捨棄的連續次數。 負數︰不會在此狀況下發出任何警告。 |
|DetailedNodeListLimit | 整數，預設值為 15 |動態| 定義在截斷之前，要在未放置的複本報告中包含的每一條件約束之節點數目。 |
|DetailedPartitionListLimit | 整數，預設值為 15 |動態| 定義在截斷之前，要在診斷中包含之條件約束每一診斷項目的資料分割數目。 |
|DetailedDiagnosticsInfoListLimit | 整數，預設值為 15 |動態| 定義在截斷之前，要在診斷中包含之每一條件約束的診斷項目 (含詳細資訊) 數目。|
|PLBRefreshGap | 時間 (秒)，預設值為 1 |動態| 以秒為單位指定時間範圍。 定義在 PLB 再次重新整理狀態之前，必須經過的時間量下限。 |
|MinPlacementInterval | 時間 (秒)，預設值為 1 |動態| 以秒為單位指定時間範圍。 定義在連續兩回放置之前，必須經過的時間量下限。 |
|MinConstraintCheckInterval | 時間 (秒)，預設值為 1 |動態| 以秒為單位指定時間範圍。 定義在連續兩回條件約束檢查之前，必須經過的時間量下限。 |
|MinLoadBalancingInterval | 時間 (秒)，預設值為 5 |動態| 以秒為單位指定時間範圍。 定義在連續兩回平衡之前，必須經過的時間量下限。 |
|BalancingDelayAfterNodeDown | 時間 (秒)，預設值為 120 |動態|以秒為單位指定時間範圍。 在節點關閉事件之後，不要在此期間啟動平衡活動。 |
|BalancingDelayAfterNewNode | 時間 (秒)，預設值為 120 |動態|以秒為單位指定時間範圍。 在新增節點之後，不要在此期間啟動平衡活動。 |
|ConstraintFixPartialDelayAfterNodeDown | 時間 (秒)，預設值為 120 |動態| 以秒為單位指定時間範圍。 在節點關閉事件之後，不要在此期間修正 FaultDomain 和 UpgradeDomain 條件約束違規。 |
|ConstraintFixPartialDelayAfterNewNode | 時間 (秒)，預設值為 120 |動態| 以秒為單位指定時間範圍。 在新增節點之後，不要在此期間修正 FaultDomain 和 UpgradeDomain 條件約束違規。 |
|GlobalMovementThrottleThreshold | 單位，預設值為 1000 |動態| 在 GlobalMovementThrottleCountingInterval 所指出的過去間隔中，平衡階段所允許的移動數目上限。 |
|GlobalMovementThrottleThresholdForPlacement | 單位，預設值為 0 |動態| 在 GlobalMovementThrottleCountingInterval 所指出的過去間隔中，放置階段所允許的移動數目上限。0 表示沒有限制。|
|GlobalMovementThrottleThresholdForBalancing | 單位，預設值為 0 | 動態|在 GlobalMovementThrottleCountingInterval 所指出的過去間隔中，平衡階段所允許的移動數目上限。 0 表示沒有限制。 |
|GlobalMovementThrottleCountingInterval | 時間 (秒)，預設值為 600 |靜態| 以秒為單位指定時間範圍。 表示要追蹤每一網域複本移動 (搭配使用 GlobalMovementThrottleThreshold) 的過去間隔時間長度。 可設為 0，以完全忽略全域節流。 |
|MovementPerPartitionThrottleThreshold | 單位，預設值為 50 |動態| 如果資料分割之複本的平衡相關移動數目，已達到或超過 MovementPerPartitionThrottleCountingInterval 所指出之過去間隔中的 MovementPerFailoverUnitThrottleThreshold，該資料分割不會發生任何平衡相關移動。 |
|MovementPerPartitionThrottleCountingInterval | 時間 (秒)，預設值為 600 |靜態| 以秒為單位指定時間範圍。 表示要追蹤每個資料分割之複本移動 (搭配使用 MovementPerPartitionThrottleThreshold) 的過去間隔時間長度。 |
|PlacementSearchTimeout | 時間 (秒)，預設值為 0.5 |動態| 以秒為單位指定時間範圍。 在放置服務時，至少搜尋這麼長的時間再傳回結果。 |
|UseMoveCostReports | 布林值，預設值為 false | 動態|指示 LB 略過評分函式的成本項目，以便有機會產生大量移動來獲得更平衡的放置。 |
|PreventTransientOvercommit | 布林值，預設值為 false | 動態|決定 PLB 是否應立即計算所起始之移動將釋放的資源。 根據預設，PLB 可以在相同節點上起始移出和移入，以便能建立暫時性的過量使用。 將此參數設為 true 將會避免這類過量使用，且會停用隨選重組 (也稱為 placementWithMove)。 |
|InBuildThrottlingEnabled | 布林值，預設值為 false |動態| 決定是否啟用內建的節流。 |
|InBuildThrottlingAssociatedMetric | 字串，預設值為 "" |靜態| 此節流的相關度量名稱。 |
|InBuildThrottlingGlobalMaxValue | 整數，預設值為 0 |動態|在全域中允許的內建複本數目上限。 |
|SwapPrimaryThrottlingEnabled | 布林值，預設值為 false|動態| 決定是否啟用 swap-primary 節流。 |
|SwapPrimaryThrottlingAssociatedMetric | 字串，預設值為 ""|靜態| 此節流的相關度量名稱。 |
|SwapPrimaryThrottlingGlobalMaxValue | 整數，預設值為 0 |動態| 在全域中允許的 swap-primary 複本數目上限。 |
|PlacementConstraintPriority | 整數，預設值為 0 | 動態|決定放置條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|PreferredLocationConstraintPriority | 整數，預設值為 2| 動態|決定慣用位置條件約束的優先順序︰0︰硬式、1︰軟式、2：最佳化、負數︰忽略 |
|CapacityConstraintPriority | 整數，預設值為 0 | 動態|決定容量條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|AffinityConstraintPriority | 整數，預設值為 0 | 動態|決定同質性條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|FaultDomainConstraintPriority | 整數，預設值為 0 |動態| 決定容錯網域條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|UpgradeDomainConstraintPriority | 整數，預設值為 1| 動態|決定升級網域條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|ScaleoutCountConstraintPriority | 整數，預設值為 0 |動態| 決定向外擴充計數條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|ApplicationCapacityConstraintPriority | 整數，預設值為 0 | 動態|決定容量條件約束的優先順序︰0︰硬式、1︰軟式、負數︰忽略。 |
|MoveParentToFixAffinityViolation | 布林值，預設值為 false |動態| 決定是否可以移動父系複本以修正同質性條件約束的設定。|
|MoveExistingReplicaForPlacement | 布林值，預設值為 true |動態|決定是否要在放置期間移動現有複本的設定。 |
|UseSeparateSecondaryLoad | 布林值，預設值為 true | 動態|決定是否使用不同的次要負載的設定。 |
|PlaceChildWithoutParent | 布林值，預設值為 true | 動態|決定當父系複本皆未運作時，是否可以放置子服務複本的設定。 |
|PartiallyPlaceServices | 布林值，預設值為 true |動態| 決定當叢集中的所有服務複本適合使用的節點有限時，是要「全都或全不」放置。|
|InterruptBalancingForAllFailoverUnitUpdates | 布林值，預設值為 false | 動態|決定任何類型的容錯移轉單元更新是否應中斷快速或緩慢的平衡執行。 若指定 "false"，當 FailoverUnit 符合下列條件時，平衡執行將會中斷︰已建立/已刪除、有遺漏的複本、變更了主要複本位置或變更了複本數目。 若為 FailoverUnit 符合下列條件的其他情況下，平衡執行則不會中斷︰有額外的複本、變更了任何複本旗標、只變更了資料分割版本或任何其他情況。 |
|GlobalMovementThrottleThresholdPercentage|雙精確度，預設值為 0|動態|在 GlobalMovementThrottleCountingInterval 所指出的過去間隔中，平衡和放置階段所允許的總移動數目上限 (以叢集中的複本總數百分比來表示)。 0 表示沒有限制。 如果您同時指定此設定和 GlobalMovementThrottleThreshold，則系統會使用更保守的限制。|
|GlobalMovementThrottleThresholdPercentageForBalancing|雙精確度，預設值為 0|動態|在 GlobalMovementThrottleCountingInterval 所指出的過去間隔中，平衡階段所允許的移動數目上限 (以 PLB 中的複本總數百分比來表示)。 0 表示沒有限制。 如果您同時指定此設定和 GlobalMovementThrottleThresholdForBalancing，則系統會使用更保守的限制。|
|AutoDetectAvailableResources|布林值，預設值為 TRUE|靜態|此組態會觸發自動偵測節點上可用資源 (CPU 和記憶體) 的作業。當此組態設定為 true 時，我們會讀取實際的容量，並在使用者指定的節點容量錯誤或完全未定義容量時加以更正。如果此組態設定為 false，我們會追蹤使用者指定的節點容量錯誤的警告，但不會加以更正，這表示使用者想要將容量指定為大於節點實際擁有的容量，如果並未定義容量，則會採用無限容量 |

### <a name="section-name-hosting"></a>區段名稱：Hosting
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| ServiceTypeRegistrationTimeout |時間 (秒)，預設值為 300 |動態|允許 ServiceType 向網狀架構註冊的最大時間 |
| ServiceTypeDisableFailureThreshold |整數，預設值為 1 |動態|這是失敗次數的閾值，超過此值之後，就會通知 FailoverManager (FM) 停用該節點上的服務類型，並嘗試放置在另一個節點。 |
| ActivationRetryBackoffInterval |時間 (秒)，預設值為 5 |動態|每次啟用失敗的輪詢間隔；在每次連續啟用失敗之後，系統會重試啟用最多 MaxActivationFailureCount 次。 每次嘗試的重試間隔是連續啟用失敗與啟用輪詢間隔的乘積。 |
| ActivationMaxRetryInterval |時間 (秒)，預設值為 300 |動態|在每次連續啟用失敗時，系統會重試啟用最多 ActivationMaxFailureCount 次。 ActivationMaxRetryInterval 指定每次啟用失敗之後在重試之前等待的時間間隔 |
| ActivationMaxFailureCount |整數，預設值為 10 |動態|系統在放棄之前重試失敗啟用的次數 |
|ActivationTimeout| 時間範圍，預設值為 Common::TimeSpan::FromSeconds(180)|動態| 以秒為單位指定時間範圍。 應用程式的啟用、停用和升級逾時。 |
|ApplicationHostCloseTimeout| 時間範圍，預設值為 Common::TimeSpan::FromSeconds(120)|動態| 以秒為單位指定時間範圍。 在自我啟動的程序中偵測到網狀架構結束時，FabricRuntime 會關閉使用者主機 (applicationhost) 處理序中的所有複本。 這是關閉作業的逾時值。 |
|ApplicationUpgradeTimeout| 時間範圍，預設值為 Common::TimeSpan::FromSeconds(360)|動態| 以秒為單位指定時間範圍。 應用程式升級的逾時。 如果逾時值小於「ActivationTimeout」，部署便會失敗。 |
|CreateFabricRuntimeTimeout|時間範圍，預設值為 Common::TimeSpan::FromSeconds(120)|動態| 以秒為單位指定時間範圍。 同步 FabricCreateRuntime 呼叫的逾時值 |
|DeploymentMaxFailureCount|整數，預設值為 20| 動態|應用程式部署會先重試 DeploymentMaxFailureCount 次數，才讓節點上該應用程式的部署失敗。| 
|DeploymentMaxRetryInterval| 時間範圍，預設值為 Common::TimeSpan::FromSeconds(3600)|動態| 以秒為單位指定時間範圍。 部署的重試間隔上限。 在每次連續失敗之後，重試間隔的計算方式為 Min( DeploymentMaxRetryInterval; Continuous Failure Count * DeploymentRetryBackoffInterval) |
|DeploymentRetryBackoffInterval| 時間範圍，預設值為 Common::TimeSpan::FromSeconds(10)|動態|以秒為單位指定時間範圍。 部署失敗的輪詢間隔。 在每個連續的部署失敗發生時，系統最多會將部署重試 MaxDeploymentFailureCount 次。 重試間隔是連續部署失敗和部署輪詢間隔的乘積。 |
|EnableActivateNoWindow| 布林值，預設值為 FALSE|動態| 已啟動的程序會建立在沒有任何主控台的背景中。 |
|EnableProcessDebugging|布林值，預設值為 FALSE|動態| 能夠在偵錯工具下啟動應用程式主機 |
|EndpointProviderEnabled| 布林值，預設值為 FALSE|靜態| 能夠由網狀架構管理端點資源。 必須在 FabricNode 中指定開始和結束的應用程式連接埠範圍。 |
|FabricContainerAppsEnabled| 布林值，預設值為 FALSE|靜態| |
|FirewallPolicyEnabled|布林值，預設值為 FALSE|靜態| 能夠為端點資源開啟防火牆連接埠，並於 ServiceManifest 中明確指定連接埠 |
|GetCodePackageActivationContextTimeout|時間範圍，預設值為 Common::TimeSpan::FromSeconds(120)|動態|以秒為單位指定時間範圍。 CodePackageActivationContext 呼叫的逾時值。 這不適用於特定服務。 |
|IPProviderEnabled|布林值，預設值為 FALSE|靜態|能夠管理 IP 位址。 |
|NTLMAuthenticationEnabled|布林值，預設值為 FALSE|靜態| 能夠支援執行身分為其他使用者的程式碼套件使用 NTLM，以便機器之間的處理序可以安全地通訊。 |
|NTLMAuthenticationPasswordSecret|SecureString，預設值為 Common::SecureString(L"")|靜態|是加密的，用來產生 NTLM 使用者的密碼。 如果 NTLMAuthenticationEnabled 為 true，則必須加以設定。 由部署人員驗證。 |
|NTLMSecurityUsersByX509CommonNamesRefreshInterval|時間範圍，預設值為 Common::TimeSpan::FromMinutes(3)|動態|以秒為單位指定時間範圍。 環境特有的設定。FileStoreService NTLM 設定所要使用的新憑證定期主控掃描間隔。 |
|NTLMSecurityUsersByX509CommonNamesRefreshTimeout|時間範圍，預設值為 Common::TimeSpan::FromMinutes(4)|動態| 以秒為單位指定時間範圍。 使用憑證通用名稱來設定 NTLM 使用者的逾時。 FileStoreService 共用需要 NTLM 使用者。 |
|RegisterCodePackageHostTimeout|時間範圍，預設值為 Common::TimeSpan::FromSeconds(120)|動態| 以秒為單位指定時間範圍。 FabricRegisterCodePackageHost 同步呼叫的逾時值。 這僅適用於多程式碼套件應用程式主機，例如 FWP |
|RequestTimeout|時間範圍，預設值為 Common::TimeSpan::FromSeconds(30)|動態| 以秒為單位指定時間範圍。 這代表使用者的應用程式主機和網狀架構的各種主控相關作業 (例如工廠註冊、執行階段註冊) 處理序之間的通訊逾時值。 |
|RunAsPolicyEnabled| 布林值，預設值為 FALSE|靜態| 能夠以本機使用者身分 (而非用來執行網狀架構處理序的使用者身分) 執行程式碼套件。 若要啟用此原則，網狀架構必須以 SYSTEM 身分或具有 SeAssignPrimaryTokenPrivilege 之使用者的身分來執行。 |
|ServiceFactoryRegistrationTimeout| 時間範圍，預設值為 Common::TimeSpan::FromSeconds(120)|動態|以秒為單位指定時間範圍。 同步暫存器 (無狀態/具狀態) ServiceFactory 呼叫的逾時值 |
|ServiceTypeDisableGraceInterval|時間範圍，預設值為 Common::TimeSpan::FromSeconds(30)|動態|以秒為單位指定時間範圍。 時間間隔，之後便可以停用服務類型 |

### <a name="section-name-federation"></a>區段名稱︰Federation
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| LeaseDuration |時間 (秒)，預設值為 30 |動態|節點與其相鄰節點之間的租用持續時間。 |
| LeaseDurationAcrossFaultDomain |時間 (秒)，預設值為 30 |動態|所有容錯網域的節點與其相鄰節點之間的租用持續時間。 |

### <a name="section-name-clustermanager"></a>區段名稱︰ClusterManager
| **參數** | **允許的值** | **升級原則** | **指引或簡短描述** |
| --- | --- | --- | --- |
| UpgradeStatusPollInterval |時間 (秒)，預設值為 60 |動態|輪詢應用程式升級狀態的頻率。 此值決定任何 GetApplicationUpgradeProgress 呼叫的更新速率 |
| UpgradeHealthCheckInterval |時間 (秒)，預設值為 60 |動態|受監視應用程式升級期間的健康狀態檢查頻率 |
| FabricUpgradeStatusPollInterval |時間 (秒)，預設值為 60 |動態|輪詢網狀架構升級狀態的頻率。 此值決定任何 GetFabricUpgradeProgress 呼叫的更新速率 |
| FabricUpgradeHealthCheckInterval |時間 (秒)，預設值為 60 |動態|受監視網狀架構升級期間的健康狀態檢查頻率 |
|InfrastructureTaskProcessingInterval | 時間 (秒)，預設值為 10 |動態|以秒為單位指定時間範圍。 處理狀態機器的基礎結構工作所使用的處理間隔。 |
|TargetReplicaSetSize |整數，預設值為 7 |不允許|ClusterManager 的 TargetReplicaSetSize。 |
|MinReplicaSetSize |整數，預設值為 3 |不允許|ClusterManager 的 MinReplicaSetSize。 |
|ReplicaRestartWaitDuration |時間 (秒)，預設值為 (60.0 * 30)|不允許|以秒為單位指定時間範圍。 ClusterManager 的 ReplicaRestartWaitDuration。 |
|QuorumLossWaitDuration |時間 (秒)，預設值為 MaxValue |不允許| 以秒為單位指定時間範圍。 ClusterManager 的 QuorumLossWaitDuration。 |
|StandByReplicaKeepDuration | 時間 (秒)，預設值為 (3600.0 * 2)|不允許|以秒為單位指定時間範圍。 ClusterManager 的 StandByReplicaKeepDuration。 |
|PlacementConstraints | 字串，預設值為 "" |不允許|ClusterManager 的 PlacementConstraints。 |
|SkipRollbackUpdateDefaultService | 布林值，預設值為 false |動態|CM 會在應用程式升級復原期間，略過已更新之預設服務的還原作業。 |
|EnableDefaultServicesUpgrade | 布林值，預設值為 false |動態|在應用程式升級期間啟用預設服務升級作業。 在升級之後，將會覆寫預設的服務描述。 |
|InfrastructureTaskHealthCheckWaitDuration |時間 (秒)，預設值為 0|動態| 以秒為單位指定時間範圍。 在後置處理基礎結構工作之後，要開始健康狀態檢查之前所等待的時間量。 |
|InfrastructureTaskHealthCheckStableDuration | 時間 (秒)，預設值為 0|動態| 以秒為單位指定時間範圍。 在讓基礎結構工作的後置處理成功完成前，所要觀察到連續通過健康狀態檢查的時間量。 若觀察到失敗的健康狀態檢查將會重設此計時器。 |
|InfrastructureTaskHealthCheckRetryTimeout | 時間 (秒)，預設值為 60 |動態|以秒為單位指定時間範圍。 在後置處理基礎結構工作時，要花在重試失敗健康狀態檢查的時間量。 若觀察到已通過的健康狀態檢查將會重設此計時器。 |
|ImageBuilderTimeoutBuffer |時間 (秒)，預設值為 3 |動態|以秒為單位指定時間範圍。 要允許 Image Builder 的特定逾時錯誤傳回給用戶端的時間量。 若此緩衝區過小，用戶端便會在伺服器之前逾時，並收到一般逾時錯誤。 |
|MinOperationTimeout | 時間 (秒)，預設值為 60 |動態|以秒為單位指定時間範圍。 在 ClusterManager 上內部處理作業的全域逾時下限。 |
|MaxOperationTimeout |時間 (秒)，預設值為 MaxValue |動態| 以秒為單位指定時間範圍。 在 ClusterManager 上內部處理作業的全域逾時上限。 |
|MaxTimeoutRetryBuffer | 時間 (秒)，預設值為 600 |動態|以秒為單位指定時間範圍。 由於逾時而在內部重試時的作業逾時上限為 <Original Time out> + <MaxTimeoutRetryBuffer>。 會以 MinOperationTimeout 增量來加上額外的逾時。 |
|MaxCommunicationTimeout |時間 (秒)，預設值為 600 |動態|以秒為單位指定時間範圍。 ClusterManager 和其他系統服務 (也就是，命名服務、容錯移轉管理員等) 之間的內部通訊逾時上限。 此逾時值應小於全域 MaxOperationTimeout (因為每個用戶端作業的系統元件之間可能會有多個通訊)。 |
|MaxDataMigrationTimeout |時間 (秒)，預設值為 600 |動態|以秒為單位指定時間範圍。 在網狀架構升級之後，資料移轉復原作業的逾時上限。 |
|MaxOperationRetryDelay |時間 (秒)，預設值為 5|動態| 以秒為單位指定時間範圍。 發生失敗時的內部重試延遲上限。 |
|ReplicaSetCheckTimeoutRollbackOverride |時間 (秒)，預設值為 1200 |動態| 以秒為單位指定時間範圍。 如果 ReplicaSetCheckTimeout 設為 DWORD 的最大值，則會以此組態的值加以覆寫以便進行復原。 向前復原所使用的值絕不會遭到覆寫。 |
|ImageBuilderJobQueueThrottle |整數，預設值為 10 |動態|Image Builder Proxy 作業佇列對於應用程式要求的執行緒計數節流。 |
|MaxExponentialOperationRetryDelay|時間範圍，預設值為 Common::TimeSpan::FromSeconds(30)|動態|以秒為單位指定時間範圍。 重複發生失敗時的內部重試指數延遲上限 |

### <a name="section-name-defragmentationemptynodedistributionpolicy"></a>區段名稱：DefragmentationEmptyNodeDistributionPolicy
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|KeyIntegerValueMap，預設值為 None|動態|指定重組在清空節點時所遵循的原則。 在給定計量時，0 表示 SF 應該嘗試在 UD 和 FD 之間平均地重組節點，1 僅表示必須重組節點 |

### <a name="section-name-defragmentationmetrics"></a>區段名稱：DefragmentationMetrics
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|KeyBoolValueMap，預設值為 None|動態|決定應用於重組而非用於負載平衡的一組計量。 |

### <a name="section-name-defragmentationmetricspercentornumberofemptynodestriggeringthreshold"></a>區段名稱：DefragmentationMetricsPercentOrNumberOfEmptyNodesTriggeringThreshold
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap，預設值為 None|動態|藉由指定一個範圍的百分比 [0.0-1.0) 或將空節點的數目指定為 >= 1.0 的數字，來決定要將叢集視為已重組所需具有的可用節點數目 |

### <a name="section-name-dnsservice"></a>區段名稱：DnsService
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|IsEnabled|布林值，預設值為 FALSE|靜態| |
|InstanceCount|整數，預設值為 -1|靜態|  |

### <a name="section-name-metricactivitythresholds"></a>區段名稱：MetricActivityThresholds
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|KeyIntegerValueMap，預設值為 None|動態|決定叢集計量的一組 MetricActivityThresholds。 如果 maxNodeLoad 大於 MetricActivityThresholds，則會執行平衡作業。 對於重組計量，此參數會定義負載數量，一旦等於或低於此數量，Service Fabric 便會將節點視為空白 |

### <a name="section-name-metricbalancingthresholds"></a>區段名稱：MetricBalancingThresholds
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap，預設值為 None|動態|決定叢集計量的一組 MetricBalancingThresholds。 如果 maxNodeLoad/minNodeLoad 大於 MetricBalancingThresholds，則會執行平衡作業。 如果至少一個 FD 或 UD 中的 maxNodeLoad/minNodeLoad 小於 MetricBalancingThresholds，則會執行重組作業。 |

### <a name="section-name-nodebufferpercentage"></a>區段名稱：NodeBufferPercentage
| **參數** | **允許的值** |**升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap，預設值為 None|動態|每一計量名稱的節點容量百分比，作為緩衝區以便在容錯移轉案例中，於節點上保留一些可用位置。 |

### <a name="section-name-replication"></a>區段名稱：Replication
| **參數** | **允許的值** | **升級原則**| **指引或簡短描述** |
| --- | --- | --- | --- |
|MaxCopyQueueSize|單位，預設值為 1024|靜態|這是用來定義保有複寫作業之佇列初始大小的最大值。  請注意，此值必須是 2 的乘冪。  如果在執行階段期間，佇列增加到此大小，則會將主要和次要複寫器之間的作業節流。|
|BatchAcknowledgementInterval|時間範圍，預設值為 Common::TimeSpan::FromMilliseconds(15)|靜態|以秒為單位指定時間範圍。 決定複寫器在收到作業後，要等待多久才回傳通知。 在此期間所收到的其他作業會透過同一則訊息回傳其通知，以降低網路流量，但可能也會降低複寫器的輸送量。|
|MaxReplicationMessageSize|單位，預設值為 52428800|靜態|複寫作業的訊息大小上限。 預設值為 50 MB。|
|ReplicatorAddress|字串，預設值為 L"localhost:0"|靜態|-'IP:Port' 字串形式的端點，Windows Fabric 複寫器使用此端點來建立與其他複本的連線，以便傳送/接收作業。|
|ReplicatorListenAddress|字串，預設值為 L"localhost:0"|靜態|'IP:Port' 字串形式的端點，Windows Fabric 複寫器會使用此端點接收來自其他複本的作業。|
|ReplicatorPublishAddress|字串，預設值為 L"localhost:0"|靜態|'IP:Port' 字串形式的端點，Windows Fabric 複寫器會使用此端點將作業傳送至其他複本。|
|MaxPrimaryReplicationQueueSize|單位，預設值為 1024|靜態|這是主要複寫佇列中可存在的作業數目上限。 請注意，此值必須是 2 的乘冪。|
|MaxPrimaryReplicationQueueMemorySize|單位，預設值為 0|靜態|這是主要複寫佇列的最大值 (位元組)。|
|MaxSecondaryReplicationQueueSize|單位，預設值為 2048|靜態|這是次要複寫佇列中可存在的作業數目上限。 請注意，此值必須是 2 的乘冪。|
|MaxSecondaryReplicationQueueMemorySize|單位，預設值為 0|靜態|這是次要複寫佇列的最大值 (位元組)。|
|QueueHealthMonitoringInterval|時間範圍，預設值為 Common::TimeSpan::FromSeconds(30)|靜態|以秒為單位指定時間範圍。 此值會決定複寫器用來監視複寫作業佇列中所發生之任何警告/錯誤健康情況事件的時間週期。 值為 '0' 會停用健康情況監視 |
|QueueHealthWarningAtUsagePercent|單位，預設值為 80|靜態|此值會決定複寫佇列使用量 (以百分比表示)，一旦超過此使用量，我們便會發出有關佇列使用量偏高的警告。 我們會在 QueueHealthMonitoringInterval 寬限間隔過後這麼做。 如果佇列使用量在寬限期間內低於這個百分比|
|RetryInterval|時間範圍，預設值為 Common::TimeSpan::FromSeconds(5)|靜態|以秒為單位指定時間範圍。 當作業遺失或遭到拒絕，此計時器會決定複寫器重新試著傳送作業的頻率。|

### <a name="section-name-transport"></a>區段名稱：Transport
| **參數** | **允許的值** |**升級原則** |**指引或簡短描述** |
| --- | --- | --- | --- |
|ResolveOption|字串，預設值為 L"unspecified"|靜態|決定 FQDN 的解析方式。  有效值為 "unspecified/ipv4/ipv6"。 |
|ConnectionOpenTimeout|時間範圍，預設值為 Common::TimeSpan::FromSeconds(60)|靜態|以秒為單位指定時間範圍。 內送和接收端 (包括安全模式下的安全性交涉) 的連線設定逾時 |

## <a name="next-steps"></a>後續步驟
如需有關叢集管理的詳細資訊，請參閱下列文件︰

[新增、變換、移除 Azure 叢集的憑證 ](service-fabric-cluster-security-update-certs-azure.md) 

