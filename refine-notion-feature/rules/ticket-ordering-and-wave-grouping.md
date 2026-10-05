# Rule 1 - 行為票採 vertical slice，必要前置技術票明示例外

- Level: `MUST`
- 行為票切穿所有必要層級，形成窄而完整的 vertical slice，能在宣告的 blockers 完成後於一個 fresh implementation context 中獨立展示或驗證。
- 依系統介面盤點確認必要的共用成果或部署限制時，可例外切出前置技術票；它必須交代成果、可驗證性與下游相依，不把例外擴張成按技術層分工。
- 每張票落地後系統必須維持可運作狀態；宣告的 blocking edge 必須是真實相依，blocker 先落地。
- 不可只按層水平切票（前端一票、後端一票、測試一票），也不可切出需要其他票先落地才能驗證卻未宣告相依的票。
- 票只在 Notion 存在，frontier 傳達當下可拿的工作；不產出本地計畫檔，也不另設 checkpoint 票。

## Good Example

- 這個例子是好的，因為單一 slice 的 migration 與行為一起交付，下游票有真實 API 相依。

```text
- 票 1：結帳 API 建立訂單並回傳 retry_token（含只服務本 slice 的 migration 與測試）
- 票 2：付款失敗頁顯示重試入口並串接票 1 的 API（blocked by 票 1）
```

## Bad Example

- 這個例子是壞的，因為按層水平切，任何一票落地都無法單獨驗證。

```text
- 票 1：所有 DB migration
- 票 2：所有 API endpoint
- 票 3：所有前端畫面
```

# Rule 2 - 票的大小以 S 與 M 為目標，L 以上必須繼續拆

- Level: `MUST`
- 依下表評估每張票：

| Size | 約略檔案數 | 範圍 |
|------|-----------|------|
| XS | 1 | 單一函式或設定變更 |
| S | 1–2 | 一個 component 或 endpoint |
| M | 3–5 | 一個 feature slice |
| L | 5–8 | 多 component feature |
| XL | 8+ | 過大，必須繼續拆 |

- 觸及兩個以上獨立子系統（如帳號與金流）的票必須拆開。
- 標題需要「and／與／以及」才能描述的票通常是兩張票。
- 無法在單一 focused implementation context 驗證的票繼續拆。
- size 只是拆票啟發法，不發布在票上；票上只記錄 `理想工程工時`。

## Good Example

- 這個例子是好的，因為觸及金流與通知兩個子系統的工作被拆成兩張可獨立驗證的票。

```text
- 票 A：付款失敗時產生重試 token 的 API
- 票 B：重試失敗的通知發送
```

## Bad Example

- 這個例子是壞的，因為一張票同時跨金流與通知子系統，估 8+ 檔案仍未拆。

```text
- 票 A：實作付款失敗重試與通知發送（預估 9 個檔案）
```

# Rule 3 - blocking edge 必須真實且落地順序保持系統可用

- Level: `MUST`
- 只宣告真實存在的相依；每條 blocking edge 的 blocker 必須排在相依票之前落地。
- 根據系統介面盤點確認的共用先行成果與部署限制形成相依；必要前置票先落地，彼此不依賴的 slice 才可平行，不只因修改相同對象就添加 blocking edge。
- 高風險或未知量高的 slice 排早，讓整合意外在大部分工項前浮現。
- 每兩到三張票對齊一個 verification checkpoint，作為給 review 者的排序訊號，不發布成票。
- expand–contract 只用於無法以 vertical slice 落地綠燈的大型機械式 refactor。

## Good Example

- 這個例子是好的，因為共用 contract 變更先以前置票落地，相依票再平行展開。

```text
- 票 1（前置）：`POST /orders` contract 加 `retry_token` 欄位
- 票 2：重試 API（blocked by 票 1）
- 票 3：後台顯示（blocked by 票 1，可與票 2 平行）
```

## Bad Example

- 這個例子是壞的，因為宣告了不存在的相依，或讓相依票排在 blocker 之前。

```text
- 票 2：重試 API（blocked by 票 3，但票 3 排在票 2 之後才落地）
```

# Rule 4 - 給主管的 draft 必須分組 Wave 並交代安排理由

- Level: `MUST`
- 呈給主管的編號 draft 必須把票分組為 Wave：同一 Wave 只放彼此不需要等待對方產出的票。
- 每個 Wave 必須寫明「交付重點」與「安排理由」：說明這一波實際要收斂什麼、為什麼這些票同波、為什麼排在這個順序。
- Wave 順序必須反映相依與資訊供給順序，不是平均分配端點；frontier 是 Wave 1 中無 blocker 的票。

## Good Example

- 這個例子是好的，因為每波都有焦點與理由，主管能 review 相依邏輯而非只看箭頭。

```md
#### Wave 1
- 平行票：票 1（orders schema 前置 migration）、票 4（金流 webhook 文件調查 spike）
- 交付重點：共用 schema 落地、第三方限制收斂
- 安排理由：兩者互不依賴，且都是後續票的前置資訊

#### Wave 2
- 平行票：票 2（重試 API）、票 3（後台顯示）
- 交付重點：依賴票 1 schema 的兩個獨立 slice
- 安排理由：同樣依賴 Wave 1 產出，彼此無相依
```

## Bad Example

- 這個例子是壞的，因為只列票名沒有理由，且把有相依的票塞進同一波。

```md
#### Wave 1
- 票 1、票 2、票 3
```
