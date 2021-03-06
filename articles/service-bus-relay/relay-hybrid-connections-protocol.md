---
title: "Azure 轉送混合式連線通訊協定指南 | Microsoft Docs"
description: "Azure 轉送混合式連線通訊協定指南。"
services: service-bus-relay
documentationcenter: na
author: clemensv
manager: timlt
editor: 
ms.assetid: 149f980c-3702-4805-8069-5321275bc3e8
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2017
ms.author: sethm
ms.openlocfilehash: 9d015678dbd99b8d978c2c8200b36bf51cac8893
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
---
# Azure 轉送混合式連線通訊協定
Azure 轉送是 Azure 服務匯流排平台的重要功能支柱。 轉送的新「混合式連線」功能是以 HTTP 和 Websocket 為基礎的安全、開放式通訊協定演化。 它會取代其前身，也就是建置在專屬通訊協定基礎上、名為「BizTalk 服務」的功能。 整合到 Azure 應用程式服務的混合式連線會繼續如往常般運作。

「混合式連線」可在網路上的兩個應用程式之間，建立雙向的二進位串流通訊，藉此讓其中一方或雙方都能位在 NAT 或防火牆之後。 本文說明用戶端為了與擔任接聽程式和傳送者角色的用戶端連線而與混合式連線轉送進行的互動，以及接聽程式如何接受新的連線。

## 互動模型
混合式連線轉送會藉由在雙方均可探索到，並可由自己的網路觀點連線到的 Azure 雲端提供會合點，來讓雙方連線。 在本文件和其他文件、在 API 以及在 Azure 入口網站中，該會合點稱為「混合式連線」。 本文其餘地方將混合式連線服務端點稱為「服務」。 互動模型仰賴其他許多網路 API 所建立的專門用語。

有接聽程式會先指出已準備好可以處理連入連線，接著在連線抵達時予以接受。 而在另一端，則有連線用戶端會連線到接聽程式，並預期該連線會被接受，以便能夠建立雙向通訊路徑。
您可在大部分通訊端 API 中發現「連線」、「接聽」、「接受」等同義詞彙。

任何轉送的通訊模型皆會讓任一方對服務端點建立輸出連線，在口語上，這會讓「接聽程式」也變成「用戶端」，並可能造成術語承載其他意義。 因此，我們對混合式連線所下的精確術語如下︰

連線兩端的程式稱為「用戶端」，因為兩者皆為服務的用戶端。 等候並接受連線的用戶端為「接聽程式」，或稱為擔任「接聽程式角色」。 透過服務對接聽程式起始新連線的用戶端則稱為「傳送者」或擔任「傳送者角色」。

### 接聽程式互動
接聽程式會與服務進行四種互動；本文稍後的參考資料章節會詳述所有連線細節。

#### 接聽
為了讓服務知道接聽程式已準備好接受連線，它會建立輸出 WebSocket 連線。 連線交握會攜帶轉送命名空間中所設定的混合式連線名稱，以及對該名稱授予「接聽」權限的安全性權杖。
服務接受 WebSocket 時，註冊便已完成，所建立的 Web WebSocket 會保持運作以作為用來啟用所有後續互動的「控制通道」。 在混合式連線中，服務最多允許 25 個並行接聽程式。 如果有 2 個以上的作用中接聽程式，則會將連入連線隨機平衡分配給這些接聽程式；但不保證會公平分配。

#### Accept
當傳送者在服務上開啟新的連線時，服務就會選擇並通知混合式連線中的其中一個作用中接聽程式。 此通知會以 JSON 訊息的形式，透過開啟的控制通道傳送給接聽程式，在該訊息中，則含有接聽程式為了接受連線所必須連線到之 WebSocket 端點的 URL。

此 URL 可以且必須由接聽程式直接使用，不得再行加工。
編碼資訊的有效期間很短，基本上就是傳送者願意等待兩端建立連線的時間，但上限為 30 秒。 URL 只能用於進行一次成功的連線嘗試。 一旦建立了具有會合 URL 的 WebSocket 連線，此 WebSocket 上的所有後續活動就會在傳送者來回轉送，服務不必再介入或解譯。

#### 續訂
必須用來註冊接聽程式和維護控制通道的安全性權杖，可能會在接聽程式作用期間過期。 權杖過期不會影響進行中的連線，但會導致服務在權杖過期當下或隨後立即捨棄控制通道。 「更新」作業是一則 JSON 訊息，接聽程式傳送此訊息以取代與控制通道相關聯的權杖，讓控制通道可以延長維持運作時間。

