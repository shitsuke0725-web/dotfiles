---
name: session-wrap-up
description: |
  收工技能：工作階段結束時的標準收尾流程。自動檢查專案狀態、整理工作成果、確認 Git 狀態、檢查 AGENTS.md 更新需求，並將今日新增的檔案（技能、PDF、圖表等）同步到 Obsidian 或 dotfiles。
  觸發詞：收工、下班、結束工作、儲存進度、整理今天的工作、工作階段結束、session wrap up、wrap up、結束會話、存檔。
  適用場景：任何工作階段結束前，需要確保所有成果已被記錄、檔案已備份、知識已沉澱到 Obsidian 第二大腦。
license: MIT
metadata:
  version: "1.0.0"
  category: productivity
  author: shitsuke0725-web
---

# 收工 · Session Wrap-Up

工作階段結束時的標準收尾流程。確保所有成果被記錄、檔案已備份、知識已沉澱。

## 使用前提

此技能在「任何工作階段結束時」自動啟用，確保不遺漏重要成果。

**適用場景**：
- 一段對話/工作即將結束
- 需要確認今日成果已保存
- 需要將工作記錄到 Obsidian
- 需要確認 Git 倉庫狀態
- 需要更新 AGENTS.md

**輸出**：
- 工作階段摘要
- Git 狀態報告
- 待處理項目清單
- Obsidian 同步確認

## 收工流程

### Phase 1: 專案狀態檢查

#### 1.1 檢查目前工作目錄

```powershell
# 顯示目前位置與專案資訊
Get-Location
Get-ChildItem -Path . -Name
```

#### 1.2 檢查 Git 狀態

**必做檢查**：
```powershell
# 檢查是否有未提交的變更
git status --short

# 檢查最近的提交歷史
git log --oneline -5

# 檢查是否有未追蹤的重要檔案
git ls-files --others --exclude-standard
```

**判斷邏輯**：
- 若有未提交的變更 → 詢問是否提交
- 若有未追蹤的技能檔 → 建議加入版本控制
- 若有衝突標記 → 標記為需手動處理

#### 1.3 檢查今日新增/修改的檔案

```powershell
# 尋找今日修改的檔案（跨專案）
$Today = Get-Date -Format "yyyy-MM-dd"
Get-ChildItem -Path "C:\Users\<user>" -Recurse -File |
  Where-Object { $_.LastWriteTime.ToString("yyyy-MM-dd") -eq $Today } |
  Select-Object FullName, LastWriteTime, Length |
  Sort-Object LastWriteTime -Descending
```

**重點關注路徑**：
- `~/.config/opencode/skills/` ← 新技能檔案
- `~/Documents/NotebookLM/` ← NotebookLM 產出
- `~/OneDrive/Desktop/` ← 桌面工作檔案
- `~/.config/opencode/agents/` ← AGENTS.md 相關

### Phase 2: AGENTS.md 檢查

#### 2.1 檢查專案 AGENTS.md

```powershell
# 檢查專案根目錄是否有 AGENTS.md
Test-Path "./AGENTS.md"

# 檢查最後更新時間
(Get-Item "./AGENTS.md").LastWriteTime
```

#### 2.2 檢查是否需要更新

**更新觸發條件**：
- [ ] 本階段新增了重要決策或架構變更
- [ ] 發現了新的專案約束或規範
- [ ] 建立了新的工作流程
- [ ] 修改了專案結構
- [ ] 新增了依賴或工具

**更新內容格式**：
```markdown
## YYYY-MM-DD Session Summary

### 完成事項
- [事項1]
- [事項2]

### 重要決策
- [決策1]：原因與影響

### 待處理
- [ ] [待辦1]
- [ ] [待辦2]

### 新發現
- [發現1]
```

### Phase 3: 技能檔案同步

#### 3.1 檢查新技能

```powershell
# 列出今日新增的技能
Get-ChildItem -Path "C:\Users\<user>\.config\opencode\skills" -Directory |
  Where-Object { (Get-Item $_.FullName).CreationTime.ToString("yyyy-MM-dd") -eq (Get-Date -Format "yyyy-MM-dd") }
```

#### 3.2 備份技能到 Git

若使用者有 dotfiles 或 skills 專用倉庫：
```powershell
# 進入 skills 目錄
cd ~/.config/opencode/skills

# 檢查狀態
git status

# 若有新增技能，提交並推送
git add .
git commit -m "feat(skills): add [skill-name] - [description]"
git push
```

### Phase 4: Obsidian 同步

