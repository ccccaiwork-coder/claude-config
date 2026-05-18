---
name: obsidian
description: "Obsidian vault operations for L2 Obsidian workshop learners. Use when the user says /obsidian, 寫日誌, 記進度, 記日記, log progress, daily note, 把這個對話記進去, 幫我記今天的進度, 讀 pulse, search vault, or needs to write/read/search their Obsidian vault via the official Obsidian CLI."
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Obsidian Skill（L2 Workshop 精簡版）

這是 L2 Obsidian 課程為學員精簡過的 skill，讓 Claude 用**官方 Obsidian CLI**（Obsidian v1.12.4+ 內建）讀寫學員的 vault。學員只要說「幫我記今天的進度」之類的話，Claude 就會自動整理對話重點寫進 daily note。

## 前置條件

- **Obsidian Desktop v1.12.4 或更新版本**（官方 CLI 是 2026-02-27 釋出）
- Obsidian 必須正在執行（skill 透過 IPC 和 Obsidian 溝通）
- 學員有一個 Obsidian vault（通常在 `~/obsidian-workshop`）

## CLI binary 路徑

**IMPORTANT**：skill shell 不 source `~/.zshrc`，每次呼叫 Obsidian CLI 要先加 PATH：

```bash
export PATH="$PATH:/Applications/Obsidian.app/Contents/MacOS"
```

**正確語法**：

```bash
obsidian vault=<vault-name> <command> [key=value ...]
```

## Vault 自動偵測（不寫死）

**絕對不要寫死 vault 名稱**。每次執行前用下列順序偵測：

1. 如果 `pwd` 是 `~/obsidian-workshop` 或其子資料夾 → `vault=obsidian-workshop`
2. 如果 `pwd` 有 `.obsidian/` 子資料夾 → 取 `basename $(pwd)` 當 vault 名
3. 如果偵測到 `~/obsidian-workshop/.obsidian/` 存在 → `vault=obsidian-workshop`
4. 以上都不符 → 先執行 `obsidian list` 看有哪些 vault，然後問學員：「我看到你有這些 vault：[list]，要對哪個操作？」

**絕不要假設 vault 叫 knowledge-base 或其他名字**。

---

## Subcommands（從 $ARGUMENTS 解析）

### `/obsidian log <what-happened>`

**這是學員最常用的指令**。把對話重點整理後寫進今天的 daily note。

**三段式結構**：每個 daily note 用以下固定 section：

```markdown
# YYYY-MM-DD

## 完成事項
- HH:MM — 做了什麼（每條 1-2 行，簡明扼要）

## 決定
- HH:MM — 決定了什麼、為什麼（非 actionable）

## 卡點 / 疑問
- HH:MM — 哪裡卡住、哪裡不確定（可寫詳細）
```

**寫入流程**：

1. **偵測 vault**（見上面的自動偵測邏輯）
2. **取今天日期**：`TODAY=$(date '+%Y-%m-%d')`
3. **檢查 daily note 是否存在**：
   ```bash
   obsidian vault=<vault> read path="daily/$TODAY.md" 2>/dev/null
   ```
4. **若檔案不存在或缺少三段 section header**，先用 Write 工具建立完整結構
5. **分析對話內容**：讀取當前對話的最近訊息，萃取以下三類內容：
   - **完成事項**：學員做了什麼（建了哪些檔案、點了什麼 wikilink、列了什麼生活線）
   - **決定**：學員做了什麼選擇（例如「本週最重要三件事」）
   - **卡點**：學員在哪裡卡住或表示不確定
6. **用 Edit 工具**插入到對應 section 的末尾（不用 append 到檔尾，以免破壞 section 結構）
7. **自動加 wikilink**：如果內容提到 vault 裡存在的筆記名稱，自動包成 `[[wikilink]]`：
   - 提到「生活線 X」或 pulse 檔案名 → 查 `pulse/` 資料夾確認後用 `[[pulse/X]]`
   - 提到「my-pulse」或「本週狀態」→ 用 `[[my-pulse]]`
   - 提到專案名 → 查 `projects/` 資料夾確認後用 `[[projects/Y]]`
8. **每條加時間戳**：`- HH:MM — 內容`
9. **不是所有 log 都有三類內容** — 沒有就不用硬寫（例如只有「完成事項」沒有「決定」也沒關係）

**範例流程**：

學員說：`/obsidian log 幫我記今天的進度`

Claude 要做：

```bash
# Step 1: 偵測 vault
cd ~/obsidian-workshop  # 或從 pwd 推斷
VAULT="obsidian-workshop"
TODAY=$(date '+%Y-%m-%d')
TIME=$(date '+%H:%M')

# Step 2: 檢查 daily note
export PATH="$PATH:/Applications/Obsidian.app/Contents/MacOS"
obsidian vault=$VAULT read path="daily/$TODAY.md" 2>/dev/null

# Step 3: 若缺 section header，Write 建立完整結構
# Step 4: 萃取對話重點，Edit 插入到對應 section
# Step 5: 加 wikilink（查 pulse/ 資料夾確認存在後才加）
# Step 6: 回報學員
```

