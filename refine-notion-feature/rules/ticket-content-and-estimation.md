# Rule 1 - 票內容使用台灣繁體中文，程式識別物保留原文

- Level: `MUST`
- 票標題、標題層級、說明、估時與驗證文字一律使用台灣用語繁體中文（`zh-TW`），使用「資料、介面、回傳、預設、建立、程式碼」等台灣慣用詞。
- 程式識別字、API path、schema 名稱、enum 值與產品名稱保留原文。

## Good Example

- 這個例子是好的，因為敘述用台灣用語，而 API path 與 enum 保留原文。

```md
使用者呼叫 `POST /orders/:id/retry` 後，系統回傳 `RETRY_PENDING` 狀態。
```

## Bad Example

- 這個例子是壞的，因為混用中國用語與半形標點，或把程式識別字翻成中文。

```md
用户调用“重试接口”后，系统返回"重试中"状态。
```

# Rule 2 - 引用識別碼，不複製產品文字、不訂任務級驗收

- Level: `MUST`
- 票以穩定識別碼參照 requirement 與 Feature AC，不複製其商業驗收文字，也不重複 feature-level Out of Scope。
- 不為票定義硬性技術驗收條件或完成證據；技術設計與驗證策略留給負責工程師在 Tech Spec 階段決定。
- refinement 新建票時 `Tech Spec` 區塊只放空白模板，由負責工程師在實作前填寫；恢復既有票的發布時保留工程師已填內容，不重設成空白。

## Good Example

- 這個例子是好的，因為只引用 AC 識別碼，驗收定義維持單一溯源在 feature。

```md
- 對應 Feature AC：AC-002、AC-005
```

## Bad Example

- 這個例子是壞的，因為把 feature AC 整段複製進票，之後 spec 更新時兩處會分叉。

```md
- 對應 Feature AC：當使用者付款失敗時，系統必須在 5 秒內…（整段貼上）
```

# Rule 3 - 實作前必讀只列穩定識別物，禁寫檔案路徑

- Level: `MUST`
- 每張票的「實作前必讀」列出工程師動手前需要看的穩定識別物：spec 章節標題、requirement ID、Feature AC ID、contract 或事件名稱、schema 名稱、Notion 連結。
- 禁止寫檔案路徑；路徑在 refinement 到實作之間會過期，反而誤導。
- 沒有需要先讀的來源時省略整個區塊，不寫「無」。

## Good Example

- 這個例子是好的，因為列的都是不會過期的識別物。

```md
## 實作前必讀

- Primary spec「付款重試」章節
- `POST /orders/:id/retry` contract 草案
- OQ-011 的 PM 回覆
```

## Bad Example

- 這個例子是壞的，因為列了會過期的檔案路徑。

```md
## 實作前必讀

- `src/api/orders/retry.ts`
- `src/pages/Checkout.tsx`
```

# Rule 4 - 工程考量用 decision-record 格式，只記錄不裁決

- Level: `MUST`
- 每筆工程考量包含三欄：`決策點`（留給工程師決定的問題）、`考慮過的方案`（refinement 時看到的候選）、`擱置理由`（為什麼留給工程師）。
- 記錄考量不等於決定架構；若 PM gate 或主管已定調，`決策點` 寫明已定方向。
- `assumed` OQ 的暫定假設若影響此票，草擬時必須引用 OQ 識別碼並在此區塊記錄預設答案；未承接前不得交付核准或發布，不把 PM 沉默記成已核定。
- token 的有效期限、使用次數與重複操作結果若未定義，屬於 product decision，不能當成工程考量擱置；只保留滿足相同產品與安全要求的實作方案。
- 沒有工程考量時省略整個區塊。

## Good Example

- 這個例子是好的，因為交代了候選與擱置理由，工程師不用重新考古。

```md
### 考量 1：重試通知的派送方式

- 決策點：在既定通知結果與延遲要求下，用既有 queue 還是呼叫通知 service
- 考慮過的方案：`OrderEventQueue` 發事件、內部 API 呼叫
- 擱置理由：由工程師確認哪種方案符合已定產品要求，refinement 不代為定案

### 考量 2：assumed OQ-015 假設（UI slice 票）

- 決策點：暫採純文字空狀態，並非 PM 已核定
- 考慮過的方案：插圖加文案、純文字
- 擱置理由：registry 的低風險假設承接到本票；PM 反對時重新檢查草稿與核准範圍
```

## Bad Example

- 這個例子是壞的，因為只有一句話，工程師不知道考量過什麼、為什麼未定案。

```md
## 工程考量

- 重試通知可以用 queue。
```

# Rule 5 - 估時與 owner 的記錄方式

