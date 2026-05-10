---
name: teaching-cockpit-builder
description: |
  從 NotebookLM 備課包產出的教學資源，製作單一 HTML 教學駕駛艙網頁的技能。
  觸發詞：製作駕駛艙、建立教學網頁、備課包轉網頁、教學儀表板製作、課程網頁生成、NotebookLM 網頁化、製作 dashboard。
  適用場景：教師使用 NotebookLM 生成備課資料（簡報 PDF、音訊、測驗 JSON、心智圖、資訊圖表）後，需要將這些分散資源整合為一個可互動的單一 HTML 網頁，提供給學生課堂使用或課後複習。
license: MIT
metadata:
  version: "1.0.0"
  category: education
  author: shitsuke0725-web
---

# 教學駕駛艙網頁製作器 · Teaching Cockpit Builder

將 NotebookLM 備課包產出的分散教學資源，整合為單一 HTML 互動駕駛艙網頁。

## 使用前提

此技能專為「NotebookLM 備課包素材 → HTML 教學駕駛艙」的製作流程設計。

**適用場景**：
- 已有 NotebookLM 產出的備課素材（簡報、音訊、測驗、心智圖、資訊圖表）
- 需要將多種格式的資源整合為單一入口
- 需要課堂互動功能（測驗、音訊播放、心智圖探索）
- 需要視覺一致的主題設計

**輸入素材類型**：
- `slides/` - 簡報 PDF
- `audio/` - 課程音訊 mp4/m4a
- `infographics/` - 資訊圖表 PNG
- `mindmaps/` - 心智圖 JSON
- `quizzes/` - 測驗 JSON/CSV
- `differentiated/` - 差異化教材（選擇性納入）

**輸出**：
- `dashboard.html` - 主駕駛艙網頁
- `quiz.html` - 獨立測驗頁（選配）

## 製作流程

### Phase 1: 素材分析與規劃

#### 1.1 掃描備課包結構

```
NotebookLM/
├── slides/           ← 簡報 PDF（必用）
├── audio/            ← 音訊檔（必用）
├── infographics/     ← 資訊圖表（必用）
├── mindmaps/         ← 心智圖 JSON（必用）
├── quizzes/          ← 測驗資料（必用）
├── sheets/           ← CSV 資料（選用）
├── differentiated/   ← 分層教材（選用，不顯示答案）
└── video/            ← 影片（建議排除，改放 YouTube）
```

**操作指令**：
```powershell
# 掃描所有素材
Get-ChildItem "C:\Users\<user>\Documents\NotebookLM" -Recurse |
  Where-Object { $_.FullName -notmatch 'node_modules|\.git|\.opencode' } |
  Select-Object FullName, Length
```

#### 1.2 識別課程主題與命名

從檔名提取課程資訊：
- 課號：第08課
- 課名：海洋的殺手
- 年級：國小國語5下
- 產出日期：20260418

**用途**：
- 網頁標題
- 主題色選擇依據
- 檔案路徑設定

#### 1.3 決定主題色系

根據課程內容選擇色彩方案：

| 課程類型 | 主題色 | CSS 變數前綴 | 輔助色 |
|---------|--------|------------|--------|
| 海洋/自然 | 藍色系 | `--ocean-*` | 珊瑚色 `#ff7e5f` |
| 森林/植物 | 綠色系 | `--forest-*` | 泥土色 `#8b6914` |
| 歷史/文化 | 棕色系 | `--earth-*` | 金色 `#d4af37` |
| 科技/未來 | 藍紫色 | `--tech-*` | 螢光色 `#00ff88` |
| 節慶/傳統 | 紅金色 | `--fest-*` | 喜氣紅 `#e74c3c` |

**色階定義（以海洋為例）**：
```css
:root {
  --ocean-50:  #e6f4f8;   /* 最淺 */
  --ocean-100: #b3e1f2;
  --ocean-200: #80cde9;
  --ocean-300: #4dbae0;
  --ocean-400: #1aa6d6;   /* 主色 */
  --ocean-500: #0088b5;   /* 強調 */
  --ocean-600: #006b8f;
  --ocean-700: #004e68;   /* 文字 */
  --ocean-800: #003141;
  --ocean-900: #00141a;   /* 最深 */
  
  --accent: #ff7e5f;      /* 輔助強調色 */
  --bg-gradient: linear-gradient(135deg, #e6f4f8 0%, #b3e1f2 40%, #80cde9 100%);
}
```

