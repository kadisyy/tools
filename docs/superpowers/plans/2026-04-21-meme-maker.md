# Meme Maker Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 纯前端表情包批量制作工具，上传一张图片 + 输入文字，自动生成 12 种风格变体，每张可单独下载。

**Architecture:** 单页 HTML 文件（`meme-maker.html`），分屏布局：左侧控制面板（上传/输入/参数），右侧 Canvas 渲染的变体网格。所有渲染通过 HTML5 Canvas API 完成，无需后端，无需第三方库。首页 `index.html` 新增工具卡片入口。

**Tech Stack:** 纯 HTML5 + CSS3 + Vanilla JS，Canvas 2D API，FileReader API，`canvas.toBlob()` 导出 PNG。

---

## File Structure

- Create: `meme-maker.html` — 完整工具页面（控制面板 + Canvas 渲染 + 下载逻辑）
- Modify: `index.html` — 将 TOOL_003 占位卡片替换为 Meme Maker 真实卡片

---

### Task 1: 创建 meme-maker.html 页面骨架与样式

**Files:**
- Create: `meme-maker.html`

- [ ] **Step 1: 创建 HTML 骨架**

创建 `meme-maker.html`，包含完整 CSS 变量（与 index.html 一致）、字体引入、网格背景、分屏布局结构：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Meme Maker — TOOLBOX</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
<style>
:root {
  --bg: #f0f4ff;
  --surface: #ffffff;
  --surface2: #f7f9ff;
  --border: #dde4f5;
  --border2: #c8d4f0;
  --blue: #0057ff;
  --blue2: #00aaff;
  --pink: #ff2d78;
  --green: #00c97a;
  --purple: #7c3aed;
  --text: #0a0e1a;
  --text2: #4a5578;
  --text3: #8896b8;
  --mono: 'Space Mono', monospace;
  --sans: 'Syne', sans-serif;
}
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html, body { height: 100%; background: var(--bg); color: var(--text); font-family: var(--sans); overflow: hidden; }

/* grid bg */
body::before {
  content: '';
  position: fixed; inset: 0;
  background-image:
    linear-gradient(rgba(0,87,255,0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0,87,255,0.05) 1px, transparent 1px);
  background-size: 44px 44px;
  pointer-events: none; z-index: 0;
}

/* layout */
.app { position: relative; z-index: 1; display: flex; flex-direction: column; height: 100vh; }

/* header */
.header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 16px 32px; border-bottom: 1px solid var(--border);
  background: rgba(240,244,255,0.85); backdrop-filter: blur(12px);
  flex-shrink: 0;
}
.header-left { display: flex; align-items: center; gap: 16px; }
.back-btn {
  font-family: var(--mono); font-size: 11px; color: var(--text3);
  text-decoration: none; letter-spacing: 0.1em;
  border: 1px solid var(--border2); padding: 6px 14px;
  background: var(--surface); transition: all 0.15s;
}
.back-btn:hover { color: var(--blue); border-color: var(--blue); }
.header-title { font-family: var(--sans); font-size: 18px; font-weight: 800; letter-spacing: -0.02em; }
.header-badge {
  font-family: var(--mono); font-size: 10px; color: var(--pink);
  border: 1px solid var(--pink); padding: 3px 10px; letter-spacing: 0.1em;
}

/* main split */
.main { display: flex; flex: 1; overflow: hidden; }

/* left panel */
.panel {
  width: 320px; flex-shrink: 0;
  border-right: 1px solid var(--border);
  background: var(--surface);
  overflow-y: auto; padding: 24px;
  display: flex; flex-direction: column; gap: 20px;
}

.section-label {
  font-family: var(--mono); font-size: 10px; color: var(--text3);
  letter-spacing: 0.15em; margin-bottom: 10px;
}

/* upload zone */
.upload-zone {
  border: 2px dashed var(--border2); padding: 32px 16px;
  text-align: center; cursor: pointer; transition: all 0.2s;
  background: var(--surface2); position: relative;
}
.upload-zone:hover, .upload-zone.drag-over {
  border-color: var(--blue); background: rgba(0,87,255,0.04);
}
.upload-zone input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }
.upload-icon { font-size: 28px; margin-bottom: 8px; }
.upload-hint { font-family: var(--mono); font-size: 11px; color: var(--text3); }
.upload-preview { width: 100%; aspect-ratio: 1; object-fit: cover; display: none; }