- Level: `MUST`
- refinement 為每張票填一個 `理想工程工時`，統一 `N 人日` 格式；`1 人日 = 8 小時`，允許 `0.5 人日` 半日粒度，不發布小時格式。
- 估時是不中斷投入下的初步工程估算，不是交付承諾；實作者評估後認為難以在估時內完成或發現可能延遲時，在票留言或私訊主管動態調整。
- Owner 只設在 Notion owner property，不寫進票內文；不發布 size 欄位。

## Good Example

- 這個例子是好的，因為格式正確且語意為初步估算。

```md
- 理想工程工時：2.5 人日
```

## Bad Example

- 這個例子是壞的，因為用了小時格式並把 owner 寫進內文。

```md
- 估時：20 小時
- 負責人：王小明
```

# Rule 6 - 任務邊界以只做／不做成對書寫，且僅在必要時填

- Level: `MUST`
- 「任務邊界」只在相鄰票責任容易混淆時填寫，其餘情況省略整個區塊。
- 填寫時必須成對寫 `只做` 與 `不做`：`不做` 明確指出留給相鄰票或超出範圍的部分。

## Good Example

- 這個例子是好的，因為不做的一半防止工程師順手做進別張票的範圍。

```md
## 任務邊界

- 只做：重試 API 的行為與 contract
- 不做：`orders` 表 migration（前置票負責）、後台顯示（相依票負責）
```

## Bad Example

- 這個例子是壞的，因為只說自己負責什麼，沒有擋住越界行為。

```md
## 任務邊界

- 由本票負責重試功能。
```

# Rule 7 - 發布識別、關聯與 Prototype 的設法

- Level: `MUST`
- 每張 draft 首次草擬時產生固定草稿識別碼 `DR-<UUID>`，與顯示編號分離；重新排序不改碼，核准後恢復發布沿用原稿原碼。同一識別碼不挪作另一張票；合併或拆分的替代票使用新碼並重新核准。
- 「feature page ID＋草稿識別碼」是發布比對鍵，不只靠標題判定同一票；保留主管核准稿與識別碼在主管對話中，不產出本地計畫檔。恢復時缺少原核准稿或其識別碼，先請主管提供，不自行重建。
- 草稿識別碼優先寫入 schema 中已有且適合的可寫 property；沒有時使用「來源」中的固定「草稿識別碼」欄位，不為此新增 database property。採 property 時可省略內文重複欄位，但兩者若同時存在必須一致。
- create 時一併寫入草稿識別碼與 task→feature relation；工具無法同次記錄時暫停，不先建立無法恢復比對的票。「來源」連結不能取代 relation；發布後確認 reciprocal relation 顯示該票。
- 每次 create 前都查詢 feature 下的既有 task，包括完整分頁結果，只把識別、內容與關聯用於發布比對，不把既有票當成產品來源。零筆確認對應才建立；一筆則驗證核准的交付範圍與來源，保留工程師已填的 Tech Spec 與後續工作狀態，只補齊可確認的未完成發布操作；多筆則停止。
- create 結果不明時先重新查證；若無法確認前次未建立，停止回報，不因暫時查不到就重送。疑似本輪既有票卻缺識別資訊、內容衝突或無法唯一對應時也停止，不猜測、不覆蓋工程師後續填寫的 Tech Spec、不自行合併或刪除。
- 這是單一發布者的恢復流程，不保證多 agent 並行時嚴格只建立一次；已知有另一發布者時暫停，請主管指定單一發布者。
- 「來源」的 Prototype 列從 feature 的「影響頁面」抄寫：每列寫頁面名稱與原型網址；feature 沒有「影響頁面」或整表無網址時省略 Prototype 整項，只抄名稱與網址不加說明。
- status、assignee、估時與 blocker 只在 schema 支援處填寫；無原生 blocking 關係時用內文「前置任務」fallback。

## Good Example

- 這個例子是好的，因為原核准稿保留識別碼，重跑只補缺漏，Prototype 仍照抄來源。

```md
- 原核准稿 A、B 各有固定草稿識別碼；A 查到唯一 task 且內容與關聯正確，B 確認尚未建立 -> 沿用 A，只建立 B。
- A 的 Tech Spec 已由工程師填寫 -> 保留，不用核准時的空白模板覆蓋。
- Prototype：
  - 結帳失敗頁：https://www.figma.com/proto/xxx
```

## Bad Example

- 這個例子是壞的，因為恢復時重建身份或忽略不確定結果，且編造 Prototype。

```md
- create timeout 後換新草稿識別碼直接重建，或查到兩筆同鍵 task 仍宣告完成。
- 缺少原核准稿時只靠標題猜是哪張票，再覆蓋既有 task。
- Prototype：
  - 結帳失敗頁：https://www.figma.com/proto/yyy（推測應該是這個）
```
