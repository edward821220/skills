## 來源

- Feature ticket：https://www.notion.so/feat-subscription-retry-001
- 草稿識別碼：DR-6d2c9ce1-2cb8-446f-9b76-dc1c45b37e90
- Primary spec：https://www.notion.so/spec-subscription-retry-002
- UI/UX spec：https://www.notion.so/ux-subscription-retry-003
- Prototype：
  - 結帳失敗頁：https://www.figma.com/proto/retry-checkout-fail
- 對應需求：FR-004、FR-006
- 對應 Feature AC：AC-002、AC-005

## 技術交付內容

打通付款失敗後的重試入口：`POST /orders/:id/retry` 接受重試請求、檢查訂單狀態合法轉換、回傳 `RETRY_PENDING` 與 `retry_token`。完成後可用 API client 對一筆 `PAYMENT_FAILED` 訂單發起重試並觀察到狀態轉換與 token 回傳。

## 影響標記

- MODIFY：`POST /orders/:id/retry` 端點回應新增 `retry_token` 欄位
- NOOP：`orders` 表 schema（重試 API 與後台共用的 migration 已由前置票完成，本票不動）

## 實作前必讀

- Primary spec「付款重試」章節
- `POST /orders/:id/retry` contract 草案
- OQ-011 的 PM 回覆

## 工程考量

### 考量 1：重試 token 的驗證與失效機制

- 決策點：在 Primary spec「付款重試」已確定有效期限、使用次數與重複操作結果的前提下，如何實作 token 驗證與失效
- 考慮過的方案：服務端保存 token 狀態、簽章 token 搭配必要的失效紀錄
- 擱置理由：由工程師依既有機制與安全要求選擇，只保留滿足相同產品與安全要求的方案；有效期限、使用次數或重複操作語意若仍有缺口，回到 PM 釐清，不在本票自行定案

## 估時

- 理想工程工時：2.5 人日

> 此數值為不中斷投入下的初步工程估算，不是交付日期承諾。若實作者評估後認為難以在估時內完成，或實作過程發現可能延遲，請在 task 留言或私訊主管，依最新資訊動態調整估時。

## 前置任務

- https://www.notion.so/task-orders-retry-migration-010

## 任務邊界（選填）

- 只做：重試 API 的行為、狀態轉換與 contract
- 不做：`orders` 表 migration（前置票已落地）、後台重試狀態顯示（相依票負責）

## Tech Spec（工程師填寫）

> 本區由負責工程師在實作前完成，可使用此模板或是用自己習慣的方式。

### 現況與既有 seam

- <目前相關行為、可重用的 module／interface／trait，以及既有技術限制>

### 技術方案

- <預計採用的設計與資料流>
- <關鍵 technical decision、替代方案與選擇理由>

### Contract 與資料變更

- <API、event、schema、migration、generated artifact 或相容性影響；沒有則填「無」>

### 測試與驗證策略

- <預計在哪些 seam 驗證此任務的技術交付與 Feature AC>
- <必要的 unit、integration、contract 或 end-to-end test>

### Migration、Rollout 與 Observability

- <部署順序、backfill、feature flag、rollback、metric、log 或 audit 需求；沒有則填「無」>

### 風險與待確認事項

- <技術風險、未知因素與緩解方式>
- <是否需要 ADR、domain 文件或其他工程決策紀錄>

### 實作計畫

- [ ] <依 dependency 排列的實作步驟 1>
- [ ] <實作步驟 2>
- [ ] <驗證與收尾步驟>
