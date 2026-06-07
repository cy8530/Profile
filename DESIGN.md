# 个人数据小馆 · 设计文档

## 概述

「个人数据小馆」是一个单文件 HTML 个人主页，通过「日间 / 夜间」双模式切换实现**专业简历**与**兴趣生活**两个完全不同视觉风格的页面，无需多页面跳转。

### 文件清单

| 文件 | 用途 |
|---|---|
| `index.html` | 主页面，包含所有 CSS / HTML / JS，完全自包含 |
| `fetch_steam.py` | 通过代理拉取 Steam API 数据并下载游戏图标 |
| `steam_games.json` | Steam 游戏库的本地 JSON 缓存（完整 70+ 款） |
| `icons/` | 游戏图标本地存储目录（供 Python 脚本下载用） |
| `fetch_netease.py` | 通过本地 API 服务器拉取网易云听歌排行 |
| `netease_songs.json` | 网易云听歌排行本地 JSON 缓存（100 首） |
| `netease_cookie.txt` | 网易云登录 Cookie 缓存（免重复扫码） |
| `DESIGN.md` | 本文档 |

---

## 设计理念

### 核心思路：一页两面

同一份自我介绍区作为通用模块置于页面顶部，下方主体内容根据当前模式**完全替换**。切换通过 `body` 标签上的 `.night` class 控制，所有视觉变化均由 CSS 自定义属性（Custom Properties）驱动，无需 JavaScript 操作 DOM 样式。

### 视觉策略

| 维度 | 日间模式 | 夜间模式 |
|---|---|---|
| **定位** | 正式个人简历 | 兴趣生活馆 |
| **色调** | 米白底 + 藏蓝点缀 | 深色底 + 霓虹暖色 |
| **卡片** | 实色白色卡片 + 细阴影 | 毛玻璃（`backdrop-filter`）+ 半透明 |
| **字体** | 系统无衬线，端正 | 同字体，但氛围更轻松 |
| **背景** | 纯色留白 | 多层径向渐变 + 预留自定义背景图位 |

### 双主题切换机制

```
┌──────────────────────────────────────────────┐
│  :root (日间)     │  body.night (夜间)        │
│  --bg: #fafaf7    │  --bg: #0d1117           │
│  --accent: #2c3e6b│  --accent: #ff6b6b       │
│  --text: #1e1e1e  │  --text: #e6edf3         │
│  ...              │  ...                     │
└──────────────────────────────────────────────┘
```

所有组件引用 CSS 变量，不写死颜色。切换 class 后浏览器自动重新计算所有样式，0 行 JS 操作颜色。

---

## 页面结构

```
┌─────────────────────────────────────────┐
│  Museum Header                           │
│  「个人数据小馆」logo + 副标题             │
├─────────────────────────────────────────┤
│  01 · 自我介绍区（共享模块）               │
│  ┌──────┬──────────────────────────────┐ │
│  │ 头像  │ 昵称 + 标签 + 个人简介        │ │
│  │ (交互)│                              │ │
│  └──────┴──────────────────────────────┘ │
├─────────────────────────────────────────┤
│  02 · 数据可视化区                        │
│  ┌─ 日间面板 (day-content) ────────────┐ │
│  │ 个人总结 · 教育背景 · 项目经历 ·     │ │
│  │ 荣誉奖项                            │ │
│  └────────────────────────────────────┘ │
│  ┌─ 夜间面板 (night-content) ──────────┐ │
│  │ Steam 游戏 · 读书书单 · 歌单        │ │
│  └────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│  03 · 交互彩蛋区                         │
│  ┌─ 日间面板 (day-panel) ──────────────┐ │
│  │ 正式简历时间线                       │ │
│  └────────────────────────────────────┘ │
│  ┌─ 夜间面板 (night-panel) ────────────┐ │
│  │ 书单/歌单/游戏兴趣卡片               │ │
│  └────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│  Footer + 切换按钮                       │
└─────────────────────────────────────────┘
```

### 日间面板内容

| 模块 | 内容来源 | 展示形式 |
|---|---|---|
| **个人总结** | LaTeX 简历 `\section{个人总结}` | 时间线列表 |
| **教育背景** | LaTeX 简历 `\section{教育背景}` | 时间线列表 |
| **项目经历** | LaTeX 简历 `\section{项目经历}`（3 个项目） | 时间线列表，含详细描述 |
| **荣誉奖项** | LaTeX 简历 | 时间线列表 |
| **自我介绍区** | 实时数据 | 头像 + 标签 + 简介文案 |

### 夜间面板内容

