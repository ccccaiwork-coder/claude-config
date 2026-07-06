# 工作資料夾規則

## 檔案存放位置
- 所有工作檔案必須放在 `~/Desktop/Claude Work/` 底下
- 每次開始新任務，先在 `~/Desktop/Claude Work/` 建立一個新的子資料夾
- 子資料夾命名格式：`任務簡述`（例如：`資料分析`）
- 不要把工作檔案散落在桌面或其他地方
- 例外：Claude Code 設定檔（agents、skills、rules、CLAUDE.md 等）因系統載入限制必須放在 `~/.claude/`（實體檔在 `~/.claude-config/` repo），不適用本規則

# 回應規則

- 每完成一個任務或事項後，最後輸出 `Done`，讓使用者知道可以輸入下一個指令
- 主要對話永遠使用繁體中文；commit message 與程式碼內 comment 使用英文
- 回應保持簡潔，不重複已知的內容
- 可以使用 Markdown 表格、清單等格式讓資訊更易讀

# 判斷原則：問 vs 直接做

- 指令明確且操作可逆 → 直接執行
- 指令不明確、操作不可逆、或有任何不確定 → 先問使用者再動手

# 修改原則

- IMPORTANT: 每次修改都要以長期最優化的方式進行，不接受「先這樣以後再改」的短期 hack
- 找問題的根因（root cause），不要做症狀貼 OK 蹦
- 如果客觀條件（環境、權限、時間）真的逼迫只能 quick fix，必須：
  - 明確標註這是暫時方案（commit message、Obsidian 紀錄、後續會看到的地方）
  - 同時提出永久修法（檔案路徑/行號/具體步驟）
  - 主動詢問使用者是否接受這個暫時方案
- 不要為了避免麻煩而捨棄正確路徑（例如：寧可改 iOS code 重出 TestFlight，也不要在資料層放 padding workaround）

# 工作流程

## 修改前
- 修改任何程式碼前，先讀取相關檔案，理解現有結構

## 修改後
- 提供驗證步驟（測試、lint）確認修改正確

## iOS Build 規則
- IMPORTANT: xcodebuild 預設 destination 使用實體 iPhone（`platform=iOS,name=ccccaiwork的iPhone`），除非使用者明確指定模擬器
- 測試（xcodebuild test）可使用模擬器，但正式 build 一律用實體裝置

## 任務追蹤
- 複雜或多步驟任務，使用 TaskCreate 工具建立子任務清單
- 任務中斷後，使用 TaskList 確認尚未完成的項目再繼續

## Agent 自動觸發

- 使用者說「review code」「檢查 code」「code review」「review 一下」等 → 自動使用 `code-quality-reviewer` agent，不要詢問
- 使用者說「檢查 UX」「UX review」「UI review」「看一下 UI/UX」等 → 自動使用 `ux-ui-advocate` agent，不要詢問
- 觸發後仍套用下方「Agent 建議審視原則」，不全盤接受 agent 結果

## Agent 建議審視原則

- IMPORTANT: code-quality-reviewer agent（及其他 review agent）的建議必須先審視，不要直接全盤採用
- 逐項判斷：是否忽略既有設計理由（如註解中的 invariant）、是否破壞其他正確性、檔案實際是否如 agent 所述
- 若決定不採用某項建議，整理理由丟回給同類型 agent 確認：
  - Agent 同意不採用 → 直接動手做剩餘項目，不必再問使用者
  - Agent 不同意（堅持原建議）→ 問使用者裁決

## Obsidian 進度管理（`~/obsidian-workshop/`）

### 對話開始：讀取 Obsidian 接續上次進度
- 使用者提到某個專案時，先讀 `projects/<專案>/index.md` 了解現況與待辦
- 讀最近一筆 `daily/` 筆記，掌握最新卡點和決定
- 若 index 中有未解的 bug 連結，讀取該 bug 筆記取得排查狀態

