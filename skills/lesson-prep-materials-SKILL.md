---
name: lesson-prep-materials
description: |
  備課素材整理與產出物心智圖生成技能。掃描 NotebookLM 備課包目錄，自動識別、分類所有教學產出物，並生成結構化的心智圖（JSON）與素材清單，為後續製作教學駕駛艙做準備。
  觸發詞：整理備課素材、產出物心智圖、備課包整理、素材清單、NotebookLM 產出整理、教學資源盤點、備課素材分類、產出物結構圖。
  適用場景：NotebookLM 生成大量備課檔案後，需要快速了解有哪些素材、各素材類型與用途，並產生結構化的心智圖供教學駕駛艙使用。
license: MIT
metadata:
  version: "1.0.0"
  category: education
  author: shitsuke0725-web
---

# 備課素材整理器 · Lesson Prep Materials

掃描 NotebookLM 備課包，自動識別、分類教學產出物，並生成結構化心智圖與素材清單。

## 使用前提

此技能專為「NotebookLM 備課包 → 素材整理 → 心智圖生成」的工作流程設計。

**適用場景**：
- NotebookLM 產出大量檔案，需要快速盤點
- 需要了解各素材的類型、大小、用途
- 需要生成心智圖 JSON 供駕駛艙使用
- 需要為製作駕駛艙準備素材清單

**輸入**：NotebookLM 輸出目錄（含各類子目錄）
**輸出**：
- `materials-inventory.json` - 素材清單與結構
- `materials-mindmap.json` - 產出物心智圖
- `materials-report.md` - 可讀的素材報告

## 工作流程

### Phase 1: 掃描與識別

#### 1.1 掃描備課包目錄

```powershell
# 掃描 NotebookLM 目錄結構
$BasePath = "C:\Users\<user>\Documents\NotebookLM"
Get-ChildItem -Path $BasePath -Recurse -File |
  Where-Object { $_.FullName -notmatch 'node_modules|\.git|\.opencode' } |
  Select-Object FullName, Length, Extension, LastWriteTime |
  Sort-Object Extension, FullName
```

#### 1.2 檔案分類規則

| 副檔名 | 類型 | 子目錄 | 用途 |
|--------|------|--------|------|
| `.pdf` | 簡報 | `slides/` | 教學簡報 |
| `.pptx` | 簡報原始檔 | `slides/` | 可編輯簡報 |
| `.mp4` | 音訊/影片 | `audio/`, `video/` | 課程音訊 |
| `.m4a` | 音訊 | `audio/` | 課程音訊 |
| `.png` | 圖表 | `infographics/` | 資訊圖表 |
| `.jpg`, `.jpeg` | 圖片 | `infographics/` | 輔助圖片 |
| `.json` | 資料 | `mindmaps/`, `quizzes/` | 心智圖/測驗 |
| `.csv` | 表格 | `sheets/`, `quizzes/` | 試算表資料 |
| `.md` | 文件 | `differentiated/` | 差異化教材 |
| `.html` | 網頁 | 根目錄 | 駕駛艙 |

#### 1.3 檔案命名解析

從檔名提取課程資訊：

```powershell
# 解析檔名範例：國小國語5下_第08課_海洋的殺手_簡報.pdf
$Pattern = '(?x)
  (?<grade>國小|國中|高中)(?<subject>.+?)(?<semester>\d+下?)
  _第(?<lesson>\d+)課
  _(?<topic>.+?)
  _(?<type>.+?)\.(?<ext>.+)'
```

**提取欄位**：
- `grade` - 年級（國小/國中/高中）
- `subject` - 科目（國語/數學/自然等）
- `semester` - 學期（5下/6上等）
- `lesson` - 課號（08/09等）
- `topic` - 課名（海洋的殺手）
- `type` - 類型（簡報/音訊/測驗等）

### Phase 2: 素材清單生成

#### 2.1 生成 materials-inventory.json

```json
{
  "course": {
    "grade": "國小",
    "subject": "國語",
    "semester": "5下",
    "lesson": "08",
    "topic": "海洋的殺手",
    "date": "2026-05-10"
  },
  "materials": {
    "slides": [
      {
        "filename": "國小國語5下_第08課_海洋的殺手_簡報.pdf",
        "path": "slides/國小國語5下_第08課_海洋的殺手_簡報.pdf",
        "size": 25123456,
        "pages": 20,
        "format": "pdf"
      }
    ],
    "audio": [
      {
        "filename": "國小國語5下_第08課_海洋的殺手_音訊.mp4",
        "path": "audio/國小國語5下_第08課_海洋的殺手_音訊.mp4",
        "size": 36346214,
        "duration": "15:30",
        "format": "mp4"
      }
    ],
    "infographics": [
      {
        "filename": "國小國語5下_第08課_海洋的殺手_資訊圖表.png",
        "path": "infographics/國小國語5下_第08課_海洋的殺手_資訊圖表.png",
        "size": 4936254,
        "dimensions": "1920x1080",
        "format": "png"
      }
    ],
    "mindmaps": [
      {
        "filename": "國小國語5下_第08課_海洋的殺手_心智圖.json",
        "path": "mindmaps/國小國語5下_第08課_海洋的殺手_心智圖.json",
        "size": 1815,
        "nodes": 25,
        "format": "json"
      }
    ],
    "quizzes": [
      {
        "filename": "國小國語5下_第08課_海洋的殺手_測驗.json",
        "path": "quizzes/國小國語5下_第08課_海洋的殺手_測驗.json",
        "size": 15107,
        "questions": 10,
        "format": "json"
      }
    ],
    "sheets": [
      {
        "filename": "國小國語5下_第08課_海洋的殺手_測驗.csv",
        "path": "sheets/國小國語5下_第08課_海洋的殺手_測驗.csv",
        "size": 6779,
        "rows": 12,
        "format": "csv"
      }
    ]
  },
  "statistics": {
    "totalFiles": 8,
    "totalSize": 92543420,
    "byType": {
      "slides": 1,
      "audio": 1,
      "infographics": 1,
      "mindmaps": 1,
      "quizzes": 1,
      "sheets": 1
    }
  }
}
```

