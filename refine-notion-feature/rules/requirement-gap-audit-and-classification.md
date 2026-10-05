# Rule 1 - 每個缺口必須歸入三種分類之一

- Level: `MUST`
- 對 feature、primary spec、指定 UI/UX spec 與 PM 回覆做審計時，每個發現的缺口必須分類為 `repository fact`、`product decision` 或 `engineering consideration`。
- `repository fact`：答案在當前 codebase（source code、tests、migrations 與 schema、configuration、generated contracts、repository 文件）裡。用 legwork 自行確認並把證據記錄給主管，不問 PM、不查歷史票。
- `product decision`：答案會改變外部可觀察行為、商業意義、範圍、優先級、角色權限或驗收。問 PM，且必須附建議答案與其影響。
- `engineering consideration`：答案不改變產品驗收的實作決策。帶入擁有該結果的票的工程考量，不在 refinement 階段決定架構。
- 追溯方式以需求標題或識別碼記錄，不複製產品原文。

## Good Example

- 這個例子是好的，因為三類缺口各自流向正確去處，PM 只收到真正的產品決策。

```text
- repository fact：`GET /orders` 已存在且測試涵蓋分頁 -> 記錄測試證據，不問 PM。
- product decision：「訂單取消後能否重試付款」改變使用者可見行為 -> 列為 OQ 問 PM。
- engineering consideration：重試要用 queue 還是 polling -> 帶入對應票的工程考量。
```

## Bad Example

- 這個例子是壞的，因為把 codebase 可查的事實與工程實作選擇都丟給 PM，浪費非同步往返。

```text
- OQ：請問 GET /orders 目前支援分頁嗎？
- OQ：請問重試要用 queue 還是 polling？
```

# Rule 2 - 審計必須掃描全部維度與子項

- Level: `MUST`
- 審計必須套用以下全部維度，不可只掃描想得到的面向：
  - **Outcome and scope**：actor、goal、business value、in-scope cases、explicit exclusions。
  - **Behavior**：happy path、alternate paths、empty states、invalid actions、cancellation and recovery。
  - **State**：initial state、legal transitions、terminal states、retry semantics、repeated and concurrent actions。
  - **Authorization**：actor permissions、separation of duties、visibility、disabled or ineligible actors。
  - **Contracts**：inputs、outputs、validation、errors、compatibility、events、third-party behavior。
  - **Data**：source of truth、history、migration、backfill、retention、audit evidence。
  - **Experience**：UI states、帶商業意義的 copy、accessibility-relevant behavior、operator workflows。
  - **Failure**：timeout、partial success、dependency outage、reconciliation、user-visible recovery。
  - **Operations**：rollout constraints、observability outcomes、support or manual intervention。
  - **Acceptance**：happy paths、邊界、失敗、權限與（相關時）並發的可量測斷言。
  - **Dependencies**：前置產品決策、外部團隊、provider 整備度與其他 feature。

## Good Example

- 這個例子是好的，因為掃描涵蓋狀態、失敗與權限等容易被忽略的維度，而不只看 happy path。

```text
- State：重複點擊送出時的 retry semantics 未定義。
- Failure：第三方 webhook 斷線時的 reconciliation 未定義。
- Authorization：被停權的 actor 能否看到此頁未定義。
```

## Bad Example

- 這個例子是壞的，因為只檢查功能面，遺漏狀態機、權限與失敗路徑。

```text
- 審計結果：功能流程清楚，沒有缺口。
```

# Rule 3 - OQ 識別碼依影響力順序指派

- Level: `MUST`
- 同一輪新浮現的問題，依影響力由高到低指派 `OQ-###`：會改變需求範圍、核心流程、架構邊界、角色權限、資料模型、驗收標準或不可逆決策的缺口優先取得較早編號。
- 只影響命名、體驗微調、枝節格式或可延後補充的低影響缺口排在後面。
- 較早編號先發布，讓 PM 先回答能解鎖最多工作的問題；既有 `OQ-###` 不可重新編號，新問題接續全域序號。

## Good Example

- 這個例子是好的，因為影響最大的問題拿到最早編號，第一批 comment 就能解鎖主要工作。

```text
- OQ-011：第一版是否包含 HR 後台（改變範圍）
- OQ-012：審核是單層還是雙層（改變核心流程）
- OQ-013：錯誤訊息文案（低影響，最後一批）
```

## Bad Example

- 這個例子是壞的，因為按發現順序編號，低影響瑣事搶走第一批 PM 注意力。

```text
- OQ-011：按鈕要寫「提交」還是「送出」？
- OQ-012：第一版是否包含 HR 後台？
```

# Rule 4 - 提問有預算，低風險模糊走明示假設

- Level: `MUST`
- 同一時間整個 feature 的待 PM 定案問題以五題為上限，計數依已載入的 OQ 狀態判準；新增提問須先扣除既有待定題數，不得超出剩餘額度，超額候選依影響力排序延至下一輪。
- 每題必須標記 `blocking` 或 `assumed`：會改變範圍、核心流程、權限、驗收或不可逆決策的缺口是 `blocking`；其餘低風險模糊可標 `assumed`，在 Recommendation 明示預設答案，其生效與撤回依已載入的 OQ 狀態判準。
- Phase 3 先把 `assumed` 的預設答案、來源與受影響需求記入既有 OQ registry；Phase 5 草擬票時再引用 OQ 識別碼，將有效假設寫入受影響票的工程考量，未承接前不得交付核准或發布。
- 若五題內仍無法收斂關鍵 blocking 缺口，明確向主管說明殘餘風險與卡點，不無限追問。

## Good Example

- 這個例子是好的，因為低風險歧義用明示假設放行，只有真正卡住範圍的問題擋住拆票。

```text
- OQ-014：歷史訂單是否納入本次範圍 -> blocking（改變 scope）。
- OQ-015：空狀態顯示插圖還是純文字 -> assumed，以 Recommendation 為暫定預設，
  Phase 3 記入 registry，Phase 5 才寫入 UI slice 票的工程考量。
```

## Bad Example

- 這個例子是壞的，因為把所有模糊點都當 blocking，讓不影響驗收的問題也卡住整個流程。

```text
- OQ-014：空狀態文案用哪一版 -> blocking，等 PM 回覆前不拆任何票。
```

# Rule 5 - 已收斂或可唯一推論的缺口不再提問

- Level: `SHOULD`
- PM 已明確拍板的決策、或可由既有上下文推出唯一合理解讀的缺口，不應重複提問。
- 提問目的是移除高風險不確定性，不是把所有資訊改寫成問卷。

## Good Example

- 這個例子是好的，因為既有 spec 已寫明的事不再佔用提問額度。

```text
- Spec 已明定「管理員可見全部訂單」-> 不再問管理員可見範圍。
```

## Bad Example

- 這個例子是壞的，因為把 spec 已回答的事再問一次，製造無效往返。

```text
- OQ-016：請問管理員可以看到全部訂單嗎？（spec 已明定）
```