回報訊息範例：

> 「我把剛才 Stage 3 的進度寫進 `daily/2026-04-11.md` 了：
> - **完成事項**：建了 5 條生活線 [[pulse/客戶-藍霧科技]] [[pulse/課程業務]] ...，建了 `my-pulse.md` hub
> - **決定**：本週最重要三件事是 X、Y、Z
>
> 打開 Obsidian 看看！」

---

### `/obsidian daily [content]`

讀取或寫入今天的 daily note。

```bash
VAULT="obsidian-workshop"  # 或自動偵測
TODAY=$(date '+%Y-%m-%d')
export PATH="$PATH:/Applications/Obsidian.app/Contents/MacOS"

# 讀取
obsidian vault=$VAULT read path="daily/$TODAY.md"

# 寫入（追加）
obsidian vault=$VAULT append path="daily/$TODAY.md" content="<text>"
```

### `/obsidian read <note-name-or-path>`

讀取筆記。`file=` 用 wikilink 風格（不帶副檔名），`path=` 用完整相對路徑。

```bash
obsidian vault=obsidian-workshop read file="my-pulse"
obsidian vault=obsidian-workshop read path="pulse/客戶-藍霧科技.md"
```

### `/obsidian append <note-name> <content>`

在指定筆記末尾追加內容。

```bash
obsidian vault=obsidian-workshop append file="my-pulse" content="<text>"
# 多行用 \n
obsidian vault=obsidian-workshop append path="pulse/客戶-藍霧科技.md" content="\n## 2026-04-11 更新\n完成 Phase 1"
```

### `/obsidian search <query>`

全文搜尋 vault。

```bash
obsidian vault=obsidian-workshop search query="<query>" format=json
# 顯示 match context
obsidian vault=obsidian-workshop search:context query="<query>"
# 限定在特定資料夾
obsidian vault=obsidian-workshop search query="<query>" path="pulse"
```

### `/obsidian backlinks <note>`

看指定筆記被誰連結。

```bash
obsidian vault=obsidian-workshop backlinks file="my-pulse"
```

### `/obsidian pulse`

讀取學員的 `my-pulse.md`（Mission Control hub）。

```bash
obsidian vault=obsidian-workshop read file="my-pulse"
```

### `/obsidian tasks`

列出 vault 裡所有未完成的待辦。

```bash
obsidian vault=obsidian-workshop tasks todo format=json
```

---

## 預期的 Vault 結構（L2 Workshop）

```
~/obsidian-workshop/
├── CLAUDE.md                # Vault 規則檔
├── my-pulse.md              # 學員的 Mission Control hub
├── pulse/                   # 生活線檔案
│   ├── 生活線1.md
│   ├── 生活線2.md
│   └── ...
├── daily/                   # 每日筆記（YYYY-MM-DD.md 格式）
│   └── 2026-04-11.md
└── projects/                # 專案主檔（選用）
    └── ...
```

## Key Rules

1. **永遠自動偵測 vault**，不要寫死 `vault=knowledge-base` 或其他特定名稱
2. **每次呼叫 obsidian CLI 前加 PATH**（`export PATH="$PATH:/Applications/Obsidian.app/Contents/MacOS"`）
3. `file=` 用 wikilink 風格（不帶副檔名）
4. `path=` 用完整相對路徑（含 `.md`）
5. content 裡的換行用 `\n`
6. **絕不覆蓋**既有筆記 — 用 `append` 或 Edit 工具做目標修改
7. **寫 daily note 前先檢查三段 section header**，缺的話先用 Write 建立結構
8. **加 wikilink 前先確認目標檔案存在**（Glob 或 Read），不要建虛擬連結
9. **不是所有 log 都有三類內容** — 沒有的 section 就不寫，不要硬塞

## When to Use This Skill

| 學員說 | 用哪個指令 |
|---|---|
| 幫我記今天的進度 | `/obsidian log <萃取對話重點>` |
| 把這個對話記進去 | `/obsidian log <萃取對話重點>` |
| 讀我的 pulse | `/obsidian pulse` |
| 今天的日記 | `/obsidian daily` |
| 搜尋 vault 裡有沒有 X | `/obsidian search X` |
| X 檔案被誰連結？ | `/obsidian backlinks X` |
| 列出所有待辦 | `/obsidian tasks` |

---

## Troubleshooting

| 問題 | 解法 |
|---|---|
| `obsidian: command not found` | Obsidian 版本 < 1.12.4，請學員更新到最新版 |
| `vault not found` | 確認 Obsidian 正在執行，且 vault 已在 Obsidian 裡打開過一次 |
| Daily note 格式被破壞 | 用 Read 讀出來，手動用 Edit 修復三段 section |
| 找不到 wikilink 目標 | 用 Glob 掃 `pulse/` 和 `projects/` 確認檔案真的存在再加 wikilink |
