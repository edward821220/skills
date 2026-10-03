# Notion Task Ticket 模板

本模板定義發布至 Notion task database 的工程任務格式。引用 feature、primary spec、需求與 feature AC 的穩定識別碼，不重複產品背景、商業驗收文字或 feature-level Out of Scope。

## 語言規則

Task 標題、標題層級、說明、估時與驗證文字一律使用台灣用語繁體中文（`zh-TW`）。使用「資料、介面、回傳、預設、建立、程式碼」等台灣慣用詞。程式識別字、API path、schema、enum value 與產品名稱保留原文。

## 關聯規則

建立頁面時，把 task database 中指向 feature database 的 relation 設為 feature ticket，並確認 feature database 的 reciprocal relation 會顯示該 task。下方「來源」段落是供人閱讀的追溯資訊，不能取代資料庫 relation。

「來源」的 Prototype 列從 feature 的「影響頁面」抄：每一列寫頁面名稱與原型網址（同一列多個網址都帶上）。feature 沒有「影響頁面」、或整表都沒有原型網址，就省略 Prototype 整項。只抄頁面名稱與網址。

```markdown
## 來源

- Feature ticket：<Notion feature link>
- Primary spec：<Notion spec link>
- UI/UX spec：<Notion UI/UX spec link；沒有則省略>
- Prototype：
  - <影響頁面列的頁面名稱>：<原型網址>
- 對應需求：<穩定的 requirement ID 或標題>
- 對應 Feature AC：<AC ID；只引用，不複製商業驗收文字>

## 技術交付內容

<說明這個窄而完整的 vertical tracer bullet 會打通哪些技術行為、contract 或 seam，以及完成後可觀察的結果。>

## 工程考量

- <由負責工程師決定的 constraint 或 technical decision；沒有則省略>

## 估時

- 理想工程工時：<由 refinement skill 填入預設值，例如 2 人日>

> 此數值為不中斷投入下的初步工程估算，不是交付日期承諾。若實作者評估後認為難以在估時內完成，或實作過程發現可能延遲，請在 task 留言或私訊主管，依最新資訊動態調整估時。

## 前置任務

- <Blocking ticket link，或「無，可立即開始」>

## 任務邊界（選填）

- <只有相鄰 task 的責任容易混淆時，才說明由哪張 task 負責；其餘情況省略>

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
```

## 估時與拆票規則

- Skill 拆票時為每張 task 填入一個理想工程工時預設值，統一使用 `N 人日` 格式。
- `1 人日 = 8 小時`；可使用 `0.5 人日`的半日粒度，不發布小時格式的估時。
- Owner 只放在 Notion owner property，不重複寫入 task 內文。
- 無法在單一 fresh implementation context 完成驗證的 slice 要繼續拆分。
- 若任務觸及兩個以上獨立子系統（如帳號與金流），應拆分為獨立票；涉及共用資料庫 migration、共用狀態或共用 API contract 者，必須設為循序相依的前置任務。
- 若實作者評估後認為難以在估時內完成，或實作過程發現可能延遲，由實作者在 task 留言或私訊主管，依最新資訊動態調整估時。

## Ticket 品質關卡

每張 ticket 即使有主要負責人，仍須是 vertical tracer bullet。Ticket 聚焦技術交付目標、對應需求與 Feature AC 參照、contract、seam 與相關工程考量，不預先規範硬性的技術驗收條件與完成證據，讓負責工程師在 `Tech Spec` 階段保有設計與驗證的發揮空間。Refinement 發布 task 時，`Tech Spec` 只放空白模板。feature「影響頁面」有原型網址時，「來源」必須帶上對應的 Prototype 列。
