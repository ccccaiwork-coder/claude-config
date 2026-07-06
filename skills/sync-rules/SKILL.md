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
2. 若 `~/.claude-config-private` 存在，執行 `git -C ~/.claude-config-private pull`；不存在則先 clone（見情況 B 步驟 2）
3. 執行下方「Symlink 檢查與補建」
4. 回覆 pull 結果，最後輸出 `Done`

### 情況 B：~/.claude-config 不存在（新電腦首次設定）
1. 執行以下指令完成完整安裝：
   ```
   git clone https://github.com/ccccaiwork-coder/claude-config.git ~/.claude-config
   mkdir -p ~/.claude
   ln -sf ~/.claude-config/CLAUDE.md ~/.claude/CLAUDE.md
   ln -sfn ~/.claude-config/rules ~/.claude/rules
   ln -sfn ~/.claude-config/skills ~/.claude/skills
   ```
2. clone private repo（放公司專用 skill 與行銷團隊 agents）：
   ```
   git clone https://github.com/ccccaiwork-coder/claude-config-private.git ~/.claude-config-private
   ```
   若無權限 clone，跳過並提醒使用者：行銷團隊 agents 與 patent-search skill 在此機器不可用
3. 執行下方「Symlink 檢查與補建」
4. 回覆安裝結果，最後輸出 `Done`

### Symlink 檢查與補建（冪等，情況 A/B 都要執行）
```
mkdir -p ~/.claude/agents
for f in ~/.claude-config/agents/*.md; do ln -sf "$f" ~/.claude/agents/"$(basename "$f")"; done
if [ -d ~/.claude-config-private/agents ]; then
  for f in ~/.claude-config-private/agents/*.md; do ln -sf "$f" ~/.claude/agents/"$(basename "$f")"; done
fi
if [ -d ~/.claude-config-private/skills/patent-search ]; then
  ln -sfn ~/.claude-config-private/skills/patent-search ~/.claude-config/skills/patent-search
fi
```
執行後用 `ls -la ~/.claude/agents/` 確認 symlink 都指向兩個 repo。