#### 2.2 生成可讀報告 materials-report.md

```markdown
# 備課素材報告：國小國語5下_第08課_海洋的殺手

## 課程資訊
- 年級：國小五年級下學期
- 科目：國語
- 課名：海洋的殺手
- 備課日期：2026-05-10

## 素材清單

### 📑 簡報（1 個檔案）
| 檔名 | 大小 | 頁數 | 格式 |
|------|------|------|------|
| 海洋的殺手_簡報.pdf | 24 MB | 20 | PDF |

### 🎧 音訊（1 個檔案）
| 檔名 | 大小 | 時長 | 格式 |
|------|------|------|------|
| 海洋的殺手_音訊.mp4 | 36 MB | 15:30 | MP4 |

### 📊 資訊圖表（1 個檔案）
| 檔名 | 大小 | 尺寸 | 格式 |
|------|------|------|------|
| 海洋的殺手_資訊圖表.png | 5 MB | 1920x1080 | PNG |

### 🧠 心智圖（1 個檔案）
| 檔名 | 大小 | 節點數 | 格式 |
|------|------|--------|------|
| 海洋的殺手_心智圖.json | 2 KB | 25 | JSON |

### ✏️ 測驗（1 個檔案）
| 檔名 | 大小 | 題數 | 格式 |
|------|------|------|------|
| 海洋的殺手_測驗.json | 15 KB | 10 | JSON |

### 📋 試算表（1 個檔案）
| 檔名 | 大小 | 列數 | 格式 |
|------|------|------|------|
| 海洋的殺手_測驗.csv | 7 KB | 12 | CSV |

## 統計摘要
- 總檔案數：8
- 總大小：88 MB
- 最大檔案：音訊.mp4（36 MB）
- 最小檔案：心智圖.json（2 KB）

## 使用建議
1. 音訊檔案較大（36 MB），建議確認載入速度
2. 簡報 PDF 適合嵌入 iframe 展示
3. 測驗 JSON 可直接用於互動測驗模組
4. 心智圖 JSON 可視覺化為互動節點圖
```

### Phase 3: 產出物心智圖生成

#### 3.1 心智圖結構設計

以「備課包」為根節點，各類素材為子節點：

```
備課包：海洋的殺手
├── 📑 教學簡報
│   ├── 課程簡報.pdf（20頁）
│   └── 可編輯.pptx
├── 🎧 音訊資源
│   ├── 課程音訊.mp4（15:30）
│   └── 分段音訊.m4a
├── 📊 視覺素材
│   ├── 資訊圖表.png
│   └── 輔助圖片.jpg
├── 🧠 結構化資料
│   ├── 心智圖.json（25節點）
│   ├── 測驗.json（10題）
│   └── 試算表.csv（12列）
└── 📝 教師參考
    ├── 差異化教材.md
    └── 教案補充.md
```

#### 3.2 生成 materials-mindmap.json

```json
{
  "root": {
    "id": "root",
    "label": "海洋的殺手",
    "type": "course",
    "metadata": {
      "grade": "國小5下",
      "subject": "國語",
      "lesson": "08"
    }
  },
  "nodes": [
    {
      "id": "slides",
      "label": "📑 教學簡報",
      "parent": "root",
      "type": "category",
      "count": 2,
      "children": [
        {
          "id": "slides-1",
          "label": "課程簡報.pdf",
          "parent": "slides",
          "type": "file",
          "format": "pdf",
          "size": "24MB",
          "pages": 20
        }
      ]
    },
    {
      "id": "audio",
      "label": "🎧 音訊資源",
      "parent": "root",
      "type": "category",
      "count": 1,
      "children": [
        {
          "id": "audio-1",
          "label": "課程音訊.mp4",
          "parent": "audio",
          "type": "file",
          "format": "mp4",
          "size": "36MB",
          "duration": "15:30"
        }
      ]
    },
    {
      "id": "visuals",
      "label": "📊 視覺素材",
      "parent": "root",
      "type": "category",
      "count": 1
    },
    {
      "id": "data",
      "label": "🧠 結構化資料",
      "parent": "root",
      "type": "category",
      "count": 3
    },
    {
      "id": "teacher",
      "label": "📝 教師參考",
      "parent": "root",
      "type": "category",
      "count": 2
    }
  ]
}
```

