---
title: "使用 Azure Data Factory 從 Salesforce 複製資料 | Microsoft Docs"
description: "了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 Salesforce 複製到支援的接收資料存放區。"
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: spelluru
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2017
ms.author: jingwang
ms.openlocfilehash: c819b3e3e715427632e793ce77d31554914d248e
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
---
# <a name="copy-data-from-salesforce-using-azure-data-factory"></a>使用 Azure Data Factory 從 Salesforce 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版 - 正式推出](v1/data-factory-salesforce-connector.md)
> * [第 2 版 - 預覽](connector-salesforce.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Salesforce 資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

> [!NOTE]
> 本文適用於第 2 版的 Data Fatory (目前為預覽版)。 如果您使用第 1 版的 Data Factory 服務 (也就是正式推出版 (GA))，請參閱 [V1 中的 Salesforce 連接器](v1/data-factory-salesforce-connector.md)。

## <a name="supported-scenarios"></a>支援的案例

您可以將資料從 Salesforce 資料庫複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Salesforce 連接器支援下列 Salesforce 版本：**Developer Edition、Professional Edition、Enterprise Edition 或 Unlimited Edition**。 並且支援從 Salesforce **生產環境、沙箱及自訂網域**複製資料。

## <a name="prerequisites"></a>必要條件

* 必須在 Salesforce 中啟用 API 權限。 請參閱 [如何在 Salesforce 中透過權限集啟用 API 存取權？](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/)

## <a name="salesforce-request-limits"></a>Salesforce 要求限制

Salesforce 對於 API 要求總數和並行 API 要求均有限制。 請注意下列幾點：

- 如果並行要求數目超過限制，系統就會進行節流，您將會看到隨機失敗。
- 如果要求總數超過限制，將會封鎖 Salesforce 帳戶 24 小時。

在上述兩種情況中，您也可能會收到「REQUEST_LIMIT_EXCEEDED」錯誤。 如需詳細資訊，請參閱 [Salesforce 開發人員限制](http://resources.docs.salesforce.com/200/20/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf)文章的＜API 要求限制＞一節。

## <a name="getting-started"></a>開始使用
您可以使用 .NET SDK、Python SDK、Azure PowerShell、REST API 或 Azure Resource Manager 範本來建立具有複製活動的管線。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](quickstart-create-data-factory-dot-net.md)。

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Salesforce 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Salesforce 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| 類型 |類型屬性必須設為： **Salesforce**。 |是 |
| environmentUrl | 指定 Salesforce 執行個體的 URL。 <br><br> - 預設為 `"https://login.salesforce.com"`. <br> - 若要從沙箱複製資料，請指定 `"https://test.salesforce.com"`。 <br> - 若要從自訂網域複製資料，舉例來說，請指定 `"https://[domain].my.salesforce.com"`。 |否 |
| username |指定使用者帳戶的使用者名稱。 |是 |
| password |指定使用者帳戶的密碼。 |是 |
| securityToken |指定使用者帳戶的安全性權杖。 如需如何重設/取得安全性權杖的指示，請參閱 [取得安全性權杖](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) 。 若要整體了解安全性權杖，請參閱[安全性和 API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)。 |是 |

**範例：**

```json
{
    "properties": {
        "type": "Salesforce",
        "typeProperties": {
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            },
            "securityToken": {
                "type": "SecureString",
                "value": "<security token>"
            }
        }
    },
    "name": "SalesforceLinkedService"
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Salesforce 資料集所支援的屬性清單。

若要從 Salesforce 複製資料，請將資料集的類型屬性設定為 **RelationalTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| 類型 | 資料集的類型屬性必須設定為：**RelationalTable** | 是 |
| tableName | Salesforce 資料庫中的資料表名稱。 | 否 (如果已指定活動來源中的「查詢」) |

> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/copy-data-from-salesforce/data-factory-salesforce-api-name.png)

**範例：**

```json
{
    "name": "SalesforceDataset",
    "properties":
    {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Salesforce linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "MyTable__c"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Salesforce 來源所支援的屬性清單。

### <a name="salesforce-as-source"></a>Salesforce 作為來源

若要從 Salesforce 複製資料，請將複製活動中的來源類型設定為 **RelationalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| 類型 | 複製活動來源的類型屬性必須設定為：**RelationalSource** | 是 |
| query |使用自訂查詢來讀取資料。 您可以使用 SQL-92 查詢或 [Salesforce 物件查詢語言 (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm) 查詢。 例如： `select * from MyTable__c`。 | 否 (如果已指定資料集中的 "tableName") |

> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/copy-data-from-salesforce/data-factory-salesforce-api-name-2.png)

**範例：**

```json
"activities":[
    {
        "name": "CopyFromSalesforce",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Salesforce input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "RelationalSource",
                "query": "SELECT Col_Currency__c, Col_Date__c, Col_Email__c FROM AllDataType__c"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="query-tips"></a>查詢秘訣

### <a name="retrieving-data-from-salesforce-report"></a>從 Salesforce 報表擷取資料

您可以藉由以 `{call "<report name>"}. Example: `"query": "{call \"TestReport\"}"` 方式指定查詢，從 Salesforce 報表擷取資料。

### <a name="retrieving-deleted-records-from-salesforce-recycle-bin"></a>從 Salesforce 資源回收筒擷取已刪除的記錄

若要從「Salesforce 資源回收筒」查詢虛刪除記錄，您可以在查詢中指定 **"IsDeleted = 1"**。 例如，

* 若只要查詢已刪除的記錄，請指定 "select * from MyTable__c **where IsDeleted= 1**"
* 若要查詢所有記錄 (包括現有和已刪除的記錄)，請指定 "select * from MyTable__c **where IsDeleted = 0 or IsDeleted = 1**"

### <a name="retrieving-data-using-where-clause-on-datetime-column"></a>在 DateTime 資料行上使用 Where 子句來擷取資料

指定 SOQL 或 SQL 查詢時，請注意 DateTime 格式差異。 例如：

* **SOQL 範例**：`$$Text.Format('SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= {0:yyyy-MM-ddTHH:mm:ssZ} AND LastModifiedDate < {1:yyyy-MM-ddTHH:mm:ssZ}', <datetime parameter>, <datetime parameter>)`
* **SQL 範例**`$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\\'{0:yyyy-MM-dd HH:mm:ss}\\'}} AND LastModifiedDate < {{ts\\'{1:yyyy-MM-dd HH:mm:ss}\\'}}', <datetime parameter>, <datetime parameter>)`

## <a name="data-type-mapping-for-salesforce"></a>Salesforce 的資料類型對應

從 Salesforce 複製資料時，會使用下列從 Salesforce 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| Salesforce 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| 自動編號 |String |
| 核取方塊 |Boolean |
| 貨幣 |兩倍 |
| 日期 |DateTime |
| 日期/時間 |DateTime |
| 電子郵件 |String |
| 識別碼 |String |
| 查閱關聯性 |String |
| 複選挑選清單 |String |
| Number |兩倍 |
| 百分比 |兩倍 |
| 電話 |String |
| 挑選清單 |String |
| 文字 |String |
| 文字區域 |String |
| 文字區域 (完整) |String |
| 文字區域 (豐富) |String |
| 文字 (加密) |String |
| URL |String |


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。