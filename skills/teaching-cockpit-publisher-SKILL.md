---
name: teaching-cockpit-publisher
description: |
  發佈教學駕駛艙技能。將已製作好的教學駕駛艙網頁（dashboard.html 及相關資源）推送到 GitHub 倉庫，並啟用 GitHub Pages 公開部署。
  觸發詞：發佈教學駕駛艙、部署駕駛艙、上傳 GitHub、GitHub Pages、部署教學網頁、發佈課程網頁、推送到 GitHub、駕駛艙部署、發布 dashboard、教學網頁上線。
  適用場景：教師已完成教學駕駛艙網頁製作，需要將檔案上傳到 GitHub 並取得公開的學生訪問連結。
license: MIT
metadata:
  version: "1.0.0"
  category: education
  author: shitsuke0725-web
---

# 教學駕駛艙發佈器 · Teaching Cockpit Publisher

將已製作好的教學駕駛艙網頁及相關資源，推送到 GitHub 倉庫並啟用 GitHub Pages 公開部署。

## 使用前提

此技能專為「NotebookLM 備課包 → 教學駕駛艙 → GitHub Pages 部署」的工作流程設計。

**適用場景**：
- 教師已使用 NotebookLM 生成備課資料（簡報、音訊、測驗、心智圖等）
- 需要將分散的資源整合為單一互動網頁
- 需要公開的學生訪問連結
- 需要統一管理多個課程的駕駛艙

**不適用場景**：
- 沒有 NotebookLM 備課包的情況（需先使用 NotebookLM 生成）
- 需要動態後端（資料庫、使用者登入等）
- 純靜態內容無需互動駕駛艙

## 工作流程

### Phase 1: 備課包檢查與準備

#### 1.1 檢查 GitHub 上傳限制

**必做檢查**：
- 單一檔案大小 < 100MB（GitHub 硬性限制）
- 若 > 100MB，需要使用 Git LFS 或排除
- 建議音訊/影片檔案 < 50MB 以確保載入速度

**操作步驟**：
```powershell
# 檢查 dashboard.html 大小
(Get-Item "C:\Users\<使用者>\Documents\NotebookLM\dashboard.html").Length

# 檢查所有資源檔案
Get-ChildItem -Path "C:\Users\<使用者>\Documents\NotebookLM" -Recurse | 
  Where-Object { $_.Length -gt 100MB } | 
  Select-Object FullName, Length
```

#### 1.2 盤點引用資源

**必須確認的資源類型**：

| 資源類型 | 副檔名 | 是否上傳 | 注意事項 |
|---------|--------|---------|---------|
| 簡報 | `.pdf`, `.pptx` | ✅ 是 | PDF 優先 |
| 音訊 | `.mp4`, `.m4a` | ✅ 是 | 檔案較大，注意大小 |
| 資訊圖表 | `.png`, `.jpg` | ✅ 是 | 建議 < 10MB |
| 心智圖 | `.json` | ✅ 是 | 資料檔案 |
| 測驗資料 | `.json`, `.csv` | ✅ 是 | 資料檔案 |
| 影片 | `.mp4` | ⚠️ 視情況 | 建議放 YouTube，內嵌連結 |
| 差異化教材 | `.docx`, `.md` | ❌ 否 | 版權/個資考量 |

**dashboard.html 引用路徑檢查**：
```html
<!-- 確認所有相對路徑正確 -->
<a href="quiz.html">...</a>
<iframe src="slides/課程簡報.pdf">...</iframe>
<img src="infographics/資訊圖表.png">...</img>
<audio src="audio/課程音訊.mp4">...</audio>
```

#### 1.3 確認排除項目

**不上傳 GitHub 的項目**：
- `video/` 目錄（建議上傳 YouTube，內嵌 iframe）
- `docs/` 或 `differentiated/` 目錄（個人化教材）
- `node_modules/`, `.git/`, `.opencode/` 等系統目錄
- 任何含學生個資的檔案

### Phase 2: 建立倉庫結構

