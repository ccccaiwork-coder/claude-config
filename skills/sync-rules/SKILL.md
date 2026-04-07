---
name: sync-rules
description: 當使用者說「同步規則」時，從 GitHub pull 最新規則並確認 symlink 正確。
---

# 從 GitHub 同步規則

## 何時觸發
- 使用者說「同步規則」、「下載規則」、「pull 規則」、「更新本機規則」

## 執行步驟

### 情況 A：~/.claude-config 已存在（舊電腦或已設定過）
1. 執行 `git -C ~/.claude-config pull`
2. 回覆 pull 結果，最後輸出 `Done`

### 情況 B：~/.claude-config 不存在（新電腦首次設定）
1. 執行以下指令完成完整安裝：
   ```
   git clone https://github.com/ccccaiwork-coder/claude-config.git ~/.claude-config
   mkdir -p ~/.claude
   ln -sf ~/.claude-config/CLAUDE.md ~/.claude/CLAUDE.md
   ln -sf ~/.claude-config/rules ~/.claude/rules
   ln -sf ~/.claude-config/skills ~/.claude/skills
   ```
2. 確認 symlink 建立成功：`ls -la ~/.claude/CLAUDE.md ~/.claude/rules ~/.claude/skills`
3. 回覆安裝結果，最後輸出 `Done`
