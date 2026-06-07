# 个人档案 · 设计文档

## 概述

「个人档案」是一个单文件 HTML 个人主页（`profile.html`），通过「日间 / 夜间」双模式切换在一页内实现**专业简历**与**兴趣生活**两个完全不同视觉风格的页面，无需多页面跳转。

### 文件清单

| 文件 | 用途 |
|---|---|
| `profile.html` | 主页面，包含所有 CSS / HTML / JS，完全自包含 |
| `index.html` | 入口重定向 → `profile.html`（GitHub Pages 要求） |
| `fetch_steam.py` | 通过代理拉取 Steam API 数据并下载游戏图标 |
| `steam_games.json` | Steam 游戏库的本地 JSON 缓存（完整 70+ 款） |
| `icons/` | 游戏图标本地存储目录（供 Python 脚本下载用） |
| `fetch_netease.py` | 通过本地 API 服务器拉取网易云听歌排行 |
| `netease_songs.json` | 网易云听歌排行本地 JSON 缓存（100 首） |
| `netease_cookie.txt` | 网易云登录 Cookie 缓存（免重复扫码，不入库） |
| `DESIGN.md` | 本文档 |
| `.gitignore` | 排除敏感文件与本地工具脚本 |

---

## 设计理念

### 核心思路：一页两面

自我介绍区根据当前模式**切换内容**（日间展示专业背景，夜间展示爱好简介），下方主体内容**完全替换**。切换通过 `body` 标签上的 `.night` class 控制，所有视觉变化均由 CSS 自定义属性（Custom Properties）驱动。

### 视觉策略

| 维度 | 日间模式 | 夜间模式 |
|---|---|---|
| **定位** | 正式个人简历 | 兴趣生活馆 |
| **色调** | 米白底 + 藏蓝点缀 | 深色底 + 霓虹暖色 |
| **卡片** | 实色白色卡片 + 细阴影 | 毛玻璃（`backdrop-filter`，仅夜间启用）+ 半透明 |
| **字体** | 系统无衬线，端正 | 同字体，氛围更轻松 |
| **背景** | 纯色留白 | 多层径向渐变 + Canvas 星空粒子 |

### 双主题切换机制

```
┌──────────────────────────────────────────────┐
│  :root (日间)           body.night (夜间)     │
│  --bg: #fafaf7          --bg: #0d1117        │
│  --bg-card: #ffffff     --bg-card: rgba(...) │
│  --accent: #2c3e6b      --accent: #ff6b6b    │
│  --text: #1e1e1e        --text: #e6edf3      │
│  --glass-bg: transparent --glass-bg: rgba(...)│
│  ...                    ...                  │
└──────────────────────────────────────────────┘
```

所有组件引用 CSS 变量，不写死颜色。`body.night` 覆盖 `:root` 变量，0 行 JS 操作颜色。

### 主题持久化

通过 `localStorage` 保存用户选择的主题（key: `theme`，value: `"night"` / `"day"`）。页面加载时：

1. **`<head>` 阶段**：检查 localStorage，若为 night 则注入临时 `<style id="nightPreload">` 标签，直接设置 `body{background:#0d1117!important}`，防止渲染闪烁
2. **JS 初始化阶段**：`body.classList.add('night')` 激活完整夜间 CSS 变量，随即移除临时 preload 样式
3. **切换时**：`toggleMode()` 同步更新 localStorage

---

## 页面结构

```
┌─────────────────────────────────────────┐
│  Header                                  │
│  「个人档案」+ 英文副线 + 副标题           │
├─────────────────────────────────────────┤
│  01 · 自我介绍区（内容随模式切换）         │
│  ┌──────┬──────────────────────────────┐ │
│  │ 头像  │ 日间: 学校/专业/邮箱标签      │ │
│  │ (交互)│ 夜间: 游戏/阅读/音乐/运动标签  │ │
│  └──────┴──────────────────────────────┘ │
├─────────────────────────────────────────┤
│  02 · 数据展示区                          │
│  ┌─ 日间面板 (day-content) ────────────┐ │
│  │ 个人总结 · 教育背景 · 项目经历 ·     │ │
│  │ 荣誉奖项                            │ │
│  └────────────────────────────────────┘ │
│  ┌─ 夜间面板 (night-content) ──────────┐ │
│  │ Steam 游戏 · 读书书单 · 歌单        │ │
│  └────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│  Footer + 切换按钮                       │
└─────────────────────────────────────────┘
```