#### Ping
如果控制通道長時間閒置，途中的媒介 (例如負載平衡器或 NAT) 可能會捨棄 TCP 連線。 「Ping」作業可避免該現象，其憑藉方法為在通道上傳送少量資料，提醒網路路由上的所有人此連線是為了讓通道保持運作，並同時測試接聽程式是否仍作用中。 如果 ping 失敗，則應將控制通道視為無法使用，接聽程式應重新連線。

### 傳送者互動
傳送者只會與服務進行一項互動：就是連線。

#### 連線
「連線」作業會在服務上開啟 WebSocket，提供混合式連線名稱和 (選擇性，但預設為必要) 會在查詢字串中授予「傳送」權限的安全性權杖。 服務接著會以先前所述方式和接聽程式互動，並讓接聽程式建立會與此 WebSocket 一起加入的會合連線。 WebSocket 獲得接受後，將會與已連線的接聽程式進行所有後續互動。

### 互動摘要
此互動模型的結果是傳送者用戶端會產生具有「乾淨」WebSocket 的交握，該通訊端連線到接聽程式，並且不需要再進行前序編碼或準備。 實際上，這可讓任何現有 WebSocket 用戶端實作輕易利用混合式連線服務，只需要對其 WebSocket 用戶端層提供正確建構的 URL。

接聽程式透過接受互動所取得的會合連線 WebSocket 也是乾淨的，您只需要另外進行點擷取動作即可遞交給任何現有 WebSocket 伺服器實作，此動作是為了區分其架構之區域網路接聽程式上的「接受」作業與混合式連線的遠端「接受」作業。

## 通訊協定參考資料

本節描述上述通訊協定互動的詳細資料。

所有 WebSocket 連線都是在連接埠 443 上進行以作為從 HTTPS 1.1 (經常遭到某些 WebSocket 架構或 API 擷取) 開始的升級。 此處的描述不偏袒任何實作，不會建議您採用特定架構。

### 接聽程式通訊協定
接聽程式通訊協定是由兩個連線舉動和三項訊息作業所組成。

#### 接聽程式控制通道連線
控制通道開啟時，會對下列位置建立 WebSocket 連線︰

```
wss://{namespace-address}/$hc/{path}?sb-hc-action=...[&sb-hc-id=...]&sb-hc-token=...
```

`namespace-address` 是裝載混合式連線之 Azure 轉送命名空間的完整網域名稱，其格式通常為 `{myname}.servicebus.windows.net`。

查詢字串參數選項如下。

| 參數 | 必要 | 說明 |
| --- | --- | --- |
| `sb-hc-action` |是 |接聽程式角色的參數必須是 **sb-hc-action=listen** |
| `{path}` |是 |用來註冊此接聽程式之預先設定混合式連線的 URL 編碼命名空間路徑。 此運算式會附加至固定的 `$hc/` 路徑部分。 |
| `sb-hc-token` |是\* |針對授予**接聽**權限的命名空間或混合式連線，接聽程式必須提供有效且以 URL 編碼的服務匯流排共用存取權杖。 |
| `sb-hc-id` |否 |用戶端提供的這個選擇性識別碼可讓您進行端對端診斷追蹤。 |

如果因為混合式連線路徑未註冊、權杖無效或遺失或是其他某些錯誤，導致 WebSocket 連線失敗，將使用一般的 HTTP 1.1 狀態回饋模型提供錯誤回饋。 狀態描述會包含錯誤追蹤識別碼，以供您告知 Azure 支援人員︰

| 代碼 | 錯誤 | 說明 |
| --- | --- | --- |
| 404 |找不到 |混合式連線路徑無效或基底 URL 的格式不正確。 |
| 401 |未經授權 |安全性權杖遺失、格式不正確或無效。 |
| 403 |禁止 |對於此動作來說，此路徑的安全性權杖無效。 |
| 500 |內部錯誤 |服務發生錯誤。 |

如果 WebSocket 連線在最初設定過後遭到服務刻意關閉，服務會使用適當的 WebSocket 通訊協定錯誤碼以及同時含有追蹤識別碼的錯誤描述訊息告知您這麼做的原因。 服務不會沒發生錯誤狀況就關閉控制通道。 至於正常關機則為用戶端所為。

| WS 狀態 | 說明 |
| --- | --- |
| 1001 |混合式連線路徑已遭到刪除或停用。 |
| 1008 |安全性權杖已過期，因此違反授權原則。 |
| 1011 |服務發生錯誤。 |

### 接受交握
服務會透過先前建立的控制通道，在 WebSocket 文字框中以 JSON 訊息的形式對接聽程式傳送「accept」通知。 此訊息沒有任何回覆。

訊息中包含名為「accept」的 JSON 物件，其在此階段會定義下列屬性︰

