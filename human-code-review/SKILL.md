---
name: human-code-review
description: 以 ce-code-review 為審查引擎、speak-human-tw 潤飾輸出，進行 PR、commit 或 branch diff 的程式碼審查。支援關聯 Ticket 規格比對、PR 既有留言去重、Local 髒目錄 worktree 隔離審查，並支援選取發布 GitHub inline comments。
---

# Human Code Review

協調程式碼審查與 GitHub PR inline 留言發布。核心依賴：
- **審查引擎**：**委派 `ce-code-review`**（`~/.pi/agent/git/github.com/EveryInc/compound-engineering-plugin/skills/ce-code-review/SKILL.md`）執行多 persona 審查，取得帶 severity（P0–P3）、confidence、file:line、suggested_fix 的 findings。本 skill 不自訂審查細則；多 persona 派發需要 `pi-subagents` 套件，未安裝時退為依其 severity rubric 自行靜態審查。判定採用其 merge bar：能實質提升程式碼健康度即 Approve，不以個人寫法風格卡關；對結構退化與安全漏洞嚴格把關。
- **語氣與自然表達**：遵循 [`speak-human-tw`](../speak-human-tw/SKILL.md)（台灣繁中習慣、去 AI 味、明確點出 Why 與 How、保留英文技術術語；跳過文章潤稿確認清單）。
- **審查執行原則（禁止跑測試與 Linter）**：審查過程**嚴禁執行專案建置（build/check）、測試套件（test suites）或 Linter / Formatter（clippy, eslint, typecheck 等）**。這些檢查皆由 CI 自動化流程把關。審查一律採用**靜態程式碼閱讀、Diff 分析、架構與 Domain Model 比對**，並透過「閱讀測試程式碼本身」來審查測試意圖與邊界覆蓋，不重複執行任何本機建置與測試指令。

---

## 執行流程

### Step 1：取得 Diff、關聯 Ticket、PR 既有留言與工作區準備
依使用者指定的目標取得 diff 與上下文：
- **PR**：
  1. **讀取 PR 資訊與關聯 Ticket**：
     - 使用 `gh pr view <PR> --json title,body,headRefName` 取得 PR 標題、說明與 branch。
     - **檢查是否附有 Ticket**：若 PR 內文、標題或 branch 包含關聯 Ticket / Issue 連結或 ID（例如 Notion、Linear、Jira、GitHub Issue 或本機 task plan），**必須優先讀取該 Ticket / Spec 需求與驗收條件**，作為 Correctness 面向的核心比對依據（逐項核對驗收條件）。
  2. **取得 Diff 與既有討論**：
     - 抓取 Diff：`gh pr diff <PR>`。
     - 抓取既有討論：`gh api repos/{owner}/{repo}/pulls/<PR>/comments`（供去重比對）。
  3. **檢查 Local 工作目錄**（若需檢視完整專案程式碼或跨檔案參照）：
     - 執行 `git status --porcelain` 檢查當前目錄狀態。
     - **若 Local 有未 commit 的改動（dirty）**：為避免污染或影響進行中的工作，禁止在當前目錄直接 checkout 或 stash，改以 `git worktree` 建立隔離環境進行檔案查閱：
       ```bash
       # 抓取 PR 分支並建立獨立 worktree
       git fetch origin pull/<PR>/head:pr-<PR>-review
       git worktree add /tmp/pr-<PR>-review pr-<PR>-review
       ```
     - 後續讀取專案檔案與確認上下文皆在 `/tmp/pr-<PR>-review` 中以靜態閱讀進行（**嚴禁在 worktree 執行 build/test/lint**）。
     - **審查完畢後清理**：
       ```bash
       git worktree remove --force /tmp/pr-<PR>-review
       git branch -D pr-<PR>-review
       ```
- **Commit / Branch / Working Tree**：`git show <COMMIT>`、`git diff origin/main...<BRANCH>` 或 `git diff HEAD`。

### Step 2：執行審查、去重並以人話輸出
1. **委派 `ce-code-review` 產出 findings（純靜態分析）**：
   - 將 Step 1 的審查目標（PR 編號、commit、branch 或 diff）交給 `ce-code-review` 執行，並把 Ticket 規格與驗收條件一併餵入作為審查上下文。取得的 findings 包含 severity（P0–P3）、confidence、file:line、suggested_fix。
   - **Spec 缺口補審**：ce-code-review 審的是 diff 品質；另行核對 Ticket 驗收條件，補上「規格要求但未實作」的缺口。
   - **嚴重度映射**：P0→`[Critical]`、P1→`[Required]`、P2→`[Consider]`、P3→`[Nit]`、`advisory` 類或非問題觀察→`[FYI]`。
   - **完全靜態閱讀，不跑指令**：審查引擎與本 skill 的補審皆禁止執行 `cargo test`、`cargo check`、`cargo clippy`、`pnpm test`、`pnpm lint`、`tsc` 等任何測試或檢查指令，完全仰賴 code reading 與靜態邏輯推理；測試覆蓋以閱讀測試檔案內容判斷，不執行測試。
   - **判定標準**：能實質提升程式碼健康度即給予 Approve，不以個人寫法風格卡關；對結構退化與安全漏洞嚴格把關，不接受「之後再修」。