### 日间面板内容

| 模块 | 内容来源 | 展示形式 |
|---|---|---|
| **自我介绍** | 实时数据 | 头像 + 专业标签 + AI 方向简介 |
| **个人总结** | 简历 | 时间线列表 |
| **教育背景** | 简历 | 时间线列表 |
| **项目经历** | 简历（3 个项目） | 时间线列表，含详细描述 |
| **荣誉奖项** | 简历 | 时间线列表 |

### 夜间面板内容

| 模块 | 数据来源 | 展示形式 |
|---|---|---|
| **自我介绍** | 爱好数据 | 头像 + 爱好标签（游戏/阅读/音乐/运动）+ 简短爱好简介 |
| **Steam 游戏库** | `gamesData` 内嵌数组（10 款精选） | 竖排列表：游戏图标 + 进度条 + 时长 |
| **读书书单** | `bookPages` 内嵌数组（3 页） | 翻页式卡片：彩色书脊 + 书名/作者/代表句 |
| **歌单** | `tracksData` 内嵌数组（Top 20） | 编号列表 + 进度条 + 播放次数 + 点击播放 |

---

## 夜间模式视觉效果

### 星空背景（Canvas）
- 100 颗粒子，随机位置 + 闪烁 + 缓慢漂移
- 鼠标靠近 150px 内粒子被微弱吸引
- 通过 `MutationObserver` 监听 `body` class 变化自动启停

### 3D 倾斜
- 鼠标悬停毛玻璃卡片时，±3° rotateX/Y 跟随鼠标位置
- 离开时平滑回正

### 光标拖尾（Canvas）
- 最多 30 个拖尾点，跟随鼠标移动
- 红色调（`rgba(255,107,107,...)`），自动衰减消失

### 游戏进度条渐变
- 日间纯色，夜间使用 `linear-gradient(90deg, var(--accent), var(--accent-warm))`

### 音乐播放视觉反馈
- CSS 跳动条（4 根 `.mp-eq span`），`eqBounce` 动画错开延迟
- 当前播放歌曲行 `playingGlow` 动画（左侧边框光晕呼吸）

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

切换时 `body.classList.toggle('night')` 同时写入 `localStorage`。

### 切换时的变化

- `body` 添加/移除 `.night` class → CSS 变量全部切换
- 背景、文字、卡片、阴影、边框、毛玻璃同步变化
- `day-content` 和 `night-content` 通过 opacity + transform 交叉淡入淡出
- 自我介绍区标签与简介文案切换（日间专业 ↔ 夜间爱好）
- 页面副标题文案变更
- 右上角图标 ☀️ ↔ 🌙

---

## 歌单播放（三层降级）

点击歌单中任意歌曲行触发播放：

```
① 本地 API (localhost:3000/song/url?id={id})
   ↓ 3s 超时或失败
② 网易云官方外链 iframe (music.163.com/outchain/player?type=2&id={id})
   + 底部显示「在网易云音乐打开 ↗」链接
   ↓ iframe 无法播放
③ 用户点击链接 → 新标签页打开网易云官方歌曲页
   100% 可靠，不受网络环境限制
```

播放状态视觉反馈：
- 本地播放：迷你播放器显示歌曲信息 + 播放/暂停按钮 + CSS EQ 跳动条
- 当前播放行 `playingGlow` 动画 + 左侧 `--accent` 色边框
- 切歌时自动清理旧播放器（停止 audio / 移除 iframe）
- 同一首歌重复点击：本地模式切换播放/暂停，iframe 模式不打断

---

## 游戏列表交互