#### 1.4 功能模組規劃

必備模組（依課程素材決定）：

| 模組 | 素材來源 | 優先級 | 互動程度 |
|------|---------|--------|---------|
| 簡報展示 | slides/*.pdf | P0 | 中（iframe） |
| 音訊播放 | audio/*.mp4 | P0 | 高（自製播放器） |
| 資訊圖表 | infographics/*.png | P0 | 中（燈箱放大） |
| 隨堂測驗 | quizzes/*.json | P1 | 高（答題計分） |
| 心智圖 | mindmaps/*.json | P1 | 高（節點探索） |
| 差異化提示 | differentiated/*.md | P2 | 低（教師參考） |

### Phase 2: HTML 架構設計

#### 2.1 整體結構

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>教學駕駛艙｜{年級}_{課號}_{課名}</title>
  <style>
    /* CSS 變數與設計系統 */
    /* 版面配置 */
    /* 模組樣式 */
  </style>
</head>
<body>
  <!-- 頂部導航 -->
  <nav class="topbar">...</nav>
  
  <!-- 模式切換（教師/學生） -->
  <div class="mode-toggle">...</div>
  
  <!-- 主內容區 -->
  <main class="container">
    <!-- 簡報模組 -->
    <section id="slides" class="module">...</section>
    
    <!-- 音訊模組 -->
    <section id="audio" class="module">...</section>
    
    <!-- 資訊圖表模組 -->
    <section id="infographics" class="module">...</section>
    
    <!-- 測驗模組 -->
    <section id="quiz" class="module">...</section>
    
    <!-- 心智圖模組 -->
    <section id="mindmap" class="module">...</section>
  </main>
  
  <!-- 底部資訊 -->
  <footer>...</footer>
  
  <script>
    /* 互動邏輯 */
  </script>
</body>
</html>
```

#### 2.2 頂部導航設計

```html
<nav class="topbar">
  <div class="logo">
    <div class="icon">🎓</div>
    <span>教學駕駛艙｜{課名}</span>
  </div>
  <div class="nav-links">
    <a href="#slides">簡報</a>
    <a href="#audio">音訊</a>
    <a href="#infographics">圖表</a>
    <a href="#quiz">測驗</a>
    <a href="#mindmap">心智圖</a>
  </div>
</nav>
```

**CSS**：
```css
.topbar {
  position: fixed; top: 0; left: 0; right: 0; z-index: 100;
  height: 60px;
  background: rgba(255,255,255,0.85);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(0,136,181,0.15);
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 32px;
}
```

#### 2.3 模式切換（教師/學生）

```html
<div class="mode-toggle">
  <button class="active" data-mode="teacher">教師模式</button>
  <button data-mode="student">學生模式</button>
