---
name: refine-notion-feature
description: 將一個 Notion PM feature ticket 與其 primary spec 精煉成主管核准並發布的工程任務票。涵蓋產品來源載入、需求缺口審計、非同步 PM 釐清 gate、系統介面盤點、拆票草擬與 Notion 發布。使用時機：使用者提供 Notion feature ticket URL，要求釐清需求或拆工程票。
disable-model-invocation: true
---

# Refine Notion Feature

將一個 PM feature 與其 primary spec 精煉成主管核准、已發布的工程任務票。PM 釐清未完或工程票發布完成時結束；實作屬於被指派工程師。

## 輸入

需要一個 Notion feature-ticket URL。feature、其連結的 primary spec、明確指定的 UI/UX spec（若有）與 feature 上的 PM 回覆是產品唯一真相來源；目前使用者是核准拆票的工程主管。

# SOP

## Phase 1 -- 載入產品來源白名單

1. READ 讀取 feature ticket、其 discussions 與全部 comments；從 feature 的連結只抓取完整 primary spec 與明確指定的 UI/UX spec，不抓取其他連結頁面或票，包含 spec 內出現的連結、歷史任務、相關 feature 與相依票。database／data-source schema 與 related-task 發布比對只作為操作 metadata，不把 task rows 當成需求脈絡。
2. READ 讀取 `rules/pm-question-quality-and-split-gate.md`，依其辨識已載入 comments 中的問題與 PM 回覆，重建全域 OQ registry 並確認每個 `OQ-###` 的決策狀態。
3. THINK 探索當前 codebase（source code、tests、migrations 與 schema、configuration、generated contracts、repository 文件），確認已實作行為、可行性、冗餘、技術相依與拆票邊界的事實。
4. THINK 若未指定 primary spec 或無法判定權威來源，只將「確認權威 spec」列為 blocking product decision 進入 Phase 3，不宣稱已完成審計；來源確認後回到 Phase 1 載入並執行 Phase 2。若已知來源但無法存取，停在 Phase 1 回報存取限制。

**完成條件**：正常路徑已載入 feature、其 comments、primary spec、指定 UI/UX spec（若有）與全部既有 PM 回答，每個 `OQ-###` 狀態已知；來源不明時只走權威來源釐清出口。兩條路徑都未抓取白名單外的 Notion 頁面或票。

## Phase 2 -- 審計需求缺口並分類

1. READ 讀取 `rules/requirement-gap-audit-and-classification.md`，依其審計維度掃描 feature、primary spec、指定 UI/UX spec 與 PM 回覆。
2. THINK 依已載入判準將每個缺口分類：以 codebase legwork 解決 repository fact 並記錄證據；open product decision 帶入 Phase 3；engineering consideration 帶入對應票的工程考量，不決定架構。

**完成條件**：每個發現的缺口已分類為已解決事實、open product decision 或 engineering consideration；每個需求已辨識可測的產品結果，或其缺口已分類並記錄處理去向。

## Phase 3 -- 執行非同步 PM 釐清 gate

1. THINK 依已載入判準確認當前 OQ registry 與拆票 gate；若無新問題需發布且 gate 已通過，跳至 Phase 4。
2. THINK 依已載入判準編排新問題、指派穩定識別碼並分批，將分類結果與暫定假設記入 OQ registry。
3. READ 讀取 `templates/pm-question-comment.md` 與 `templates/pm-question-comment.example.md`，依骨架複製結構、參考範例填寫每批留言。
4. WRITE 先讀取 `rules/notion-writing-quality-gate.md` 並依其把關，按識別碼升冪發布各批問題；gate 未通過時發完當前可見批次即停止，不進入 Phase 4。
5. THINK 若 comment 發布部分成功或結果不明，重新載入全部 comments 比對已持久化 `OQ-###`，只補發仍缺的問題，不盲目重發已存在批次。
6. READ 再次執行本 skill 時，重新載入全部 comments，把新 PM 回覆併入全域 OQ registry；假設被反對時重新審計受影響需求、草稿與核准範圍，只有新浮現或仍歧義的問題回到 step 2。權威來源確認完成後回到 Phase 1，不直接跳至拆票。

**完成條件**：zero open blocking product decision；每個 in-scope 行為有可測結果，必要時由已記入 registry 的暫定假設補足，且已通過所載入 gate。

## Phase 4 -- 盤點系統介面並標記影響

1. READ 讀取 `rules/system-interface-inventory-and-impact.md`，從需求部位盤點本次涉及的系統介面，並標記每個介面對既有系統的 `ADD` / `MODIFY` / `DELETE` / `NOOP` 影響。
2. THINK 依已載入判準，用影響標記與 codebase 事實找出共用 contract、migration 與 shared state 等 sequential dependency 候選。

**完成條件**：每個盤點出的介面都能對回需求原文依據；每個 `MODIFY` / `DELETE` 都指出受影響對象與相依候選。

## Phase 5 -- 草擬工程票並取得主管核准

1. READ 讀取 `rules/ticket-ordering-and-wave-grouping.md` 與 `rules/ticket-content-and-estimation.md`。
2. THINK 依已載入判準切出 vertical slice 與必要前置票，評估 size、確認真實 blocking edge、分組 Wave 並填寫 `理想工程工時`；將 registry 中的暫定假設對應到受影響票。
3. READ 讀取 `templates/notion-task-ticket.md` 與 `templates/notion-task-ticket.example.md`，依骨架複製結構、參考範例填寫每張票內容。
4. WRITE 以具有固定草稿識別碼的編號 draft，連同 Wave 分組、安排理由與 blocking graph 呈給主管；依回饋調整直到核准，保留核准稿與識別碼供發布恢復。內容、邊界或相依若實質改變，重新取得主管核准。

**完成條件**：主管明確核准每張票的內容、邊界與 blocking edge，核准稿與草稿識別碼可取得；每個來源需求與 Feature AC 至少對應一票，受影響票已承接 assumed OQ；每票有初始估時、Feature AC 參照、Prototype URL（feature 影響頁面有列出時）與空白 Tech Spec。

## Phase 6 -- 發布到 Notion

1. READ 讀取 feature database 與 task database schema，確認 task→feature relation、草稿識別碼的記錄位置與原核准稿；必要 schema 或核准稿不可得時暫停，不使用 parent/sub-item 取代 relation。
2. WRITE 先讀取 `rules/notion-writing-quality-gate.md` 並依其把關，按相依順序逐張比對既有 task；依已載入發布判準，只建立缺漏票或補齊可確認的未完成發布操作，設定 feature relation 與 blockers。
3. READ create 結果不明時先重新查證，不盲目重送；發布後驗證每個核准 draft 的唯一 task 對應、內容與關聯，無法判定時停止回報。
4. WRITE 回報草稿識別碼對應的建票連結、blocking graph 與初始 frontier 後停止。

**完成條件**：每張核准 draft 均已對應並驗證一張 task，經 task→feature relation 指回 feature、已宣告 blockers、含 Tech Spec 區塊並出現在 reciprocal relation 中；新建票使用空白模板，恢復時保留工程師既有內容；有重複或無法唯一對應時不得宣告完成。

## 失敗邊界

未指定 primary spec 或權威來源不明時，依 Phase 1 的釐清出口處理。已知來源的 Notion 存取失敗、task database、task→feature relation 或其他必要 schema 不可得時，回報確切限制並停在當下 phase。缺少原核准稿、發現重複 task 或無法判定發布對應時停止，不猜測、不重建、不自行合併或刪除；恢復時保留既有 `OQ-###` 與核准草稿識別碼。
