## 來源

- Feature ticket：{{FEATURE_TICKET_LINK}}
- 草稿識別碼：{{DRAFT_ID}}
- Primary spec：{{PRIMARY_SPEC_LINK}}
- UI/UX spec：{{UI_UX_SPEC_LINK}}
- Prototype：
  - {{AFFECTED_PAGE_NAME}}：{{PROTOTYPE_URL}}
- 對應需求：{{REQUIREMENT_IDS}}
- 對應 Feature AC：{{FEATURE_AC_IDS}}

<!--
  - 沒有指定 UI/UX spec 時省略該列；feature「影響頁面」沒有原型網址時省略 Prototype 整項。
  - 對應需求與 Feature AC 只引用穩定識別碼，不複製商業驗收文字。
  - 草稿識別碼填原核准稿的 DR-<UUID>；若已寫入對應 property，可省略內文此列。
-->

## 技術交付內容

{{TECH_DELIVERY_DESCRIPTION}}

<!--
  說明行為票的 vertical slice 打通哪些技術行為、contract 或 seam，以及完成後可觀察的結果；必要前置技術票則說明共用成果、可驗證性與下游相依。
-->

## 影響標記

- {{ADD_MODIFY_DELETE_NOOP}}：{{IMPACT_TARGET_AND_EXISTING_OBJECT}}

<!--
  MODIFY 與 DELETE 必須指出受影響的既有 contract、schema、端點或行為。
-->

## 實作前必讀

- {{MUST_READ_REF}}

<!--
  只列穩定識別物：spec 章節標題、requirement ID、contract 名稱、Notion 連結。禁寫檔案路徑。無內容時省略整個區塊。
-->

## 工程考量

### 考量 1：{{CONSIDERATION_TITLE}}

- 決策點：{{ENGINEER_DECISION_POINT}}
- 考慮過的方案：{{CONSIDERED_OPTIONS}}
- 擱置理由：{{DEFERRAL_RATIONALE}}

<!--
  記錄考量不決定架構；assumed OQ 若影響本票，引用識別碼並記錄暫定答案，不把 PM 沉默寫成核定。無考量時省略整個區塊。
-->

## 估時

- 理想工程工時：{{PERSON_DAYS}} 人日

> 此數值為不中斷投入下的初步工程估算，不是交付日期承諾。若實作者評估後認為難以在估時內完成，或實作過程發現可能延遲，請在 task 留言或私訊主管，依最新資訊動態調整估時。

## 前置任務

- {{BLOCKING_TICKET_LINKS_OR_NONE}}

<!--
  填 blocking ticket 連結，或「無，可立即開始」。
-->

## 任務邊界（選填）

- 只做：{{IN_SCOPE_OWNERSHIP}}
- 不做：{{OUT_OF_SCOPE_OWNERSHIP}}

<!--
  只在相鄰票責任容易混淆時填寫，只做與不做必須成對；其餘情況省略整個區塊。
-->

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