</div>
```

**功能差異**：

| 功能 | 教師模式 | 學生模式 |
|------|---------|---------|
| 測驗答案 | 顯示 | 隱藏 |
| 差異化提示 | 顯示完整 | 顯示簡化 |
| 編輯功能 | 啟用 | 禁用 |
| 進度監控 | 顯示全班 | 僅個人 |

### Phase 3: CSS 設計系統

#### 3.1 變數定義

```css
:root {
  /* 主色階 */
  --primary-50: #e6f4f8;
  --primary-100: #b3e1f2;
  --primary-200: #80cde9;
  --primary-300: #4dbae0;
  --primary-400: #1aa6d6;
  --primary-500: #0088b5;
  --primary-600: #006b8f;
  --primary-700: #004e68;
  --primary-800: #003141;
  --primary-900: #00141a;

  /* 輔助色 */
  --accent: #ff7e5f;
  --success: #2ecc71;
  --warn: #f39c12;
  --danger: #e74c3c;

  /* 字體 */
  --font-body: 'Microsoft JhengHei', 'PingFang TC', 'Noto Sans TC', sans-serif;
  --font-mono: 'Consolas', 'Courier New', monospace;

  /* 間距系統 4px base */
  --space-1: 4px;   --space-2: 8px;
  --space-3: 12px;  --space-4: 16px;
  --space-5: 20px;  --space-6: 24px;
  --space-8: 32px;  --space-10: 40px;
  --space-12: 48px;

  /* 圓角 */
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-xl: 24px;

  /* 陰影 */
  --shadow-sm: 0 1px 3px rgba(0,50,80,0.08);
  --shadow-md: 0 4px 12px rgba(0,50,80,0.12);
  --shadow-lg: 0 8px 24px rgba(0,50,80,0.18);

  /* 動畫 */
  --ease-out: cubic-bezier(0.25,0.46,0.45,0.94);
  --duration-fast: 150ms;
  --duration-normal: 300ms;
}
```

#### 3.2 響應式斷點

```css
/* 手機優先 */
.container { padding: 80px 16px 40px; max-width: 1200px; margin: 0 auto; }

/* 平板 */
@media (min-width: 768px) {
  .container { padding: 80px 24px 40px; }
  .module-grid { grid-template-columns: 1fr 1fr; }
}

/* 桌面 */
@media (min-width: 1024px) {
  .container { padding: 80px 32px 40px; }
  .module-grid { grid-template-columns: 1fr 1fr 1fr; }
}
```

#### 3.3 模組卡片樣式

```css
.module {
  background: rgba(255,255,255,0.9);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  margin-bottom: var(--space-6);
  box-shadow: var(--shadow-md);
  border: 1px solid rgba(0,136,181,0.1);
}

.module-header {
  display: flex; align-items: center; gap: var(--space-3);
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 2px solid var(--primary-100);
}

.module-title {
  font-size: 20px; font-weight: 700;
  color: var(--primary-700);
}

.module-icon {
  width: 36px; height: 36px;
  background: linear-gradient(135deg, var(--primary-400), var(--primary-600));
  border-radius: var(--radius-md);
  display: flex; align-items: center; justify-content: center;
  color: white; font-size: 18px;
}
```

### Phase 4: 各模組實作

#### 4.1 簡報模組（PDF 嵌入）

```html
<section id="slides" class="module">
  <div class="module-header">
    <div class="module-icon">📑</div>
    <h2 class="module-title">課程簡報</h2>
  </div>
  <div class="pdf-container">
    <iframe src="slides/課程簡報.pdf#toolbar=0&navpanes=0" 
            width="100%" height="600px" 
            style="border: none; border-radius: var(--radius-md);">
    </iframe>
  </div>
  <div class="pdf-controls">
    <a href="slides/課程簡報.pdf" download class="btn">📥 下載簡報</a>
  </div>
</section>
```

**PDF.js 進階方案**（需外連 CDN）：
```html
<!-- 使用 PDF.js 獲得更好控制 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<div id="pdf-viewer"></div>
<script>
  pdfjsLib.getDocument('slides/課程簡報.pdf').promise.then(doc => {
    // 自定義渲染邏輯
  });
</script>
```

#### 4.2 音訊模組（自製播放器）

```html
<section id="audio" class="module">
  <div class="module-header">
    <div class="module-icon">🎧</div>
    <h2 class="module-title">課程音訊</h2>
  </div>
  <div class="audio-player">
    <audio id="lesson-audio" src="audio/課程音訊.mp4"></audio>
    <div class="player-controls">
      <button id="play-btn" class="play-btn">▶️</button>
      <div class="progress-bar">
        <div class="progress-fill"></div>
      </div>
      <span class="time-display">0:00 / 0:00</span>
    </div>
    <div class="speed-controls">
      <button data-speed="0.75">0.75x</button>
      <button data-speed="1" class="active">1x</button>
      <button data-speed="1.25">1.25x</button>
      <button data-speed="1.5">1.5x</button>
    </div>
  </div>