2. **去重與既有討論過濾（僅 PR，嚴格執行）**：
   - **檢查核心：有沒有被提過、提的內容完不完整**（不用管作者後續是否有提交修改，原留言者會自行負責追蹤）：
     - **已提過且內容完整（一律略過不列）**：只要既有留言已明確點出該問題的核心事實或實質風險，即視為完整提出，**一律直接略過不列**。嚴禁以「補充重構程式碼、補充 ADR/文件說明、換句話說、或微調觀點」為由重複提出。
     - **唯一例外（既有留言內容嚴重不完整或方向偏誤）**：僅在既有留言「未點出真正的根本原因、判斷方向有明顯錯誤或漏掉致命關鍵事實，導致原留言無法正確引導修復」時，才允許提出以補足完整脈絡（並在標題註記「（承既有討論）」）。只要原留言方向正確且已點出問題本質，就不符合例外，一律略過。
3. **自然語氣輸出**：依照 `speak-human-tw` 規範產出編號清單（#1, #2...）與總結判定：

```markdown
## Code Review 結果

### 審查發現
#### #1 [Critical] <核心問題簡述>
- **位置**：`path/to/file.ext:行號`
- **問題**：<具體風險與原因（Why）>
- **建議**：<具體解法或重構方向（How），附程式碼片段>

---

### 審查判定
- [ ] Approve / [x] Request Changes / [ ] Comment
```

4. **預覽即發布稿（Single Source of Truth）**：在顯示 Review 前，為每一項 finding 產生唯一的 canonical Markdown `publish_body`。
   - 使用者在 Step 2 看到的標題、嚴重度、位置、問題與建議，就是完整的 `publish_body`；預覽與後續 GitHub comment 共用同一份文字。
   - 編號只用於 Step 3 選取項目，不必發布到 GitHub；除此之外，發布內容逐字重用 `publish_body`，包含段落、清單、程式碼與 suggestion block。
   - GitHub 所需的 `path`、`line`、`side`、`start_line` 是定位 metadata，不改動 `publish_body`。
   - 若發布前發現內容需要修正，先把修正版重新顯示給使用者確認，再發布修正版。

### Step 3：確認發布項目（僅 PR）
審查輸出後詢問使用者：
> 以上 Review 共有 **N** 項，請問有哪些需要發布到 PR 上？（例如：`發布 1, 2` / `全部發布` / `不發布`）

### Step 4：一次發布 Inline-only Review
使用者選定編號後，先建立 pending review，再以 `COMMENT` 一次送出所有 inline comments：

1. **固定發布模式**：只使用 Review API 的 `PENDING → COMMENT` 兩段式流程。建立與送出時 `body` 都設為空字串；不使用 `REQUEST_CHANGES`，不發布 Review summary body 或一般 PR comment。
2. **逐字重用已確認內容**：將每個選定 finding 的 canonical `publish_body` 原樣放入 `comments[].body`，包含標題、嚴重度、問題、建議、段落與格式。
3. **Inline Suggestion 格式**：若建議為明確的程式碼替換，在 Step 2 產生 `publish_body` 時就使用 ` ```suggestion ` 語法，讓使用者預覽實際會發布的內容。
4. **先完成所有定位再建立 review**：確認最新 `HEAD_SHA`，並為每個選定 finding 找到有效的 `path`、`line`、`side`（多行留言再加 `start_line`、`start_side`）。
   - 問題行在 diff 內時，直接掛在該行。
   - 問題行不在 diff 內或屬於全域／架構問題時，掛在同一變更中最直接引入該問題的 diff 行；canonical `publish_body` 保留真實問題位置與完整說明。
   - 找不到合理的 diff 行時，先回報該 finding 無法 inline 並請使用者決定是否略過；維持 inline-only，不改發 Review body 或一般 PR comment。
5. **Parity Gate**：建立 pending review 前，逐項比對 `comments[].body` 與使用者確認的 canonical `publish_body`，必須逐字相同；確認頂層 `body` 為 `""` 且 payload 沒有 `event`。
6. **一次送出**：建立 pending review 成功後，以 `event: "COMMENT"`、`body: ""` 提交該 review。所有 inline comments 隨同一個 Review 一次出現，作者不會分批收到。

```bash
HEAD_SHA=$(gh pr view <PR> --json headRefOid -q .headRefOid)

REVIEW_ID=$(gh api -X POST repos/{owner}/{repo}/pulls/<PR>/reviews --input - --jq .id <<EOF
{
  "commit_id": "$HEAD_SHA",
  "body": "",
  "comments": [
    {
      "path": "path/to/file.ext",
      "line": 102,
      "side": "RIGHT",
      "body": "<Step 2 已顯示並經使用者選定的 canonical publish_body，逐字重用>"
    }
  ]
}
EOF
)

gh api -X POST repos/{owner}/{repo}/pulls/<PR>/reviews/$REVIEW_ID/events \
  -f event=COMMENT \
  -f body=
```

發布完成後回報同一個 Review 與其中每則 inline comment 的連結。