| 模块 | 数据来源 | 展示形式 |
|---|---|---|
| **Steam 游戏库** | `gamesData` 内嵌数组（10 款精选） | 竖排列表：左侧游戏图标 + 中间进度条 + 右侧时长 |
| **读书书单** | `bookPages` 内嵌数组（3 页） | 翻页式卡片：彩色书脊 + 书名/作者/代表句 |
| **歌单** | `tracksData` 内嵌数组（Top 20） | 编号列表 + 进度条 + 播放次数 + 点击播放 |

#### 游戏列表交互

- 默认展示时长前 5 的游戏
- 「展开全部（共 10 款）」按钮 → 折叠区滑出后 5 款
- 进度条以最长游玩时间（CS2，1091.2h）为满格基准等比缩放
- 已排除「Love Is All Around」

#### 游戏图标加载策略

图标通过 base64 编码直接内嵌于 `gamesData` 的 `iconB64` 字段中，零外部依赖。无论页面在本地、GitHub Pages、还是其他电脑打开，图标均能正常显示。

```javascript
{ name: 'Counter-Strike 2', appid: 730, hours: 1091.2,
  iconB64: 'data:image/jpeg;base64,/9j/4AAQ...' }
```

fallback 链（极简）：base64 图片 → 加载失败 → 显示 emoji 占位。总计 10 款游戏图标约 9KB，base64 编码后约 12KB，对页面体积影响很小。

#### 歌单交互（点击播放，双层 fallback）

点击歌单中任意歌曲行触发播放，策略如下：

1. **本地 API 可用** → 调用 `localhost:3000/song/url?id={id}` 获取 MP3 直链，通过自建迷你播放器播放。底部显示歌曲名/艺术家，播放/暂停按钮。3 秒超时后自动降级。
2. **本地 API 不可达**（无服务、其他电脑、GitHub Pages） → 自动嵌入网易云官方外链 iframe 播放器（`https://music.163.com/outchain/player?type=2&id={id}`），该播放器跨域可用，无需任何本地服务。
3. **都不行** → 静默失败，无弹窗报错。

播放状态视觉反馈：
- 当前播放行左侧高亮边框（`--accent` 色）+ 歌名变色
- 切歌时自动清理旧播放器（停止 audio / 移除 iframe）
- 同一首歌重复点击：本地模式切换播放/暂停，iframe 模式不打断

---

## 交互机制

### 头像交互（核心彩蛋）

| 行为 | 反馈 |
|---|---|
| **鼠标悬停** | 弹出轮换吐槽气泡（8 条文案库） |
| **点击** | 水波纹动画 + 顶部 5 颗能量点逐颗点亮 |
| **2 秒内连点 5 次** | 触发日夜模式切换 |
| **连点不足 5 次** | 显示递增吐槽，2 秒后自动重置计数 |

### 日夜切换（四种触发路径）

1. 头像连续点击 5 次
2. 页面右上角 ☀️/🌙 图标点击
3. 页脚「切换日夜模式」按钮
4. 键盘按 `N` 键（避开 input / textarea 聚焦状态）

### 切换时的变化

- `body` 添加/移除 `.night` class
- CSS 变量全部切换 → 背景、文字、卡片、阴影、边框同步变化
- `day-content` 和 `night-content` 通过 opacity + transform 交叉淡入淡出
- 页面副标题文案变更
- 右上角图标 ☀️ ↔ 🌙

---

## 技术实现

### 技术栈

- **纯 HTML + CSS + Vanilla JS**，零框架依赖
- Chart.js（CDN 引入，预留给未来数据可视化）
- Steam Web API（通过 Python 脚本拉取，非前端直连）
- NeteaseCloudMusicApi（Node.js 本地 API 服务，非前端直连）

### CSS 架构

```
RESET & BASE
DAY THEME (:root custom properties)
NIGHT THEME (body.night overrides)
  └─ 所有组件引用 CSS 变量，主题切换零 JS 开销

LAYOUT (.container)
SECTION TITLES (.section-label)
MUSEUM HEADER
MODE HINT (右上角切换图标)
SHARED: SELF-INTRO CARD
  └─ 头像、标签、简介，通用样式 + 主题覆写

DAY CONTENT
  └─ .resume-card + .timeline (简历卡片 + 时间线)

NIGHT CONTENT
  └─ .glass-card (毛玻璃卡片)
  └─ .game-list / .game-row (游戏竖排列表)
  └─ .books-grid / .book-card → .book-carousel / .book-page (翻页书单)
  └─ .playlist (歌单) + .mini-player (迷你播放器) + .ext-player (iframe 降级)

FOOTER
RESPONSIVE (@media max-width: 700px)
```

### JavaScript 架构

