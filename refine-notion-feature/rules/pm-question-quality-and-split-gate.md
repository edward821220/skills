# Rule 1 - 每個 PM 問題必須具備七個欄位

- Level: `MUST`
- 每個送給 PM 的問題必須包含：穩定 `OQ-###` 識別碼、需求來源與目前解讀、一個暴露缺口的具體情境、一個決策型問題、一個建議答案、受影響的行為／範圍／估時／票邊界，以及 `Gate` 標記（`blocking` 或 `assumed`）。
- `assumed` 題目必須在 Recommendation 中寫明預設答案；暫定假設依 Rule 3 生效，不把 PM 沉默視為核定。
- 受影響需求必須能由 Source 或 Impact 唯一辨識：優先引用需求識別碼，沒有識別碼時使用白名單權威文件與具體章節或需求標題。Source 已完整列出的需求不必在 Impact 重複；跨多個需求的影響須在 Impact 補齊其餘需求參照，不得只寫「影響 UI slice」。

## Good Example

- 這個例子是好的，因為欄位齊全，PM 能直接決策，也知道不回答時的走向。

```md
#### OQ-011 — 歷史訂單是否納入批次重寄範圍

**Source:** FR-004
**Current interpretation:** spec 未定義資料範圍
**Scenario:** 上線前已存在的五千筆訂單是否會收到重寄通知
**Question:** 歷史訂單是否納入本次範圍？
**Recommendation:** 不納入，只處理新訂單；歷史資料另開需求評估
**Impact:** 決定 backfill 票是否存在，影響約 2 人日
**Gate:** blocking
```

## Bad Example

- 這個例子是壞的，因為沒有具體情境與建議答案，PM 無法判斷也無法快速核准。

```md
#### OQ-011 — 資料範圍
**Question:** 訂單資料範圍是什麼？
```

# Rule 2 - 每題只處理一個決策軸

- Level: `MUST`
- 若一個問題要求 PM 同時回答兩個以上獨立決策，必須拆成不同 `OQ-###`。
- Context 可以補充前因後果，但 `Question` 只要求回答一件事。

## Good Example

- 這個例子是好的，因為範圍與通知通道是兩個獨立決策軸，各自可單獨作答。

```text
- OQ-011：第一版是否包含 HR 後台？
- OQ-012：通知第一版走 email 還是站內訊息？
```

## Bad Example

- 這個例子是壞的，因為一題塞了範圍、整合與時程三個決策，PM 無法用單一答案回覆。

```text
- OQ-011：要不要做 HR 後台、要不要串 Slack，而且兩週內上線可以嗎？
```

# Rule 3 - OQ registry 與分批發布紀律

- Level: `MUST`
- 首次載入、發布或重試前，必須讀取 feature 上全部 comments 與 discussions，包含完整分頁結果，再辨識問題與 PM 回覆，不提前排除未寫 `OQ-` 的留言。
- 問題原文以問題標題中的 `OQ-###` 識別碼與七欄位結構辨識；單純提及或引用識別碼不算新問題。既有題欄位不全時，仍保留識別碼並記錄缺漏，不因格式缺漏而重用識別碼。
- PM 回覆以 discussion 關聯、明示識別碼或可唯一判定的上下文對應；一則 discussion 有多題時，不因同串就視為全部已回答。對應不明確時，優先在原問題的 discussion 內追問；未寫識別碼但可唯一對應的 PM 回覆仍有效。
- OQ registry 是可重建的暫存資料，不另設持久化檔案或 Notion property。依白名單產品來源、已發布 OQ 留言與 PM 回覆，記錄每題的 Gate、決策狀態、目前有效決策或暫定答案、來源、受影響需求與問題／回覆的 comment 或 discussion 參照；不能只靠主管對話重建已生效假設。
- 已發布票僅可輔助追蹤受影響 task，不作為產品決策來源，也不是重建 registry 的必要條件。識別碼全域唯一、永不重用，新問題接續序號。
- Gate 的 `blocking`／`assumed` 是風險分類，與下列決策狀態分開；狀態由證據推導，不寫成新的留言欄位：

| 決策狀態 | 判定與可用內容 |
|---|---|
| `open` | 尚無可用決策，回覆對應或內容仍歧義，或權威性、來源衝突未解決；無權威決策時，未成功發布的預設不能補足缺口。 |
| `assumed-active` | Gate 為 `assumed`，低風險預設、來源與受影響需求已在 OQ 留言明示且確認發布，未被 PM 反對，也沒有未解決的 PM 回覆歧義或來源衝突；可暫用但不等於核定。 |
| `resolved` | 白名單權威產品來源或明確、具權威性的 PM 決策已消除缺口，且沒有未解決衝突；使用該決策，不沿用已失效的 Recommendation。 |