/* text input */
.text-input {
  width: 100%; padding: 12px 14px;
  border: 1px solid var(--border2); background: var(--surface2);
  font-family: var(--sans); font-size: 15px; color: var(--text);
  resize: vertical; min-height: 80px; outline: none; transition: border 0.15s;
}
.text-input:focus { border-color: var(--blue); }

/* generate btn */
.gen-btn {
  width: 100%; padding: 14px;
  background: var(--blue); color: #fff;
  font-family: var(--mono); font-size: 12px; letter-spacing: 0.12em;
  border: none; cursor: pointer; transition: all 0.15s;
}
.gen-btn:hover { background: #0044cc; }
.gen-btn:disabled { background: var(--border2); cursor: not-allowed; }

/* right grid */
.grid-area {
  flex: 1; overflow-y: auto; padding: 24px;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 16px; align-content: start;
}

.meme-card {
  background: var(--surface); border: 1px solid var(--border);
  overflow: hidden; transition: transform 0.15s, box-shadow 0.15s;
}
.meme-card:hover { transform: translateY(-2px); box-shadow: 0 8px 24px rgba(0,87,255,0.1); }

.meme-canvas { width: 100%; display: block; }

.card-footer {
  padding: 10px 12px; display: flex; align-items: center; justify-content: space-between;
  border-top: 1px solid var(--border);
}
.style-name { font-family: var(--mono); font-size: 10px; color: var(--text3); letter-spacing: 0.08em; }
.dl-btn {
  font-family: var(--mono); font-size: 10px; color: var(--blue);
  background: none; border: 1px solid var(--blue); padding: 4px 12px;
  cursor: pointer; letter-spacing: 0.08em; transition: all 0.15s;
}
.dl-btn:hover { background: var(--blue); color: #fff; }

/* empty state */
.empty-state {
  grid-column: 1/-1; display: flex; flex-direction: column;
  align-items: center; justify-content: center; min-height: 400px;
  color: var(--text3); font-family: var(--mono); font-size: 12px;
  letter-spacing: 0.1em; gap: 12px;
}
.empty-icon { font-size: 48px; opacity: 0.3; }
</style>
</head>
<body>
<div class="app">
  <header class="header">
    <div class="header-left">
      <a href="index.html" class="back-btn">← BACK</a>
      <div class="header-title">MEME MAKER</div>
      <div class="header-badge">TOOL_003</div>
    </div>
    <div style="font-family:var(--mono);font-size:10px;color:var(--text3);letter-spacing:0.1em;">BATCH · LOCAL · FREE</div>
  </header>

  <div class="main">
    <!-- Left Panel -->
    <div class="panel">
      <div>
        <div class="section-label">01 / UPLOAD IMAGE</div>
        <div class="upload-zone" id="uploadZone">
          <input type="file" id="fileInput" accept="image/*">
          <div class="upload-icon">🖼️</div>
          <div class="upload-hint">CLICK OR DRAG IMAGE HERE</div>
        </div>
        <img class="upload-preview" id="preview" alt="preview">
      </div>

      <div>
        <div class="section-label">02 / ENTER TEXT</div>
        <textarea class="text-input" id="memeText" placeholder="输入表情包文字..."></textarea>
      </div>

      <button class="gen-btn" id="genBtn" disabled>GENERATE MEMES →</button>
    </div>

    <!-- Right Grid -->
    <div class="grid-area" id="gridArea">
      <div class="empty-state">
        <div class="empty-icon">🎭</div>
        <div>UPLOAD IMAGE + ENTER TEXT TO START</div>
      </div>
    </div>
  </div>
</div>
<script>
// JS will be added in Task 2
</script>
</body>
</html>
```

- [ ] **Step 2: 在浏览器中打开验证布局**

```bash
open meme-maker.html
```

预期：分屏布局正常，左侧面板宽 320px，右侧显示 empty state，header 有返回按钮。

- [ ] **Step 3: Commit**

```bash
git add meme-maker.html
git commit -m "feat: add meme-maker page skeleton and layout"
```

---

### Task 2: 图片上传与预览逻辑

**Files:**
- Modify: `meme-maker.html` — 替换 `<script>` 内容

- [ ] **Step 1: 实现上传与拖拽逻辑**

将 `<script>` 标签内容替换为：

```js
const fileInput = document.getElementById('fileInput');
const uploadZone = document.getElementById('uploadZone');
const preview = document.getElementById('preview');
const memeText = document.getElementById('memeText');
const genBtn = document.getElementById('genBtn');
const gridArea = document.getElementById('gridArea');

let sourceImage = null;

function loadImage(file) {
  if (!file || !file.type.startsWith('image/')) return;
  const reader = new FileReader();
  reader.onload = e => {
    const img = new Image();
    img.onload = () => {
      sourceImage = img;
      preview.src = e.target.result;
      preview.style.display = 'block';
      uploadZone.querySelector('.upload-icon').style.display = 'none';
      uploadZone.querySelector('.upload-hint').style.display = 'none';
      checkReady();
    };
    img.src = e.target.result;
  };
  reader.readAsDataURL(file);
}

fileInput.addEventListener('change', e => loadImage(e.target.files[0]));

uploadZone.addEventListener('dragover', e => { e.preventDefault(); uploadZone.classList.add('drag-over'); });
uploadZone.addEventListener('dragleave', () => uploadZone.classList.remove('drag-over'));
uploadZone.addEventListener('drop', e => {
  e.preventDefault();
  uploadZone.classList.remove('drag-over');
  loadImage(e.dataTransfer.files[0]);
});

memeText.addEventListener('input', checkReady);

function checkReady() {
  genBtn.disabled = !(sourceImage && memeText.value.trim());
}

genBtn.addEventListener('click', generateMemes);
```

- [ ] **Step 2: 验证上传功能**

打开页面，上传一张图片，确认：
- 预览图显示
- 输入文字后 Generate 按钮变为可点击状态

- [ ] **Step 3: Commit**

```bash
git add meme-maker.html
git commit -m "feat: implement image upload and drag-drop with preview"
```

---

### Task 3: 12 种风格定义与 Canvas 渲染引擎

**Files:**
- Modify: `meme-maker.html` — 在 Task 2 的 script 后追加

- [ ] **Step 1: 定义 12 种风格配置**

在 `generateMemes` 函数前追加风格数组：

```js
const STYLES = [
  // 位置 × 字体 × 颜色 × 滤镜 × 排版模板
  { name: 'CLASSIC TOP',    pos: 'top',    font: 'Impact',       color: '#fff', stroke: '#000', strokeW: 4, filter: null,         bg: null },
  { name: 'CLASSIC BOTTOM', pos: 'bottom', font: 'Impact',       color: '#fff', stroke: '#000', strokeW: 4, filter: null,         bg: null },
  { name: 'SUBTITLE BAR',   pos: 'bottom', font: 'Syne',         color: '#fff', stroke: null,   strokeW: 0, filter: null,         bg: 'rgba(0,0,0,0.65)' },
  { name: 'TITLE BAR',      pos: 'top',    font: 'Syne',         color: '#fff', stroke: null,   strokeW: 0, filter: null,         bg: 'rgba(0,0,0,0.65)' },
  { name: 'CENTER BOLD',    pos: 'center', font: 'Impact',       color: '#ff2d78', stroke: '#fff', strokeW: 3, filter: null,      bg: null },
  { name: 'B&W CLASSIC',    pos: 'bottom', font: 'Impact',       color: '#fff', stroke: '#000', strokeW: 4, filter: 'grayscale', bg: null },
  { name: 'VINTAGE',        pos: 'bottom', font: 'Georgia',      color: '#f5e6c8', stroke: '#5a3a1a', strokeW: 2, filter: 'sepia', bg: null },
  { name: 'NEON',           pos: 'top',    font: 'Impact',       color: '#00ffcc', stroke: '#0057ff', strokeW: 3, filter: null,   bg: null },
  { name: 'HIGH CONTRAST',  pos: 'center', font: 'Impact',       color: '#ffff00', stroke: '#000', strokeW: 5, filter: 'contrast', bg: null },
  { name: 'MINIMAL',        pos: 'bottom', font: 'Space Mono',   color: '#fff', stroke: null,   strokeW: 0, filter: null,         bg: 'rgba(0,87,255,0.75)' },
  { name: 'DARK OVERLAY',   pos: 'center', font: 'Syne',         color: '#fff', stroke: null,   strokeW: 0, filter: 'darken',     bg: 'rgba(0,0,0,0.45)' },
  { name: 'COMIC',          pos: 'top',    font: 'Impact',       color: '#000', stroke: '#fff', strokeW: 3, filter: null,         bg: 'rgba(255,255,0,0.85)' },
];
```

- [ ] **Step 2: 实现 Canvas 渲染函数**

追加 `renderMeme` 函数：

```js
function applyFilter(ctx, filter, w, h) {
  if (!filter) return;
  const imageData = ctx.getImageData(0, 0, w, h);
  const d = imageData.data;
  for (let i = 0; i < d.length; i += 4) {
    if (filter === 'grayscale') {
      const g = d[i]*0.299 + d[i+1]*0.587 + d[i+2]*0.114;
      d[i] = d[i+1] = d[i+2] = g;
    } else if (filter === 'sepia') {
      const r = d[i], g = d[i+1], b = d[i+2];
      d[i]   = Math.min(255, r*0.393 + g*0.769 + b*0.189);
      d[i+1] = Math.min(255, r*0.349 + g*0.686 + b*0.168);
      d[i+2] = Math.min(255, r*0.272 + g*0.534 + b*0.131);
    } else if (filter === 'contrast') {
      const factor = 1.8;
      d[i]   = Math.min(255, Math.max(0, (d[i]   - 128) * factor + 128));
      d[i+1] = Math.min(255, Math.max(0, (d[i+1] - 128) * factor + 128));
      d[i+2] = Math.min(255, Math.max(0, (d[i+2] - 128) * factor + 128));
    } else if (filter === 'darken') {
      d[i] *= 0.6; d[i+1] *= 0.6; d[i+2] *= 0.6;
    }
  }
  ctx.putImageData(imageData, 0, 0);
}

function wrapText(ctx, text, maxWidth) {
  const words = text.split('');
  // For CJK, split by char; for Latin, split by word
  const isLatin = /^[\x00-\x7F\s]+$/.test(text);
  const tokens = isLatin ? text.split(' ') : text.split('');
  const lines = [];
  let line = '';
  for (const token of tokens) {
    const test = line + (isLatin && line ? ' ' : '') + token;
    if (ctx.measureText(test).width > maxWidth && line) {
      lines.push(line);
      line = token;
    } else {
      line = test;
    }
  }
  if (line) lines.push(line);
  return lines;
}

function renderMeme(canvas, img, text, style) {
  const SIZE = 480;
  canvas.width = SIZE;
  canvas.height = SIZE;
  const ctx = canvas.getContext('2d');

  // Draw image (cover fit)
  const scale = Math.max(SIZE / img.width, SIZE / img.height);
  const sw = img.width * scale, sh = img.height * scale;
  const sx = (SIZE - sw) / 2, sy = (SIZE - sh) / 2;
  ctx.drawImage(img, sx, sy, sw, sh);

  // Apply filter
  applyFilter(ctx, style.filter, SIZE, SIZE);

  // Text setup
  const fontSize = Math.round(SIZE * 0.085);
  const fontStr = `${style.font === 'Impact' ? 'bold ' : ''}${fontSize}px "${style.font}", Impact, sans-serif`;
  ctx.font = fontStr;
  ctx.textAlign = 'center';

  const padding = 20;
  const maxW = SIZE - padding * 2;
  const lines = wrapText(ctx, text, maxW);
  const lineH = fontSize * 1.2;
  const blockH = lines.length * lineH;

  // Position
  let startY;
  if (style.pos === 'top') startY = padding + fontSize;
  else if (style.pos === 'bottom') startY = SIZE - padding - blockH + fontSize;
  else startY = (SIZE - blockH) / 2 + fontSize; // center

  // Background bar
  if (style.bg) {
    const barPad = 12;
    ctx.fillStyle = style.bg;
    ctx.fillRect(0, startY - fontSize - barPad, SIZE, blockH + barPad * 2);
  }

  // Draw text lines
  lines.forEach((line, i) => {
    const y = startY + i * lineH;
    if (style.strokeW > 0 && style.stroke) {
      ctx.strokeStyle = style.stroke;
      ctx.lineWidth = style.strokeW;
      ctx.lineJoin = 'round';
      ctx.strokeText(line, SIZE / 2, y);
    }
    ctx.fillStyle = style.color;
    ctx.fillText(line, SIZE / 2, y);
  });
}
```

- [ ] **Step 3: Commit**

```bash
git add meme-maker.html
git commit -m "feat: add 12 style definitions and canvas render engine"
```

---

### Task 4: 批量生成与下载功能

**Files:**
- Modify: `meme-maker.html` — 追加 generateMemes 函数

- [ ] **Step 1: 实现 generateMemes 函数**

追加：

```js
function generateMemes() {
  gridArea.innerHTML = '';

  STYLES.forEach((style, i) => {
    const card = document.createElement('div');
    card.className = 'meme-card';

    const canvas = document.createElement('canvas');
    canvas.className = 'meme-canvas';
    renderMeme(canvas, sourceImage, memeText.value.trim(), style);

    const footer = document.createElement('div');
    footer.className = 'card-footer';

    const label = document.createElement('div');
    label.className = 'style-name';
    label.textContent = style.name;

    const dlBtn = document.createElement('button');
    dlBtn.className = 'dl-btn';
    dlBtn.textContent = '↓ SAVE';
    dlBtn.addEventListener('click', () => downloadCanvas(canvas, `meme-${style.name.toLowerCase().replace(/\s+/g,'-')}.png`));

    footer.appendChild(label);
    footer.appendChild(dlBtn);
    card.appendChild(canvas);
    card.appendChild(footer);
    gridArea.appendChild(card);
  });
}

function downloadCanvas(canvas, filename) {
  canvas.toBlob(blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
    URL.revokeObjectURL(url);
  }, 'image/png');
}
```

- [ ] **Step 2: 端到端验证**

1. 打开 `meme-maker.html`
2. 上传任意图片
3. 输入文字，点击 GENERATE MEMES
4. 确认 12 张变体卡片出现，每张样式不同
5. 点击任意一张的 ↓ SAVE，确认下载 PNG 文件

- [ ] **Step 3: Commit**

```bash
git add meme-maker.html
git commit -m "feat: implement batch generate and per-card download"
```

---

### Task 5: 更新首页工具卡片

**Files:**
- Modify: `index.html` — 将 TOOL_003 占位卡片替换为 Meme Maker 卡片

- [ ] **Step 1: 替换 TOOL_003 占位卡片**

找到 index.html 中 `<div class="tool-card soon" data-color="purple">` 整块，替换为：

```html
<a href="meme-maker.html" class="tool-card" data-color="pink">
  <div class="card-num">TOOL_003</div>
  <div class="card-icon">
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
      <rect x="3" y="3" width="18" height="18" rx="2"/>
      <path d="M3 9h18"/>
      <circle cx="8" cy="6" r="1" fill="currentColor"/>
      <circle cx="12" cy="6" r="1" fill="currentColor"/>
      <path d="M7 14s1 2 5 2 5-2 5-2"/>
      <circle cx="9" cy="13" r="1" fill="currentColor"/>
      <circle cx="15" cy="13" r="1" fill="currentColor"/>
    </svg>
  </div>
  <div class="card-title">Meme Maker</div>
  <div class="card-desc">上传图片 + 输入文字，一键批量生成 12 种风格表情包，本地处理无需上传。</div>
  <div class="card-tags">
    <span class="tag">IMAGE</span>
    <span class="tag">MEME</span>
    <span class="tag">BATCH</span>
  </div>
  <div class="card-arrow">→</div>
</a>
```

- [ ] **Step 2: 验证首页**

打开 `index.html`，确认 TOOL_003 卡片显示为 Meme Maker，点击可跳转到 `meme-maker.html`。

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add meme-maker card to homepage"
```

---

## Self-Review

**Spec coverage:**
- ✅ 上传图片（拖拽 + 点击）
- ✅ 输入文字
- ✅ 12 种风格变体（字体×位置×滤镜×排版模板）
- ✅ 每张单独下载（Canvas toBlob → PNG）
- ✅ 首页入口卡片
- ✅ 与现有 toolbox 风格一致

**Placeholder scan:** 无 TBD/TODO。

**Type consistency:** `renderMeme(canvas, img, text, style)` 在 Task 3 定义，Task 4 调用，签名一致。`downloadCanvas(canvas, filename)` 同理。