```
State: clickCount, clickTimer, currentSongId, useExtPlayer

DOM Refs: body, modeHint, footerBtn, headerSub,
          avatarWrap, avatarImg, avatarToast, clickDots,
          audioPlayer, miniPlayer, extPlayer, mpBtn, mpName, mpArtist

Data:
  gamesData[]     ← 自包含 (base64 图标 + 时长数据), 10 款
  bookPages[]     ← 自包含 (作者/书籍/引用), 3 页 12 本
  tracksData[]    ← 自包含 (歌名/歌手/播放次数/歌曲 ID), 20 首

Render Functions:
  renderGames()   → 游戏列表 + 展开/收起 + 图标渲染
  renderBooks()   → 翻页书单 + 触摸滑动
  renderPlaylist() → 歌单 + 展开/收起 + 点击播放(双层 fallback)

Toggle: toggleMode()
  → body.classList.toggle('night')
  → 更新 UI 文字和图标

Avatar Interaction:
  mouseenter → 轮换 toast
  click → ripple + 计数 + 5 次触发 toggleMode

Keyboard: keydown 'n' → toggleMode()

Audio:
  click song → fetch localhost:3000/song/url (3s timeout)
  success → <audio> 播放 + mini-player 显示
  failure → NetEase iframe player 嵌入
  no error alerts to user
```

### Steam 数据管道

```
Steam Web API
  ↓ (Python + 代理)
fetch_steam.py
  ├→ steam_games.json (JSON 缓存)
  ├→ icons/{appid}.jpg (游戏图标)
  └→ 打印 JS 数组格式
  ↓
手动取 Top 10 + base64 图标 → 写入 index.html
```

### 网易云数据管道

```
本地 Node.js 服务 (NeteaseCloudMusicApi)
  ↓ (localhost:3000)
fetch_netease.py
  ├→ QR 码登录 (如无缓存 Cookie)
  ├→ 轮询扫码状态
  ├→ 拉取全部时间听歌排行 (type=0)
  ├→ netease_songs.json (JSON 缓存, 100 首)
  └→ 打印 JS 数组格式
  ↓
手动取 Top 20 → 写入 index.html 的 tracksData

运行时播放:
  ┌─ 本地: fetch localhost:3000/song/url → MP3 → <audio>
  └─ 远程: NetEase iframe player ← 懒加载, 点击时触发
```

---

## 数据更新流程

### 更新 Steam 游戏数据

```bash
# 确保代理已开 (7892 端口)
python3 fetch_steam.py
```

脚本自动完成：
1. 拉取 Steam API 全部游戏数据
2. 覆盖保存 `steam_games.json`
3. 下载/更新所有游戏图标到 `icons/` 目录
4. 在终端打印 JS 数组格式

如需更新 `index.html` 中的游戏列表：
1. 运行脚本获取最新数据
2. 选取 Top N，使用 `python3 -c "import base64; ..."` 生成图标 base64
3. 替换 `gamesData` 数组

### 更新网易云听歌排行

```bash
# 确保本地 API 服务运行
node node_modules/neteasecloudmusicapi/app.js

# 拉取数据
python3 fetch_netease.py
```

脚本自动完成：
1. 检查本地 Cookie 缓存
2. 如过期 → 生成登录二维码 → 等待手机扫码授权
3. 拉取全部时间听歌排行（最多 100 首）
4. 保存 `netease_songs.json` + 更新 Cookie
5. 在终端打印 JS 数组格式（含歌曲 ID）

如需更新 `index.html` 中的歌单：
1. 运行脚本获取最新数据
2. 替换 `tracksData` 数组（保留 `id` 字段以支持播放）

### 更新书单

直接编辑 `index.html` 中的 `bookPages` 数组。

---

## 自包含性

页面设计目标为**单 HTML 文件可在任意环境打开**（本地文件、GitHub Pages、其他电脑），所有外部依赖均有降级策略：

| 功能 | 本地 | 外部环境 |
|---|---|---|
| Steam 游戏图标 | base64 内嵌 ✅ | base64 内嵌 ✅ |
| 歌单展示 | 内嵌数据 ✅ | 内嵌数据 ✅ |
| 歌单播放 | 本地 API → MP3 ✅ | 网易云 iframe ✅ |
| 书单 | 内嵌数据 ✅ | 内嵌数据 ✅ |
| Chart.js | CDN 加载 | CDN 加载（预留，未使用） |

唯一不可降级的外部依赖是 Chart.js CDN 引用（`<script src="...">`），但该库当前未在任何功能中使用，仅为未来扩展预留。

---

## 浏览器兼容性

- `backdrop-filter`（毛玻璃效果）需 Safari 9+ / Chrome 76+ / Firefox 103+
- `clamp()` 需 Safari 13+ / Chrome 79+ / Firefox 75+
- CSS Custom Properties 需 Safari 9.1+ / Chrome 49+ / Firefox 31+
- `AbortController`（fetch 超时）需 Safari 12.1+ / Chrome 66+ / Firefox 57+
- Chart.js 4.x 需 ES2020+ 浏览器