* **位址** – 用來對服務建立 WebSocket 以接受連入連線的 URL 字串。
* **識別碼** – 此連線的唯一識別碼。 如果識別碼為傳送者用戶端所提供，則為傳送者提供的值，否則會是系統產生的值。
* **connectHeaders** – 傳送者已提供給轉送端點的所有 HTTP 標頭，其中也包括 Sec-WebSocket-Protocol 和 Sec-WebSocket-Extensions 標頭。

#### 接受訊息

```json
{                                                           
    "accept" : {
        "address" : "wss://168.61.148.205:443/$hc/{path}?..."    
        "id" : "4cb542c3-047a-4d40-a19f-bdc66441e736",  
        "connectHeaders" : {                                         
            "Host" : "...",                                                
            "Sec-WebSocket-Protocol" : "...",                              
            "Sec-WebSocket-Extensions" : "..."                             
        }                                                            
     }                                                         
}
```

接聽程式會使用 JSON 訊息中提供的位址 URL 來建立 WebSocket，以接受或拒絕傳送者通訊端。

#### 接受通訊端
為了接受，接聽程式會對提供的位址建立 WebSocket 連線。

如果「accept」訊息執行 `Sec-WebSocket-Protocol` 標題，則接聽程式只接受支援該通訊協定的 WebSocket。 此外，會在建立 WebSocket 時設定標題。

對 `Sec-WebSocket-Extensions` 標題同樣適用。 如果架構支援擴充功能，其應將標題設定為擴充功能所需 `Sec-WebSocket-Extensions` 交握的伺服器端回覆。

URL 必須保持原樣以用來建立接受通訊端，但要包含下列參數︰

| 參數 | 必要 | 說明 |
| --- | --- | --- |
| `sb-hc-action` |是 |若要接受通訊端，參數必須是 `sb-hc-action=accept` |
| `{path}` |是 |(請參閱下列段落) |
| `sb-hc-id` |否 |請參閱先前的**識別碼**描述。 |

`{path}` 是用來註冊此接聽程式之預先設定混合式連線的 URL 編碼命名空間路徑。 此運算式會附加至固定的 `$hc/` 路徑部分。 

`path` 運算式可能會使用後置字元與接在分隔斜線後之已登錄名稱的查詢字串運算式進行擴充。 這可讓寄件者用戶端在不可能包括 HTTP 標題時，將分派引數傳遞至接受接聽程式。 預期為接聽程式架構會剖析固定的路徑部分和路徑中的已註冊名稱，並提供其餘部分，可能是不含前置詞為 `sb-` 的任何查詢字串引數，可供應用程式決定是否要接受連線。

如需詳細資訊，請參閱「寄件者通訊協定」一節。

如果發生錯誤，服務會有如下回覆︰

| 代碼 | 錯誤 | 說明 |
| --- | --- | --- |
| 403 |禁止 |URL 無效。 |
| 500 |內部錯誤 |服務發生錯誤 |

連線建立後，伺服器會在傳送者 WebSocket 關閉時，或是具有下列狀態時關閉 WebSocket：

| WS 狀態 | 說明 |
| --- | --- |
| 1001 |傳送者用戶端關閉連線。 |
| 1001 |混合式連線路徑已遭到刪除或停用。 |
| 1008 |安全性權杖已過期，因此違反授權原則。 |
| 1011 |服務發生錯誤。 |

#### 拒絕通訊端
在檢查「accept」訊息之後拒絕通訊端需要類似的交握，因此會對傳送者回傳可告知拒絕原因的狀態碼和狀態描述。

在這裡，通訊協定的設計選擇是使用 WebSocket 交握 (其設計是要以定義的錯誤狀態來結束)，以便讓接聽程式用戶端實作可以繼續依賴 WebSocket 用戶端，而不必另外運用裸機 HTTP 用戶端。

若要拒絕通訊端，用戶端會從「accept」訊息取得位址 URI，並對其附加兩個查詢字串參數，如下所示︰

| 參數 | 必要 | 說明 |
| --- | --- | --- |
| StatusCode |是 |數字型 HTTP 狀態碼。 |
| statusDescription |是 |使用者可以看懂的拒絕原因。 |

接著會使用產生的 URI 來建立 WebSocket 連線。

當正確完成時，因為尚未建立任何 WebSocket，此交握會刻意失敗，而其 HTTP 錯誤碼為 410。 如果發生錯誤，下列程式碼會描述錯誤：

| 代碼 | 錯誤 | 說明 |
| --- | --- | --- |
| 403 |禁止 |URL 無效。 |
| 500 |內部錯誤 |服務發生錯誤。 |