#### 2.1 命名規範

**倉庫名稱建議**：
- `teaching-cockpit`（教學駕駛艙）
- `lesson-dashboards`（課程儀表板）
- `edu-cockpit`（教育駕駛艙）

**命名原則**：
- 使用 kebab-case（短橫線連接）
- 避免中文（URL 相容性）
- 可擴展，容納多課程

#### 2.2 目錄結構

```
teaching-cockpit/
├── index.html              # 主駕駛艙（必須，GitHub Pages 入口）
├── quiz.html               # 測驗頁面
├── slides/                 # 簡報 PDF
│   └── 課程簡報.pdf
├── audio/                  # 音訊檔案
│   └── 課程音訊.mp4
├── infographics/           # 資訊圖表
│   └── 資訊圖表.png
├── mindmaps/               # 心智圖 JSON
│   └── 心智圖.json
├── quizzes/                # 測驗資料
│   └── 測驗.json
└── sheets/                 # 試算表 CSV
    └── 測驗.csv
```

**多課程擴展結構**（未來）**：
```
teaching-cockpit/
├── index.html              # 課程總覽入口
├── lesson-08/              # 第08課
│   ├── index.html
│   ├── quiz.html
│   ├── slides/
│   ├── audio/
│   └── ...
└── lesson-09/              # 第09課
    ├── index.html
    └── ...
```

#### 2.3 檔案準備

**必做操作**：
1. 建立本地工作目錄：`C:\temp\teaching-cockpit`
2. 複製 `dashboard.html` → `index.html`（GitHub Pages 需要 index.html 作為入口）
3. 複製 `quiz.html`
4. 建立子目錄並複製對應資源檔案

**篩選邏輯**：
- 只複製「當前課程」相關檔案（透過檔名關鍵字如「第08課」、「海洋的殺手」篩選）
- 排除其他課程的資源（避免混亂）

### Phase 3: GitHub 推送與部署

#### 3.1 前置需求

**必要工具**：
- Git（已安裝）
- GitHub CLI (`gh`)：`winget install --id GitHub.cli`
- Git 使用者設定：
  ```bash
  git config --global user.name "你的名稱"
  git config --global user.email "你的郵箱"
  ```

**GitHub CLI 驗證**：
```bash
gh --version        # 確認安裝
gh auth status      # 確認已登入
```

#### 3.2 初始化與提交

```powershell
cd C:\temp\teaching-cockpit
git init
git add .
git commit -m "Initial commit: 課程名稱 teaching cockpit"
```

#### 3.3 創建遠端倉庫並推送

```powershell
gh repo create teaching-cockpit --public --source=. --remote=origin --push
```

**參數說明**：
- `--public`：公開倉庫（GitHub Pages 免費版需要公開）
- `--source=.`：使用當前目錄作為來源
- `--remote=origin`：設定遠端名稱
- `--push`：自動推送

#### 3.4 啟用 GitHub Pages

**API 方式**（推薦）：
```powershell
# 建立 JSON 設定檔
Set-Content -Path "C:\temp\pages.json" -Value '{"source":{"branch":"master","path":"/"}}'

# 呼叫 GitHub API 啟用 Pages
gh api -X POST /repos/<使用者名稱>/teaching-cockpit/pages --input "C:\temp\pages.json"
```

**回應確認**：
```json
{
  "html_url": "https://<使用者名稱>.github.io/teaching-cockpit/",
  "status": "built",
  "source": {"branch":"master","path":"/"}
}
```

**驗證部署狀態**：
```powershell
gh api /repos/<使用者名稱>/teaching-cockpit/pages
```

### Phase 4: 驗證與交付

#### 4.1 驗證檢查清單

- [ ] 倉庫已建立且可見
- [ ] 所有檔案已推送（`git ls-tree -r master --name-only`）
- [ ] GitHub Pages 狀態為 `built`
- [ ] 公開 URL 可訪問（`https://<使用者名稱>.github.io/teaching-cockpit/`）
- [ ] `index.html` 正確載入（非 `dashboard.html` 404）
- [ ] 所有資源連結正常（開發者工具 Console 無 404 錯誤）

