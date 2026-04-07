---
name: update-rules
description: 當使用者說「更新規則」時，自動將 ~/.claude-config 的變更 commit 並 push 到 GitHub。
---

# 更新規則到 GitHub

## 何時觸發
- 使用者說「更新規則」、「push 規則」、「同步規則到 github」

## 執行步驟

1. 執行 `git -C ~/.claude-config status --short` 確認有無變更
2. 若無變更，回覆「規則沒有變更，無需更新」，結束
3. 若有變更，依序執行：
   ```
   git -C ~/.claude-config add .
   git -C ~/.claude-config commit -m "Update Claude rules"
   git -C ~/.claude-config push
   ```
4. 回覆 push 結果，最後輸出 `Done`