### 接聽程式權杖更新
接聽程式權杖即將到期時，透過所建立的控制通道對服務傳送文字框訊息即可加以取代。 訊息中包含名為 `renewToken` 的 JSON 物件，其在此階段會定義下列屬性︰

* **權杖** – 針對授予**接聽**權限的命名空間或混合式連線，有效且以 URL 編碼的服務匯流排共用存取權杖。

#### renewToken 訊息

```json
{                                                                                                                                                                        
    "renewToken" : {                                                                                                                                                      
        "token" : "SharedAccessSignature sr=http%3a%2f%2fcontoso.servicebus.windows.net%2fhyco%2f&amp;sig=XXXXXXXXXX%3d&amp;se=1471633754&amp;skn=SasKeyName"  
    }                                                                                                                                                                     
}
```

如果權杖驗證失敗，存取會遭到拒絕，而且雲端服務會關閉控制通道 Websocket 並產生錯誤。 否則不會有回覆。

| WS 狀態 | 說明 |
| --- | --- |
| 1008 |安全性權杖已過期，因此違反授權原則。 |

## 傳送者通訊協定
傳送者通訊協定的效果等同於接聽程式的建立方式。
其目標是讓端對端 WebSocket 擁有最大透明度。 要連線到的位址與接聽程式的情況相同，但其「動作」不同，而且權杖需要不同的權限︰

```
wss://{namespace-address}/$hc/{path}?sb-hc-action=...&sb-hc-id=...&sbc-hc-token=...
```

*namespace-address* 是裝載混合式連線之 Azure 轉送命名空間的完整網域名稱，其格式通常為 `{myname}.servicebus.windows.net`。

要求會包含任意的額外 HTTP 標題，包括應用程式所定義的標題。 提供的所有標題都會流向接聽程式，而且可以在 **accept** 控制訊息的 `connectHeader` 物件中找到。

查詢字串參數選項如下：

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| `sb-hc-action` |是 |傳送者角色的參數必須是 `action=connect`。 |
| `{path}` |是 |(請參閱下列段落) |
| `sb-hc-token` |是\* |針對授予**傳送**權限的命名空間或混合式連線，接聽程式必須提供有效且以 URL 編碼的服務匯流排共用存取權杖。 |
| `sb-hc-id` |否 |選擇性的識別碼，允許進行端對端診斷追蹤，並可供接聽程式在接受交握期間使用。 |

`{path}` 是用來註冊此接聽程式之預先設定混合式連線的 URL 編碼命名空間路徑。 `path` 運算式會使用後置字元和查詢字串運算式進一步通訊來進行擴充。 如果混合式連線在路徑 `hyco` 下註冊，則 `path` 運算式可能是 `hyco/suffix?param=value&...`，後接此處定義的查詢字串參數。 完整運算式則可能如下所示︰

```
wss://{namespace-address}/$hc/hyco/suffix?param=value&sb-hc-action=...[&sb-hc-id=...&]sbc-hc-token=...
```

`path` 運算式會傳遞至「accept」控制訊息所包含之位址 URI 中的接聽程式。

如果因為混合式連線路徑未註冊、權杖無效或遺失或是其他某些錯誤，導致 WebSocket 連線失敗，將使用一般的 HTTP 1.1 狀態回饋模型提供錯誤回饋。 狀態描述會包含錯誤追蹤識別碼，以供您告知 Azure 支援人員︰

| 代碼 | 錯誤 | 說明 |
| --- | --- | --- |
| 404 |找不到 |混合式連線路徑無效或基底 URL 的格式不正確。 |
| 401 |未經授權 |安全性權杖遺失、格式不正確或無效。 |
| 403 |禁止 |對於此動作來說，此路徑的安全性權杖無效。 |
| 500 |內部錯誤 |服務發生錯誤。 |

如果 WebSocket 連線在最初設定過後遭到服務刻意關閉，服務會使用適當的 WebSocket 通訊協定錯誤碼以及同時含有追蹤識別碼的錯誤描述訊息告知您這麼做的原因。

| WS 狀態 | 說明 |
| --- | --- |
| 1000 |接聽程式關閉通訊端。 |
| 1001 |混合式連線路徑已遭到刪除或停用。 |
| 1008 |安全性權杖已過期，因此違反授權原則。 |
| 1011 |服務發生錯誤。 |

## 後續步驟
* [轉送常見問題集](relay-faq.md)
* [建立命名空間](relay-create-namespace-portal.md)
* [開始使用 .NET](relay-hybrid-connections-dotnet-get-started.md)
* [開始使用 Node](relay-hybrid-connections-node-get-started.md)