</section>
```

**JavaScript**：
```javascript
const audio = document.getElementById('lesson-audio');
const playBtn = document.getElementById('play-btn');
const progressBar = document.querySelector('.progress-bar');
const progressFill = document.querySelector('.progress-fill');
const timeDisplay = document.querySelector('.time-display');

playBtn.addEventListener('click', () => {
  if (audio.paused) {
    audio.play();
    playBtn.textContent = '⏸️';
  } else {
    audio.pause();
    playBtn.textContent = '▶️';
  }
});

audio.addEventListener('timeupdate', () => {
  const percent = (audio.currentTime / audio.duration) * 100;
  progressFill.style.width = percent + '%';
  timeDisplay.textContent = formatTime(audio.currentTime) + ' / ' + formatTime(audio.duration);
});

progressBar.addEventListener('click', (e) => {
  const rect = progressBar.getBoundingClientRect();
  const percent = (e.clientX - rect.left) / rect.width;
  audio.currentTime = percent * audio.duration;
});

function formatTime(seconds) {
  const mins = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60);
  return `${mins}:${secs.toString().padStart(2, '0')}`;
}
```

#### 4.3 資訊圖表模組

```html
<section id="infographics" class="module">
  <div class="module-header">
    <div class="module-icon">📊</div>
    <h2 class="module-title">資訊圖表</h2>
  </div>
  <div class="infographic-grid">
    <figure class="infographic-item" data-full="infographics/圖表1.png">
      <img src="infographics/圖表1.png" alt="圖表1說明" loading="lazy">
      <figcaption>圖表1說明文字</figcaption>
    </figure>
    <!-- 更多圖表 -->
  </div>
</section>
```

**燈箱效果**：
```javascript
document.querySelectorAll('.infographic-item').forEach(item => {
  item.addEventListener('click', () => {
    const fullSrc = item.dataset.full;
    const lightbox = document.createElement('div');
    lightbox.className = 'lightbox';
    lightbox.innerHTML = `<img src="${fullSrc}" alt=""><span class="close">×</span>`;
    document.body.appendChild(lightbox);
    lightbox.addEventListener('click', () => lightbox.remove());
  });
});
```

#### 4.4 測驗模組（JSON 解析）

**JSON 格式預期**：
```json
{
  "title": "海洋的殺手 - 課後測驗",
  "questions": [
    {
      "id": 1,
      "type": "choice",
      "question": "海洋殺手指的是什麼？",
      "options": ["鯊魚", "塑膠垃圾", "海草", "珊瑚"],
      "answer": 1,
      "explanation": "塑膠垃圾是現代海洋最大的威脅..."
    },
    {
      "id": 2,
      "type": "tf",
      "question": "所有塑膠都可以在海洋中分解？",
      "answer": false,
      "explanation": "多數塑膠需要數百年才能分解..."
    }
  ]
}
```

**HTML 結構**：
```html
<section id="quiz" class="module">
  <div class="module-header">
    <div class="module-icon">✏️</div>
    <h2 class="module-title">隨堂測驗</h2>
  </div>
  <div id="quiz-container">
    <div id="quiz-intro">
      <p>準備好了嗎？點擊開始測驗！</p>
      <button id="start-quiz" class="btn-primary">開始測驗</button>
    </div>
    <div id="quiz-question" style="display:none;">
      <div class="question-progress">題目 <span id="current-q">1</span> / <span id="total-q">5</span></div>
      <h3 id="question-text"></h3>
      <div id="options-container"></div>
    </div>
    <div id="quiz-result" style="display:none;">
      <h3>測驗完成！</h3>
      <p>你的得分：<span id="score">0</span> / <span id="total-score">0</span></p>
      <div id="review-container"></div>
      <button id="retry-quiz" class="btn">重新測驗</button>
    </div>
  </div>
</section>
```

**JavaScript 核心邏輯**：
```javascript
let quizData = null;
let currentQuestion = 0;
let score = 0;
let userAnswers = [];

// 載入測驗資料
fetch('quizzes/測驗.json')
  .then(r => r.json())
  .then(data => {
    quizData = data;
    document.getElementById('total-q').textContent = data.questions.length;
  });

document.getElementById('start-quiz').addEventListener('click', startQuiz);