- 默认展示时长前 5 的游戏
- 「展开全部（共 10 款）」按钮 → 折叠区滑出后 5 款
- 进度条以最长游玩时间（CS2，1091.2h）为满格基准等比缩放
- 已排除「Love Is All Around」
- 图标通过 base64 内嵌于 `gamesData`，零外部依赖

---

## 技术实现

### 技术栈

- **纯 HTML + CSS + Vanilla JS**，零框架依赖
- Chart.js（CDN 引入，预留给未来数据可视化，当前未使用）
- Steam Web API（通过 Python 脚本拉取，非前端直连）
- NeteaseCloudMusicApi（Node.js 本地 API 服务，非前端直连）

### CSS 架构

```
RESET & BASE
DAY THEME (:root custom properties)
NIGHT THEME (body.night overrides)
  └─ 所有组件引用 CSS 变量，主题切换零 JS 开销
  └─ backdrop-filter 仅在 body.night .glass-card 启用（日间为 none）
  └─ 夜间固定层 (.night-bg, #starCanvas, #trailCanvas) 日间 visibility: hidden

LAYOUT (.container)
SECTION TITLES (.section-label)
HEADER (.museum-header → Personal Archive)
MODE HINT (右上角切换图标 .mode-hint)
SHARED: SELF-INTRO CARD (.intro-card)
  └─ 头像、标签、简介，日间/夜间通过 .day-only / .night-only 类切换

DAY CONTENT (.day-content)
  └─ .resume-card + .timeline (简历卡片 + 时间线)

NIGHT CONTENT (.night-content)
  └─ .glass-card (毛玻璃卡片，仅夜间启用 backdrop-filter)
  └─ .game-list / .game-row (游戏竖排列表)
  └─ .books-grid / .book-card → .book-carousel / .book-page (翻页书单)
  └─ .playlist (歌单) + .mini-player (迷你播放器) + .ext-player (iframe)

FOOTER
RESPONSIVE (@media max-width: 700px)
```

### JavaScript 架构

```
State:
  clickCount, clickTimer    — 头像连点计数
  currentId, useExt         — 当前播放歌曲 ID / 是否使用 iframe 模式

Theme:
  saveTheme(night)          — 写入 localStorage
  loadTheme()               — 读取 localStorage
  toggleMode()              — body.classList.toggle('night') + 更新 UI

DOM Refs:
  body, modeHint, footerBtn, headerSub,
  avatarWrap, avatarImg, avatarToast, clickDots,
  audioPlayer, miniPlayer, extPlayer, mpBtn, mpName, mpArtist, mpEq

Data (全部内嵌):
  gamesData[]     ← 10 款游戏 (base64 图标 + 时长 + appid)
  bookPages[]     ← 3 页 12 本书 (作者/书名/引用/颜色)
  tracksData[]    ← 20 首歌曲 (歌名/歌手/播放次数/歌曲 ID)

Render Functions:
  renderGames()     → 游戏列表 + 展开/收起 + 图标渲染 + fallback emoji
  renderBooks()     → 翻页书单 + 触摸滑动
  renderPlaylist()  → 歌单 + 展开/收起 + 点击播放(三层降级)

Night Visual Effects (IIFE):
  initStarfield()   → Canvas 星空 + MutationObserver 自动启停
  initTilt()        → 3D 倾斜跟随鼠标
  initTrail()       → Canvas 光标拖尾 + MutationObserver 自动启停

Avatar Interaction:
  mouseenter → 轮换 toast
  click → ripple + 计数 + 5 次触发 toggleMode

Keyboard: keydown 'n' → toggleMode()
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
手动取 Top 10 + base64 图标 → 写入 profile.html 的 gamesData
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
  └→ 打印 JS 数组格式 (含歌曲 ID)
  ↓
手动取 Top 20 → 写入 profile.html 的 tracksData

运行时播放 (三层降级):
  ┌─ ① 本地: fetch localhost:3000/song/url → MP3 → <audio>
  ├─ ② 远程: NetEase iframe player + 网易云直达链接
  └─ ③ 兜底: 新标签页打开 music.163.com/song?id={id}
```

