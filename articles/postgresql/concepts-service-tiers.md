---
title: "適用於 PostgreSQL 的 Azure 資料庫中的定價層"
description: "適用於 PostgreSQL 的 Azure 資料庫中的定價層"
services: postgresql
author: kamathsun
ms.author: sukamat
manager: jhubbard
editor: jasonwhowell
ms.custom: mvc
ms.service: postgresql
ms.topic: article
ms.date: 05/31/2017
ms.openlocfilehash: 0ebdced6ac748245faed90949fd0e76c0eacb2d3
ms.sourcegitcommit: 9c3150e91cc3075141dc2955a01f47040d76048a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/26/2017
---
# <a name="azure-database-for-postgresql-options-and-performance-understand-whats-available-in-each-pricing-tier"></a>適用於 PostgreSQL 的 Azure 資料庫選項和效能：了解每個定價層中可用的項目
當您建立適用於 PostgreSQL 伺服器的 Azure 資料庫時，有三個主要選項可供您決定如何設定配置給該伺服器的資源。 這些選項會影響伺服器的效能和規模。
- 定價層 
- 計算單位
- 儲存體 (GB)

每個定價層有各種不同的效能等級 (計算單位)，您可以根據工作負載需求從中選擇。 較高的效能等級會為設計成提供較高輸送量的伺服器提供額外的資源。 您可以在定價層內變更伺服器的效能等級，而且幾乎不會產生任何應用程式停機時間。

> [!IMPORTANT]
> 當服務處於公開預覽狀態時，並沒有保證的服務等級協定 (SLA)。

在適用於 PostgreSQL 的 Azure 資料庫內，您可以擁有一個或多個資料庫。 您可以選擇在每個伺服器建立單一資料庫，以讓資料庫利用所有伺服器資源，或是建立多個資料庫來共用伺服器資源。 

## <a name="choose-a-pricing-tier"></a>選擇定價層
在預覽版中，適用於 PostgreSQL 的 Azure 資料庫提供兩個定價層︰基本和標準。 進階層目前尚無法使用，但即將推出。 

下表提供最適用於不同應用程式工作負載的定價層範例。

| 定價層  | 目標工作負載 |
| :----------- | :----------------|
| 基本 | 最適合需要可調整計算和儲存體而不需 IOPS 保證的小型工作負載。 範例包括用於開發或測試的伺服器，或者不常使用的小規模應用程式。 |
| 標準 | 雲端應用程式的這個進階選項需要高輸送量的 IOPS 保證。 範例包括 Web 或分析應用程式。 |
| 進階 | 最適合需要為交易和 IO 提供低延遲的工作負載。 可為許多並行使用者提供最佳支援。 適用於支援任務關鍵性應用程式的資料庫。<br />預覽版中並未提供進階定價層。 |

若要決定定價層，請先從判斷您的工作負載是否需要 IOPS 保證開始。 如果需要，請使用標準定價層。

| **定價層功能** | **基本** | **標準** |
| :------------------------ | :-------- | :----------- |
| 計算單位數目上限 | 100 | 800 | 
| 總儲存體上限 | 1 TB | 1 TB | 
| 儲存體 IOPS 保證 | N/A | 是 | 
| 儲存體 IOPS 上限 | N/A | 3,000 | 
| 資料庫備份的保留期限 | 7 天 | 35 天 | 

在預覽時間範圍內，一旦建立伺服器，就無法變更定價層。 未來能將伺服器從某個定價層升級或降級至另一個定價層。