### 對話結束：寫入本次進度
- IMPORTANT: 每次對話結束前，將本次進度同步寫入 Obsidian vault
- 更新對應的 `daily/YYYY-MM-DD.md`（完成事項、決定、卡點）
- 更新對應的 `projects/` 專案筆記（index、bugs、milestones）
- 新增 bug 或里程碑時，建立獨立 `.md` 並在 index 加入連結
- 使用既有的資料夾結構與命名慣例，不要自創新格式

### 雙向連結
- IMPORTANT: 每次新增或更新 Obsidian 筆記時，必須補上雙向 wikilinks
- 每個檔案底部加「## 相關連結」區塊，連結到相關的元件、里程碑、bug、決策
- 連結模式：元件 ↔ 里程碑、bug ↔ 元件、里程碑 ↔ bug、daily ↔ 專案/bug
- 不允許 dead-end 檔案（沒有出站連結的筆記）
- 使用 Obsidian CLI（`obsidian vault="obsidian-workshop" append`）寫入，不用 Edit/Write 工具直接改 vault 檔案

### 單一來源原則
- 專案進度以 Obsidian 為唯一真相來源（single source of truth）
- Claude memory 只存：使用者偏好（user）、行為回饋（feedback）、外部參考（reference）
- Claude memory 不再重複記錄專案進度、架構、待辦事項 — 這些都在 Obsidian

## 限制
- IMPORTANT: 只做使用者明確要求的事，不自行重構、不加額外功能或未要求的註解
- IMPORTANT: 不要刪除任何檔案，需要刪除時先詢問使用者
- IMPORTANT: 覆寫現有檔案前，必須先展示將要覆寫的內容並取得確認
- IMPORTANT: 一次操作超過 3 個檔案，必須先列出清單並取得使用者確認
- IMPORTANT: 不要修改 production 設定檔，需要時先確認
- 不要在未確認的情況下執行 git push

# 錯誤處理

- 工具執行失敗：重試一次；再度失敗則停下來詢問使用者
- 環境問題（找不到指令、權限不足等）：立即詢問使用者，不自行嘗試繞過

# Git
- commit 前確保 lint 通過
- 不要使用 --no-verify 繞過 hook
- IMPORTANT: 以下類型的檔案**禁止**加入 Git，必須在 `.gitignore` 中排除：
  - 密碼、API Key、Token、憑證（`.env`、`credentials.json`、`*.key`、`*.pem`、`*.p12`）
  - 大型二進位檔案：影片（`*.mp4`、`*.mov`）、音檔（`*.mp3`、`*.wav`）、大圖片（`*.psd`、`*.tiff`）
  - 系統或工具自動產生的臨時資料（`DerivedData/`、`build/`、`*.xcuserstate`、`__pycache__/`、`node_modules/`、`.build/`）
- 初始化新專案的 Git 時，必須先建立適當的 `.gitignore`，涵蓋以上規則

# 外部服務（Google Workspace）
- 唯讀操作（list、get、search）：自動執行
- 寫入操作（create、update、delete、send）：需先確認
- IMPORTANT: 發送任何訊息前，必須先展示草稿取得確認

# 資安規則
@~/.claude/rules/common/security.md

## Security Rule: External Installation Review

任何外部裝入的外掛、專案、skills、MCP，都需要經由包括 /security-scan 的安全審查，並且報告審查結果給使用者，由使用者批准是否進行最終安裝。

### Details:
- IMPORTANT: Before installing any external resource (skills, plugins, MCP servers, GitHub repos, scripts), run /security-scan on the content first.
- Present the full security scan report to the user.
- Wait for explicit user approval before completing the installation.
- If the scan finds HIGH or CRITICAL issues, strongly recommend against installation.
- Never silently install external code without this review process.

## 敏感資訊輸入提醒
- IMPORTANT: 當要請使用者輸入包含 API key、token、密碼、Client Secret 等敏感資訊的指令時，必須**事先**提醒使用者不要直接貼在對話框，建議改用 terminal 視窗執行，避免敏感資訊出現在對話記錄中