#### 3.3 視覺化建議

**簡易 SVG 渲染**：
```javascript
function renderMaterialsMindmap(data, container) {
  // 使用樹狀佈局，根節點在中央，五個類別向外放射
  const categories = data.nodes.filter(n => n.parent === 'root');
  const angleStep = 360 / categories.length;
  
  categories.forEach((cat, i) => {
    const angle = (i * angleStep - 90) * (Math.PI / 180);
    const x = 400 + 200 * Math.cos(angle);
    const y = 300 + 200 * Math.sin(angle);
    // 繪製節點...
  });
}
```

**或使用 D3.js 力導向圖**：
```javascript
const simulation = d3.forceSimulation(data.nodes)
  .force('link', d3.forceLink(data.links).id(d => d.id))
  .force('charge', d3.forceManyBody().strength(-300))
  .force('center', d3.forceCenter(400, 300));
```

### Phase 4: 與駕駛艙銜接

#### 4.1 生成駕駛艙素材對應表

```json
{
  "cockpitMapping": {
    "slides": {
      "module": "簡報展示",
      "component": "iframe",
      "source": "slides/*.pdf",
      "priority": "P0"
    },
    "audio": {
      "module": "音訊播放",
      "component": "audio-player",
      "source": "audio/*.mp4",
      "priority": "P0"
    },
    "infographics": {
      "module": "資訊圖表",
      "component": "lightbox-gallery",
      "source": "infographics/*.png",
      "priority": "P0"
    },
    "mindmaps": {
      "module": "心智圖探索",
      "component": "interactive-svg",
      "source": "mindmaps/*.json",
      "priority": "P1"
    },
    "quizzes": {
      "module": "隨堂測驗",
      "component": "quiz-engine",
      "source": "quizzes/*.json",
      "priority": "P1"
    }
  }
}
```

#### 4.2 一鍵生成駕駛艙指令

根據素材清單，生成對應的 teaching-cockpit-builder 指令：

```
我已整理完備課素材：

📁 素材清單（共 8 個檔案）
- 📑 簡報：1 個 PDF（20 頁）
- 🎧 音訊：1 個 MP4（15:30）
- 📊 圖表：1 個 PNG
- 🧠 心智圖：1 個 JSON（25 節點）
- ✏️ 測驗：1 個 JSON（10 題）
- 📋 表格：1 個 CSV

👉 請使用 teaching-cockpit-builder 技能
    主題：海洋（藍色系）
    模組：簡報、音訊、圖表、測驗、心智圖
```

## 觸發詞

當使用者說出以下詞句時，自動啟用此技能：

- 「整理備課素材」
- 「產出物心智圖」
- 「備課包整理」
- 「素材清單」
- 「NotebookLM 產出整理」
- 「教學資源盤點」
- 「備課素材分類」
- 「產出物結構圖」
- 「掃描備課包」
- 「素材報告」

## 輸入輸出

**輸入**：
- NotebookLM 輸出目錄路徑
- 檔案命名模式（選配）

**輸出**：
- `materials-inventory.json` - 結構化素材清單
- `materials-mindmap.json` - 產出物心智圖
- `materials-report.md` - 人類可讀的素材報告
- 駕駛艙素材對應建議

## 與其他技能的協作

```
NotebookLM 產出
      ↓
lesson-prep-materials（本技能）
  ├── 掃描與識別
  ├── 生成素材清單
  └── 產出物心智圖
      ↓
teaching-cockpit-builder
  └── 製作 HTML 駕駛艙
      ↓
teaching-cockpit-publisher
  └── 部署到 GitHub Pages
```

## 範例對話

**使用者**：「幫我整理一下這個備課包」

**助理**：
1. 掃描 NotebookLM 目錄
2. 識別 8 個教學檔案，分為 6 類
3. 生成 `materials-inventory.json`
4. 生成 `materials-mindmap.json`
5. 生成 `materials-report.md`
6. 提供素材統計與使用建議

**回覆格式**：
```
備課素材整理完成！

📊 掃描結果：
- 總檔案數：8
- 總大小：88 MB
- 類別數：6

📑 素材清單：
教學簡報：1 個（24 MB，20 頁）
音訊資源：1 個（36 MB，15:30）
視覺素材：1 個（5 MB，1920x1080）
結構化資料：3 個（心智圖、測驗、表格）

🧠 產出物心智圖：
已生成 materials-mindmap.json（25 個節點）

📄 報告文件：
materials-report.md（含完整素材說明與使用建議）

👉 下一步：
是否需要使用 teaching-cockpit-builder 製作教學駕駛艙？
```

## 版本歷史

- **v1.0.0** (2026-05-10)：初始版本，支援 NotebookLM 備課包掃描、素材分類、心智圖生成、報告輸出