- 依可唯一對應的最新有效決策更新狀態；有人回覆不等於 `resolved`。PM 回覆若實質改變 primary spec，在權威 spec 記錄該決策或 PM 明示其回覆具權威性之前，該題維持 `open`。
- PM 反對假設時立即撤回原預設；有明確且具權威性的替代決策才轉為 `resolved`，否則維持 `open` 並重新判斷 Gate，不以後續沉默重新啟用被反對的假設。重新審計受影響需求與草稿，檢查票邊界、估時及 blocking edge；有實質變更時重新取得主管核准。
- 計入待 PM 定案提問預算的狀態為 `open` 與 `assumed-active`，`resolved` 不計入；同一識別碼只計一次，不以暫用假設釋放提問額度。
- 按識別碼升冪把本輪可發布問題切成每批至多三題的連續批次，每批一則 comment；每則 comment 只含一個 `Questions for PM` section，不加批次編號、AI 署名、已確立事實、審計摘要、repo 發現或後續步驟說明。
- comment 建立部分成功或結果不明時，重新載入全部 comments 比對已持久化識別碼，只補發仍缺的問題，不盲目重發整批；未確認發布的暫定假設不得用來放行，確認已發布後更新狀態並重新檢查 gate。

## Good Example

- 這個例子是好的，因為發布可安全恢復，假設及狀態也能從來源與留言重建。

```text
- 預計發 OQ-011..013，比對後發現 OQ-011 已持久化 -> 只補發 OQ-012 與 OQ-013。
- 新 session 由 OQ-015 的 Recommendation、Source 與 Impact 重建純文字空狀態假設，不需要工程票已存在。
- OQ-015 的 Source 指向 UI/UX spec「通知中心／無重寄通知空狀態」，Impact 補列另一個受影響需求 FR-007；兩者都納入受影響需求映射。
- PM 在含多題的 discussion 回覆「OQ-011 不納入歷史訂單」-> 只更新 OQ-011；沒有識別碼但可唯一對應 OQ-012 的獨立回覆也納入。
- PM 反對 OQ-015 且未給明確替代決策 -> 撤回預設、狀態回到 open，不能繼續沿用純文字假設。
```

## Bad Example

- 這個例子是壞的，因為重複發布、錯誤配對或無證據放行都會使 registry 失真。

```text
- comment 建立回傳 timeout -> 直接重發整批，OQ-011 出現兩次。
- 只掃描寫有 OQ- 的回覆 -> 漏掉可唯一對應的 PM 回答。
- PM 回覆「再看看」-> 因有人回答就標 resolved，或把同串所有題都標已解決。
- assumed 題的 Source 與 Impact 無法定位受影響需求，映射只留在主管對話 -> 無法可靠重建。
- 預設只在未發布草稿，或 PM 已反對 -> 仍標 assumed-active 並放行。
```

# Rule 4 - ready-to-split gate 的通過條件

- Level: `MUST`
- 拆票只在下列條件全部成立時通過：
  - 已確認並載入權威 primary spec，且 zero open `blocking` product decision；因提問預算延後發布的 blocking 缺口仍會擋住 gate。
  - `assumed` 題已為 `assumed-active` 或 `resolved`；暫用假設的預設答案、來源與受影響需求可由已發布 OQ 留言重建並記入 registry，不要求工程票已存在；草擬後才檢查有效假設是否承接到票。
  - 每個 in-scope 行為都有可測的產品結果，必要時由上述明示暫定假設補足。
  - 相關處的 actor、permissions、state meanings、failure behavior 與 scope 邊界沒有尚待決策的歧義。
  - PM 回覆與 primary spec 沒有未解決的衝突。
  - 依賴關係可辨識到足以草擬 blocking edge。
  - engineering consideration 可以在票內被擁有，且不改變產品驗收。

## Good Example

- 這個例子是好的，因為 blocking 全數收斂，暫定假設已記錄，不需要先有工程票才能草擬。

```text
- OQ-011（blocking）PM 已明確答覆並消除來源衝突，狀態為 resolved。
- OQ-015（assumed）留言已確認發布且可重建純文字空狀態、來源與受影響 UI 需求，狀態為 assumed-active -> gate 通過。
- Phase 5 再把 OQ-015 假設寫入 UI slice 票；未承接前不交付核准或發布。
```

## Bad Example

- 這個例子是壞的，因為真正的 blocking 缺口仍開放，或假設沒有可追溯的預設內容。

```text
- OQ-011（blocking）PM 尚未回覆 -> gate 不通過，仍不得拆票。
- OQ-015 只標了 assumed，留言未確認發布、映射不明或原假設已被反對，狀態仍為 open -> gate 不通過。
```