---

## 数据更新流程

### 更新 Steam 游戏数据

```bash
python3 fetch_steam.py
```

1. 拉取 Steam API 全部游戏数据
2. 覆盖保存 `steam_games.json`
3. 下载/更新所有游戏图标到 `icons/` 目录
4. 在终端打印 JS 数组格式
5. 手动选取 Top N，base64 编码图标，替换 `profile.html` 中的 `gamesData`

### 更新网易云听歌排行

```bash
# 确保本地 API 服务运行
node node_modules/neteasecloudmusicapi/app.js

# 拉取数据
python3 fetch_netease.py
```

1. 检查本地 Cookie 缓存
2. 如过期 → 生成登录二维码 → 等待手机扫码授权
3. 拉取全部时间听歌排行（最多 100 首）
4. 保存 `netease_songs.json` + 更新 Cookie
5. 在终端打印 JS 数组格式（含歌曲 ID）
6. 手动替换 `profile.html` 中的 `tracksData`（保留 `id` 字段以支持播放）

### 更新书单

直接编辑 `profile.html` 中的 `bookPages` 数组。

---

## 自包含性

页面设计目标为**单 HTML 文件可在任意环境打开**（本地文件、GitHub Pages、其他电脑），所有外部依赖均有降级策略：

| 功能 | 本地 | 外部环境 |
|---|---|---|
| Steam 游戏图标 | base64 内嵌 ✅ | base64 内嵌 ✅ |
| 歌单展示 | 内嵌数据 ✅ | 内嵌数据 ✅ |
| 歌单播放 | 本地 API → MP3 ✅ | iframe 外链 → 网易云直达链接 ✅ |
| 书单 | 内嵌数据 ✅ | 内嵌数据 ✅ |
| 夜间视觉效果 | Canvas 绘制 ✅ | Canvas 绘制 ✅ |

唯一不可降级的外部依赖是 Chart.js CDN，但该库当前未在任何功能中使用，仅为未来扩展预留。

---

## GitHub Pages 部署

### 文件结构

```
仓库根目录 /
  ├── profile.html    ← 主页面
  ├── index.html      ← 入口，meta refresh 重定向到 profile.html
  ├── DESIGN.md       ← 本文档
  └── .gitignore      ← 排除敏感文件（Cookie、本地脚本、原始 JSON）
```

### 访问地址

- 根路径：`https://<username>.github.io/<repo>/` → 自动跳转到 `profile.html`
- 直接访问：`https://<username>.github.io/<repo>/profile.html`

### 配置

1. 仓库 Settings → Pages → Source: Deploy from a branch → Branch: main → / (root)
2. 等待 1-2 分钟部署完成

---

## Safari 兼容性处理

Safari 对 GPU 合成层的处理与 Chrome 不同，以下措施确保日间模式不出现深色渲染残块：

| 措施 | 说明 |
|---|---|
| `backdrop-filter` 仅夜间启用 | `.glass-card` 日间设为 `none`，`body.night .glass-card` 才启用 `blur(8px)` |
| 夜间固定层添加 `visibility: hidden` | `.night-bg`、`#starCanvas`、`#trailCanvas` 日间 `visibility: hidden`，配合 transition 延迟确保不参与合成 |
| 光标拖尾使用 Canvas 非 DOM | 避免大量 DOM 元素 + `box-shadow` 造成的合成层爆炸 |

---

## 浏览器兼容性

- `backdrop-filter` 需 Safari 9+ / Chrome 76+ / Firefox 103+
- `clamp()` 需 Safari 13+ / Chrome 79+ / Firefox 75+
- CSS Custom Properties 需 Safari 9.1+ / Chrome 49+ / Firefox 31+
- `AbortController`（fetch 超时）需 Safari 12.1+ / Chrome 66+ / Firefox 57+
- Canvas 2D API 全平台支持
- `MutationObserver` 全平台支持