function startQuiz() {
  currentQuestion = 0;
  score = 0;
  userAnswers = [];
  document.getElementById('quiz-intro').style.display = 'none';
  document.getElementById('quiz-question').style.display = 'block';
  showQuestion();
}

function showQuestion() {
  const q = quizData.questions[currentQuestion];
  document.getElementById('current-q').textContent = currentQuestion + 1;
  document.getElementById('question-text').textContent = q.question;
  
  const container = document.getElementById('options-container');
  container.innerHTML = '';
  
  if (q.type === 'choice') {
    q.options.forEach((opt, idx) => {
      const btn = document.createElement('button');
      btn.className = 'option-btn';
      btn.textContent = opt;
      btn.addEventListener('click', () => checkAnswer(idx));
      container.appendChild(btn);
    });
  } else if (q.type === 'tf') {
    ['True', 'False'].forEach((val, idx) => {
      const btn = document.createElement('button');
      btn.className = 'option-btn';
      btn.textContent = val;
      btn.addEventListener('click', () => checkAnswer(idx === 0));
      container.appendChild(btn);
    });
  }
}

function checkAnswer(answer) {
  const q = quizData.questions[currentQuestion];
  const correct = answer === q.answer;
  if (correct) score++;
  userAnswers.push({ question: q.id, correct, answer });
  
  // 顯示回饋
  const buttons = document.querySelectorAll('.option-btn');
  buttons.forEach((btn, idx) => {
    if ((q.type === 'choice' && idx === q.answer) || 
        (q.type === 'tf' && (idx === 0) === q.answer)) {
      btn.classList.add('correct');
    } else if ((q.type === 'choice' && idx === answer) || 
               (q.type === 'tf' && (idx === 0) === answer)) {
      btn.classList.add('wrong');
    }
    btn.disabled = true;
  });
  
  // 顯示解析（教師模式）
  if (document.body.dataset.mode === 'teacher') {
    showExplanation(q.explanation);
  }
  
  setTimeout(() => {
    currentQuestion++;
    if (currentQuestion < quizData.questions.length) {
      showQuestion();
    } else {
      showResult();
    }
  }, 1500);
}

function showResult() {
  document.getElementById('quiz-question').style.display = 'none';
  document.getElementById('quiz-result').style.display = 'block';
  document.getElementById('score').textContent = score;
  document.getElementById('total-score').textContent = quizData.questions.length;
}
```

#### 4.5 心智圖模組

**JSON 格式**：
```json
{
  "root": "海洋的殺手",
  "nodes": [
    { "id": "1", "label": "污染源", "parent": "root" },
    { "id": "2", "label": "塑膠垃圾", "parent": "1" },
    { "id": "3", "label": "影響", "parent": "root" },
    { "id": "4", "label": "海洋生物", "parent": "3" }
  ]
}
```

**SVG 渲染**：
```javascript
function renderMindmap(data, container) {
  const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  svg.setAttribute('viewBox', '0 0 800 600');
  svg.style.width = '100%';
  
  // 簡易樹狀佈局
  const levels = {};
  data.nodes.forEach(node => {
    const level = getLevel(node, data);
    if (!levels[level]) levels[level] = [];
    levels[level].push(node);
  });
  
  // 繪製節點與連線
  Object.entries(levels).forEach(([level, nodes]) => {
    const x = level * 200 + 50;
    const gap = 600 / (nodes.length + 1);
    nodes.forEach((node, i) => {
      const y = gap * (i + 1);
      
      // 連線
      if (node.parent !== 'root') {
        const parent = data.nodes.find(n => n.id === node.parent);
        const parentLevel = getLevel(parent, data);
        const parentX = parentLevel * 200 + 50;
        // ... 繪製貝茲曲線
      }
      
      // 節點
      const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
      g.innerHTML = `
        <rect x="${x}" y="${y-20}" width="120" height="40" rx="8" 
              fill="var(--primary-100)" stroke="var(--primary-500)" stroke-width="2"/>
        <text x="${x+60}" y="${y+5}" text-anchor="middle" fill="var(--primary-800)" font-size="14">
          ${node.label}
        </text>
      `;
      svg.appendChild(g);
    });
  });
  
  container.appendChild(svg);
}
```

**或使用 D3.js**（進階）：
```html
<script src="https://d3js.org/d3.v7.min.js"></script>
<script>
  // 使用 D3 的 tree layout 實現更複雜的心智圖