#### 4.1 建立工作階段筆記

**筆記路徑**：`知識庫/工作紀錄/YYYY-MM-DD_工作階段.md`

**筆記模板**：
```markdown
---
date: 2026-05-10
tags: [工作紀錄, opencode]
project: [專案名稱]
---

# YYYY-MM-DD 工作階段

## 專案
- [專案名稱]

## 完成事項
- [ ] [事項1]
- [ ] [事項2]
- [ ] [事項3]

## 新增檔案
| 檔案 | 位置 | 說明 |
|------|------|------|
| [檔名] | [路徑] | [說明] |

## 重要決策
- [決策1]

## 待處理
- [ ] [待辦1]

## 相關連結
- [連結1]
- [連結2]
```

#### 4.2 同步教學素材

若今日有 NotebookLM 產出：
- 將 dashboard.html 截圖存入 `教學素材/`
- 將心智圖 JSON 轉為筆記存入 `知識庫/`
- 將測驗資料摘要記錄

### Phase 5: 工作階段摘要

#### 5.1 生成摘要報告

**輸出格式**：
```
═══ 收工報告 ═══

📁 專案：[專案名稱]
⏱️  時間：[開始時間] → [結束時間]（共 X 小時）

✅ 已完成：
  - [事項1]
  - [事項2]

📝 Git 狀態：
  - [X] 個未提交變更
  - [X] 個未追蹤檔案
  - 最後提交：[hash] [訊息]

📄 今日新增檔案：
  - [檔案1]（[大小]）
  - [檔案2]（[大小]）

⚠️  待處理：
  - [ ] [待辦1]
  - [ ] [待辦2]

🧠 知識同步：
  - Obsidian：[已同步/未同步]
  - Skills：[已備份/未備份]
  - AGENTS.md：[已更新/未更新]

═══ 建議 ═══
[根據檢查結果給出的下一步建議]
```

#### 5.2 互動確認

**詢問使用者**：
1. 「是否需要提交目前的 Git 變更？」
2. 「是否需要將今日工作記錄到 Obsidian？」
3. 「是否需要更新 AGENTS.md？」
4. 「是否有需要特別標記的重要成果？」

## 自動化檢查清單

```markdown
## 收工檢查清單

### 程式碼與檔案
- [ ] Git 變更已檢查
- [ ] 重要變更已提交
- [ ] 未追蹤檔案已審查
- [ ] 暫存檔已清理（*.tmp, *~）

### 知識管理
- [ ] AGENTS.md 已檢查更新需求
- [ ] 新技能已備份
- [ ] 工作記錄已寫入 Obsidian
- [ ] 重要連結已保存

### 專案狀態
- [ ] 待辦事項已記錄
- [ ] 明日優先事項已標記
- [ ] 阻礙/問題已記錄

### 個人
- [ ] 今日學習已記錄
- [ ] 需要回顧的決策已標記
```

## 觸發詞

當使用者說出以下詞句時，自動啟用此技能：

- 「收工」
- 「下班」
- 「結束工作」
- 「儲存進度」
- 「整理今天的工作」
- 「工作階段結束」
- 「session wrap up」
- 「wrap up」
- 「結束會話」
- 「存檔」
- 「我要關了」
- 「先這樣」
- 「今天就到這」

## 範例對話

**使用者**：「收工」

**助理**：
1. 檢查目前專案 Git 狀態
2. 列出今日新增/修改的檔案
3. 檢查 AGENTS.md 更新需求
4. 詢問是否提交變更
5. 詢問是否同步到 Obsidian
6. 生成工作階段摘要

**回覆格式**：
```
收工整備完成！

📊 今日成果：
- 建立了 2 個新技能（teaching-cockpit-builder, teaching-cockpit-publisher）
- 部署了「海洋的殺手」教學駕駛艙到 GitHub Pages
- 新增 8 個檔案到 teaching-cockpit 倉庫

📝 Git 狀態：
- gfes-auto：4 個未追蹤檔案（建議提交 .sisyphus/ 變更）
- teaching-cockpit：已部署完成

🧠 知識同步：
- 技能檔案：建議備份到 dotfiles
- Obsidian：已建立工作紀錄筆記

⚠️  明日提醒：
- 檢視 teaching-cockpit 的 GitHub Pages 是否正常運作
- 考慮將本次流程記錄到 AGENTS.md
```

## 版本歷史

- **v1.0.0** (2026-05-10)：初始版本，支援 Git 檢查、Obsidian 同步、AGENTS.md 更新提醒、技能備份