## <a name="understand-the-price"></a>了解價格
在 [Azure 入口網站](https://portal.azure.com/#create/Microsoft.PostgreSQLServer)內部建立適用於 PostgreSQL 的新 Azure 資料庫時，請按一下 [定價層] 刀鋒視窗，便會根據您選取的選項顯示每月成本。 如果您沒有 Azure 訂用帳戶，請使用 Azure 價格計算機取得估計的價格。 請瀏覽 [Azure 價格計算機](https://azure.microsoft.com/pricing/calculator/)網站，然後按一下 [新增項目]，展開 [資料庫] 類別，並選擇 [適用於 PostgreSQL 的 Azure 資料庫] 以自訂選項。

## <a name="choose-a-performance-level-compute-units"></a>選擇效能等級 (計算單位)
在您決定適用於 PostgreSQL 伺服器的 Azure 資料庫的定價層之後，您就可以選取所需的計算單位數來決定效能等級。 對於其 Web 或分析工作負載需要更高使用者並行處理的應用程式而言，200 或 400 個計算單位會是很好的起點。 

計算單位是 CPU 處理輸送量的量值，保證可供單一適用於 PostgreSQL 的 Azure 資料庫伺服器使用。 計算單位是 CPU 和記憶體資源的混合量值。  如需詳細資訊，請參閱[說明計算單位](concepts-compute-unit-and-storage.md)。

### <a name="basic-pricing-tier-performance-levels"></a>基本定價層效能等級：

| **效能等級** | **50** | **100** |
| :-------------------- | :----- | :------ |
| 計算單位數目上限 | 50 | 100 |
| 內含的儲存體大小 | 50 GB | 50 GB |
| 伺服器儲存體大小上限\* | 1 TB | 1 TB |

### <a name="standard-pricing-tier-performance-levels"></a>標準定價層效能等級：

| **效能等級** | **100** | **200** | **400** | **800** |
| :-------------------- | :------ | :------ | :------ | :------ |
| 計算單位數目上限 | 100 | 200 | 400 | 800 |
| 內含的儲存體大小和已佈建的 IOPS | 125 GB、<br/> 375 IOPS | 125 GB、<br/> 375 IOPS | 125 GB、<br/> 375 IOPS | 125 GB、<br/> 375 IOPS |
| 伺服器儲存體大小上限\* | 1 TB | 1 TB | 1 TB | 1 TB |
| 伺服器佈建的 IOPS 上限 | 3,000 IOPS | 3,000 IOPS | 3,000 IOPS | 3,000 IOPS |
| 每個 GB 中伺服器佈建的 IOPS 上限 | 每個 GB 中固定 3 IOPS | 每個 GB 中固定 3 IOPS | 每個 GB 中固定 3 IOPS | 每個 GB 中固定 3 IOPS |

\* 伺服器儲存體大小上限是指針對您伺服器佈建的儲存體大小上限。

## <a name="storage"></a>儲存體 
儲存體設定會定義適用於 PostgreSQL 伺服器的 Azure 資料庫可用的儲存體容量。 此服務所使用的儲存體包括資料庫檔案、交易記錄和 PostgreSQL 伺服器記錄。 選取儲存體設定時，請考慮裝載資料庫和效能需求 (IOPS) 所需的儲存體大小。

每個定價層至少會內含一些儲存體容量，也就是上表中註明的「內含的儲存體大小」。 您可以在建立伺服器時，以 125GB 為增量來增加額外的儲存體容量，最高可達允許的儲存體上限。 您可以在計算單位設定之外，獨立設定此額外的儲存體容量。 價格會根據選取的儲存體數量而變更。

每個效能等級中的 IOPS 設定與定價層和選擇的儲存體大小相關。 基本層不提供 IOPS 保證。 在標準定價層內，IOPS 會以固定 3:1 比例，按比例調整至儲存體大小上限。 125 GB 的內含儲存體可為 375 已佈建的 IOPS 提供保證，每個的 IO 大小可高達 256 KB。 您最多可以選擇 1TB 的額外儲存體，以保證有 3,000 個已佈建的 IOPS。

在 Azure 入口網站中監視計量圖表，或是撰寫 Azure CLI 命令來測量儲存體和 IOPS 的使用情況。 要監視的相關計量包括儲存體限制、儲存體百分比、已使用的儲存體和 IO 百分比。

>[!IMPORTANT]
> 在預覽版中，請在建立伺服器時選擇儲存體數量。 目前尚未支援變更現有伺服器上的儲存體大小。 

## <a name="scaling-a-server-up-or-down"></a>相應增加或減少伺服器
您會在建立適用於 PostgreSQL 的 Azure 資料庫的一開始，選擇定價層和效能等級。 稍後，您可以在相同的定價層範圍內，動態相應增加或減少計算單位。 在 Azure 入口網站中，滑動伺服器 [定價層] 刀鋒視窗上的 [計算單位]，或遵循下列範例進行指令碼處理：[使用 Azure CLI 監視和調整單一 PostgreSQL 伺服器](scripts/sample-scale-server-up-or-down.md)

計算單位的調整會與所選擇的儲存體大小上限分開進行。

變更資料庫的效能等級會在幕後於新的效能等級建立原始資料庫的複本，然後將連線切換到複本。 此程序期間不會遺失任何資料。 在我們切換到複本的短暫期間，資料庫的連線會停用，因此執行中的某些交易可能會回復。 這個時間範圍並不固定，但平均在 4 秒以內，而且有超過 99% 的案例都是在 30 秒以內。 如果在連接停用時有大型交易執行中，則此時間範圍可能會更長。

整個調整程序的期間取決於伺服器變更前後的大小和定價層。 例如，在標準定價層內變更計算單位的伺服器應在幾分鐘內完成。 完成變更之前，不會將新屬性套用至伺服器。

## <a name="next-steps"></a>後續步驟
- 如需計算單位的詳細資訊，請參閱[說明計算單位](concepts-compute-unit-and-storage.md)
- 了解如何[使用 Azure CLI 監視和調整單一 PostgreSQL 伺服器](scripts/sample-scale-server-up-or-down.md)