#### 4.2 常見問題排除

**問題 1：Pages 未啟用**
- 確認倉庫為 Public（私有倉庫需要 Pro 才能使用 Pages）
- 確認 `index.html` 存在於根目錄
- 等待 1-2 分鐘後重新整理

**問題 2：資源 404**
- 檢查檔案名稱是否含特殊字元（建議使用 URL-safe 名稱）
- 確認大小寫一致（GitHub Pages 區分大小寫）
- 檢查相對路徑是否正確

**問題 3：檔案過大**
- 音訊/影片建議壓縮或改放 YouTube
- 圖片使用 TinyPNG 等工具壓縮
- 考慮使用 Git LFS（需額外設定）

**問題 4：中文檔名亂碼**
- Git 會自動處理 Unicode 檔名
- 若出現問題，可考慮重新命名為 ASCII 安全名稱

## 最佳實踐

### 1. 檔案命名規範

```
# 建議格式
課程名稱_資源類型.副檔名
例：國小國語5下_第08課_海洋的殺手_簡報.pdf

# URL-safe 替代（若需要）
lesson-08-ocean-killer_slides.pdf
```

### 2. 資源大小建議

| 類型 | 建議大小 | 處理方式 |
|------|---------|---------|
| HTML | < 500KB | 直接上傳 |
| CSS/JS | < 100KB | 直接上傳 |
| 圖片 | < 5MB | 壓縮後上傳 |
| 音訊 | < 50MB | 壓縮或改放 YouTube |
| 影片 | < 100MB | 建議放 YouTube |

### 3. 多課程管理

**方案 A：單一倉庫 + 子目錄**（推薦）**
```
/
├── index.html          # 課程總覽
├── lesson-08/
│   └── index.html
└── lesson-09/
    └── index.html
```

**方案 B：獨立倉庫**
- `teaching-cockpit-08`
- `teaching-cockpit-09`

### 4. 安全性考量

- 不上傳含學生個資的檔案
- 不上傳版權受限的教材
- 公開倉庫意味著所有人可見
- 測驗答案建議放 `answers/` 目錄並在 `.gitignore` 排除（或放別處）

## 觸發詞

當使用者說出以下詞句時，自動啟用此技能：

- 「教學駕駛艙」
- 「備課包」
- 「駕駛艙網頁」
- 「GitHub Pages 部署」
- 「教學儀表板」
- 「課程網頁」
- 「上傳駕駛艙」
- 「部署教學頁面」
- 「NotebookLM 網頁」
- 「整合教學資源」

## 輸入輸出

**輸入**：
- NotebookLM 產出的備課包目錄路徑
- 課程名稱/編號
- 倉庫名稱偏好（可選）

**輸出**：
- GitHub 倉庫 URL
- GitHub Pages 公開 URL
- 檔案清單與大小統計
- 部署狀態確認

## 範例對話

**使用者**：「請幫我把這個駕駛艙推到 GitHub」

**助理**：
1. 檢查檔案大小是否符合限制
2. 盤點資源檔案
3. 確認排除項目（影片、個人化教材）
4. 建立 `teaching-cockpit` 倉庫
5. 複製檔案、初始化 git、推送
6. 啟用 GitHub Pages
7. 回傳公開 URL

**回覆格式**：
```
部署完成！

倉庫：https://github.com/<使用者>/teaching-cockpit
網頁：https://<使用者>.github.io/teaching-cockpit/

檔案清單：
- index.html (90 KB)
- quiz.html (35 KB)
- slides/課程簡報.pdf (24 MB)
- audio/課程音訊.mp4 (36 MB)
- ...

驗證狀態：✅ GitHub Pages 已啟用（status: built）
```

## 版本歷史

- **v1.0.0** (2026-05-10)：初始版本，支援 NotebookLM 備課包 → GitHub Pages 完整流程