</script>
```

### Phase 5: 進階功能

#### 5.1 教師/學生模式切換

```javascript
document.querySelectorAll('.mode-toggle button').forEach(btn => {
  btn.addEventListener('click', () => {
    const mode = btn.dataset.mode;
    document.body.dataset.mode = mode;
    
    // 切換顯示內容
    document.querySelectorAll('.teacher-only').forEach(el => {
      el.style.display = mode === 'teacher' ? 'block' : 'none';
    });
    
    // 更新按鈕狀態
    document.querySelectorAll('.mode-toggle button').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
  });
});
```

#### 5.2 進度儲存（LocalStorage）

```javascript
const STORAGE_KEY = 'cockpit-progress';

function saveProgress() {
  const progress = {
    audioTime: audio.currentTime,
    quizAnswers: userAnswers,
    lastVisit: new Date().toISOString()
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(progress));
}

function loadProgress() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    const progress = JSON.parse(saved);
    audio.currentTime = progress.audioTime || 0;
  }
}

window.addEventListener('beforeunload', saveProgress);
```

#### 5.3 鍵盤快速鍵

```javascript
document.addEventListener('keydown', (e) => {
  if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
  
  switch(e.key) {
    case ' ':
      e.preventDefault();
      playBtn.click();
      break;
    case 'ArrowRight':
      audio.currentTime += 10;
      break;
    case 'ArrowLeft':
      audio.currentTime -= 10;
      break;
  }
});
```

### Phase 6: 測驗與優化

#### 6.1 跨瀏覽器測試

| 瀏覽器 | 測試項目 |
|--------|---------|
| Chrome | 主要開發目標 |
| Edge | PDF iframe 相容性 |
| Safari | 音訊播放、backdrop-filter |
| Firefox | CSS Grid、SVG |

#### 6.2 效能檢查

- [ ] 圖片使用 `loading="lazy"`
- [ ] 音訳使用 `preload="metadata"`
- [ ] CSS 動畫使用 `transform` 而非 `top/left`
- [ ] 無大型外部依賴（盡量內聯）

#### 6.3 無障礙檢查

- [ ] 所有圖片有 `alt` 文字
- [ ] 色彩對比度 > 4.5:1
- [ ] 鍵盤可操作所有功能
- [ ] ARIA 標籤適當使用

## 觸發詞

當使用者說出以下詞句時，自動啟用此技能：

- 「製作駕駛艙」
- 「建立教學網頁」
- 「備課包轉網頁」
- 「教學儀表板製作」
- 「課程網頁生成」
- 「NotebookLM 網頁化」
- 「製作 dashboard」
- 「把備課包做成網頁」
- 「整合教學資源」
- 「做一個教學頁面」

## 範例對話

**使用者**：「我有一個 NotebookLM 備課包，幫我做一個教學駕駛艙網頁」

**助理**：
1. 掃描備課包目錄，識別所有素材類型
2. 詢問課程主題以選擇配色（或直接從課名推斷）
3. 規劃模組：簡報、音訊、圖表、測驗、心智圖
4. 生成完整 `dashboard.html`（單一檔案，內聯 CSS/JS）
5. 詢問是否需要獨立 `quiz.html`
6. 提供預覽與下載

**輸出檔案結構**：
```
output/
├── dashboard.html      # 主駕駛艙（單一檔案，含內聯樣式與腳本）
└── quiz.html           # 獨立測驗頁（選配）
```

## 相關技能

- `teaching-cockpit-publisher` - 將製作好的駕駛艙部署到 GitHub Pages
- `notebooklm-research` - 使用 NotebookLM 生成備課素材

## 版本歷史

- **v1.0.0** (2026-05-10)：初始版本，支援 NotebookLM 備課包 → HTML 駕駛艙完整製作流程
