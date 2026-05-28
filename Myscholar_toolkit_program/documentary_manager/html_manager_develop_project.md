# HTML Manager — 跨平台桌面 HTML 管理器开发计划

> 创建日期：2026-05-27 | 状态：待 Review

---

## 一、项目目标

开发一款跨平台（Windows / macOS / Linux）桌面应用，具备：

1. **可交互的图形化界面** — 文件管理面板 + HTML 渲染预览面板
2. **导入 HTML 文件** — 支持拖拽导入、文件夹导入、批量导入
3. **点击即渲染** — 选中 HTML 文件后，在应用内置渲染面板中直接展示完整网页效果
4. **全语言支持** — HTML5、CSS3、JavaScript/TypeScript、Three.js (WebGL)、WebAssembly 等必须全部可用

---

## 二、竞品调研结论

### 2.1 无直接竞品

经过对 GitHub 和 npm 生态的深入搜索，**不存在**一款专门为"导入 HTML 文件并内部渲染管理"设计的桌面应用。现有最接近的项目：

| 项目 | 类型 | 差距 |
|------|------|------|
| [Xplorer](https://github.com/kimlimjustin/xplorer) (Tauri) | 通用文件管理器 | 仅支持代码语法高亮预览，**不支持 HTML 渲染预览** |
| [Filedime](https://deepwiki.com/visnkmr/filedime) (Tauri) | AI 文件管理器 | 侧重 AI 文档查询，无 HTML 渲染 |
| [TheBrowserLab](https://github.com/icurtis1/thebrowserlab) | 浏览器端 3D 编辑器 | 纯 Web 应用，非桌面应用，无文件管理功能 |
| [Electrobun](https://github.com/blackboardsh/electrobun) | 桌面框架 | 是框架而非应用，需自行开发 |

**结论：这是一个蓝海需求，需要从零构建。**

### 2.2 可参考的技术方案

Xplorer 的文件管理 UI 架构（面板布局、拖拽导入、右键菜单）可作为 UI/UX 参考，但其核心渲染机制需要重新设计以支持 HTML 渲染而非纯代码预览。

---

## 三、桌面框架选型

### 3.1 候选框架对比

| 维度 | **Electron** | **Tauri v2** |
|------|-------------|-------------|
| 渲染引擎 | 内置完整 Chromium | 调用系统 WebView（Win: Edge WebView2, Mac: WKWebView, Linux: WebKitGTK） |
| 打包体积 | ~300 MB | ~3-5 MB |
| 运行时内存 | ~100 MB | 差异大（20-350 MB，因 OS 不同） |
| Three.js/WebGL | ✅ Chromium 保证一致 | ⚠️ 不同 OS 的 WebView WebGL 实现有差异 |
| TypeScript 执行 | ✅ Node.js 原生支持 | ⚠️ 系统 WebView 对 TS 无原生支持，需编译 |
| 跨平台一致性 | ✅ 极高（同一 Chromium 版本） | ⚠️ 需三平台分别测试 |
| 生态成熟度 | ⭐⭐⭐⭐⭐ (121k stars) | ⭐⭐⭐⭐ (106k stars) |
| 学习成本 | 低（JS/TS 全栈） | 高（需学 Rust） |
| GitHub Stars | 121.2k | 106.3k |

### 3.2 推荐选型：Electron

**推荐理由（针对本项目需求）：**

1. **渲染一致性是关键** — 项目核心需求是"渲染 HTML/CSS/Three.js/TypeScript"，必须保证用户在 Win/Mac/Linux 上看到完全相同的渲染结果。Electron 内置 Chromium，保证三端渲染 100% 一致；Tauri 依赖系统 WebView，不同 OS 的 WebGL 实现、CSS 渲染、JS 引擎均有差异。

2. **TypeScript 原生支持** — Electron 的渲染进程本身就是 Chromium，对 TypeScript (通过编译)、ES Module、WebAssembly 等均有完整支持，无需额外配置。Tauri 的系统 WebView 对最新 Web 标准的支持程度取决于 OS 版本。

3. **Three.js/WebGL 兼容性** — Chromium 的 WebGL 实现经过 Google 大量测试，是事实上最稳定的 WebGL 运行环境。系统 WebView（特别是 Linux WebKitGTK）的 WebGL 支持可能有坑。

4. **开发效率** — Electron 的前后端统一用 JS/TS，无需学习 Rust，开发速度快。

**体积问题的接受理由：** 300MB 在 2026 年的桌面应用中已属可接受范围（VSCode、Discord、Slack 均为 Electron 应用），且本项目定位为"HTML 管理器"专业工具，用户对体积不敏感。

---

## 四、技术架构设计

### 4.1 总体架构

```
┌─────────────────────────────────────────────────────┐
│                  Electron Main Process               │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ 文件系统管理  │  │ 窗口生命周期  │  │ IPC 通信   │ │
│  │ (fs, path)  │  │ (BrowserWin) │  │ (ipcMain)  │ │
│  └─────────────┘  └──────────────┘  └────────────┘ │
├─────────────────────────────────────────────────────┤
│                Electron Renderer Process             │
│  ┌───────────────────┐  ┌─────────────────────────┐ │
│  │   文件管理面板      │  │    HTML 渲染面板         │ │
│  │   (React 18)      │  │   (webview / iframe)    │ │
│  │                   │  │                         │ │
│  │  - 文件树          │  │  - HTML 实时渲染         │ │
│  │  - 拖拽导入        │  │  - CSS3 完整支持         │ │
│  │  - 搜索/过滤       │  │  - Three.js WebGL       │ │
│  │  - 标签/分组       │  │  - TypeScript (编译后)   │ │
│  └───────────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### 4.2 推荐技术栈

| 层 | 技术 | 版本 | 理由 |
|----|------|------|------|
| 桌面框架 | Electron | 39.x+ | 成熟稳定，Chromium 内核 |
| 前端框架 | React | 18.x | 生态最丰富，组件化开发 |
| 构建工具 | Vite | 6.x+ | 极速 HMR，Electron 集成成熟 |
| 语言 | TypeScript | 5.x+ | 类型安全 |
| UI 组件库 | Ant Design / shadcn/ui | - | 现成 Tree、Splitter 等组件 |
| 文件树 | 自研 + react-arborist | - | 高性能虚拟树 |
| 代码编辑器 | Monaco Editor | - | 可选：HTML 源码编辑 |
| 打包 | electron-builder | 25.x+ | Win/Mac/Linux 全平台打包 |
| 状态管理 | Zustand | 5.x | 轻量，无 boilerplate |
| 测试 | Vitest + Playwright | - | 单元 + E2E |
| 代码规范 | ESLint + Prettier | - | 统一风格 |

### 4.3 渲染面板技术方案（核心）

这是本项目最关键的技术决策——如何在 Electron 中渲染用户导入的 HTML 文件。

#### 方案对比

| 方案 | 实现方式 | 优点 | 缺点 |
|------|---------|------|------|
| **A: `<webview>` 标签** | 使用 Electron 的 `<webview>` 加载本地 HTML | 独立进程，隔离性好，安全沙箱 | 通信复杂度高，Electron 计划废弃 webview |
| **B: `<iframe>` + 本地服务器** | 启动本地 HTTP 服务器，iframe 加载 HTML | 最接近真实浏览器环境，同源策略可控 | 需要管理本地服务器生命周期 |
| **C: BrowserView** | Electron 的 BrowserView API | 性能最优，独立渲染进程 | API 复杂，布局管理困难 |

#### 推荐方案：**B — 本地 HTTP 服务器 + iframe**

理由：
1. HTML 文件之间可以相对路径引用（`<script src="...">`, `<link href="...">`），本地 HTTP 服务器是最自然的方案
2. 同源策略可控，安全且灵活
3. 支持所有 Web 标准（Service Worker 除外）
4. 每个 HTML 的依赖文件（JS/CSS/资源）可正常加载
5. 实现简单，维护成本低

实现方式：
- 在 Main Process 启动一个轻量 HTTP 静态文件服务器（使用 Node.js 内置 `http` 模块或 `express`）
- 用户导入 HTML 文件夹后，服务器指向该文件夹根目录
- 渲染面板使用 `<iframe>` 加载 `http://localhost:<port>/<html-file-path>`
- 切换选中文件时，更新 iframe 的 src 即可

### 4.4 数据流设计

```
用户拖入 HTML 文件夹
        │
        ▼
┌──────────────────┐
│  Main Process    │  ← 读取文件系统，验证目录结构
│  文件扫描 & 存储  │
└──────┬───────────┘
       │ IPC: 文件列表 + 元数据
       ▼
┌──────────────────┐
│  Renderer        │  ← 渲染文件树 UI
│  文件管理面板      │
└──────┬───────────┘
       │ 用户点击文件
       ▼
┌──────────────────┐
│  Main Process    │  ← 确保本地服务器指向正确目录
│  本地 HTTP 服务器  │
└──────┬───────────┘
       │ HTTP
       ▼
┌──────────────────┐
│  iframe 加载 HTML │  ← 完整渲染 HTML/CSS/JS/Three.js
│  渲染面板         │
└──────────────────┘
```

---

## 五、功能规格

### 5.1 V1.0 MVP（最小可行产品）

| 功能 | 优先级 | 说明 |
|------|--------|------|
| 导入 HTML 文件夹 | P0 | 通过拖拽或"打开文件夹"按钮导入 |
| 文件树浏览 | P0 | 左侧面板显示导入目录的文件树结构 |
| HTML 渲染预览 | P0 | 右侧 panel 使用 iframe 渲染选中的 HTML |
| 基础面板布局 | P0 | 可调整大小的左右分栏布局 |
| 文件搜索/过滤 | P1 | 按文件名快速定位 |
| 最近打开记录 | P1 | 记住上次导入的文件夹 |
| 暗色/亮色主题 | P2 | 基础主题切换 |

### 5.2 V1.5 增强功能

| 功能 | 说明 |
|------|------|
| 多标签页 | 同时打开多个 HTML 文件，标签切换 |
| HTML 源码查看/编辑 | Monaco Editor 内嵌，双模式切换（源码/预览） |
| 收藏夹/书签 | 收藏常用 HTML 文件 |
| 刷新按钮 | 手动刷新渲染面板 |
| 开发者工具 | 内嵌 Chromium DevTools（针对渲染的 HTML） |
| 导入导出项目包 | 将导入的所有文件打包为 .zip 导出 |

### 5.3 V2.0 高级功能

| 功能 | 说明 |
|------|------|
| 实时编辑预览 | 编辑 HTML/CSS/JS 源码，渲染面板实时更新（HMR） |
| TypeScript 编译 | 内置 TS 编译器，自动编译 .ts 文件 |
| 响应式预览 | 模拟不同设备尺寸（Phone/Tablet/Desktop） |
| 外部 URL 导入 | 支持输入 URL 下载并导入远程 HTML 项目 |
| 插件系统 | 支持第三方插件扩展 |

---

## 六、目录结构设计

```
html-manager/
├── package.json
├── tsconfig.json
├── electron-builder.yml          # 打包配置
├── vite.config.ts
├── src/
│   ├── main/                     # Electron Main Process
│   │   ├── index.ts              # 主入口
│   │   ├── window.ts             # 窗口管理
│   │   ├── server.ts             # 本地 HTTP 服务器
│   │   ├── fileSystem.ts         # 文件扫描与管理
│   │   ├── ipcHandlers.ts        # IPC 通信处理
│   │   └── store.ts              # 持久化存储（electron-store）
│   ├── preload/                  # Preload Scripts
│   │   └── index.ts              # contextBridge API
│   └── renderer/                 # 渲染进程（前端）
│       ├── index.html
│       ├── main.tsx              # React 入口
│       ├── App.tsx               # 根组件
│       ├── components/
│       │   ├── Layout.tsx        # 面板布局（左右分栏）
│       │   ├── FileTree.tsx      # 文件树组件
│       │   ├── HtmlPreview.tsx   # HTML 渲染面板（iframe）
│       │   ├── Toolbar.tsx       # 工具栏
│       │   ├── ImportDialog.tsx  # 导入对话框
│       │   └── SearchBar.tsx     # 搜索栏
│       ├── stores/
│       │   └── appStore.ts       # Zustand 状态管理
│       ├── hooks/
│       │   ├── useFiles.ts       # 文件操作 hooks
│       │   └── usePreview.ts     # 预览控制 hooks
│       ├── styles/
│       │   └── global.css        # 全局样式
│       └── types/
│           └── electron.d.ts     # TypeScript 类型声明
├── resources/                    # 应用图标等资源
│   └── icon.png
└── tests/
    ├── unit/
    └── e2e/
```

---

## 七、开发路线图

| 阶段 | 时间 | 内容 |
|------|------|------|
| **Phase 1: 项目初始化** | Week 1 | 搭建 Electron + React + Vite 项目骨架，配置 TypeScript、ESLint、Prettier |
| **Phase 2: 核心布局** | Week 1-2 | 实现左右分栏布局（可拖拽调整宽度），文件树 UI 组件 |
| **Phase 3: 文件系统** | Week 2 | 实现文件夹导入（拖拽 + 对话框），文件扫描，文件树数据生成 |
| **Phase 4: 本地服务器** | Week 2-3 | 实现本地 HTTP 静态服务器，管理与切换文件目录 |
| **Phase 5: HTML 渲染** | Week 3 | 实现 iframe 渲染面板，服务器 URL 与文件选择联动 |
| **Phase 6: 功能完善** | Week 3-4 | 搜索/过滤、最近打开、主题切换、右键菜单 |
| **Phase 7: 测试 & 打包** | Week 4 | 三平台测试，electron-builder 配置，打包发布 |

---

## 八、三平台兼容性保障

| 措施 | 说明 |
|------|------|
| 统一 Chromium 版本 | Electron 内置固定版本 Chromium，保证三端渲染一致 |
| CI/CD 三平台构建 | GitHub Actions 配置 Windows/macOS/Linux 构建矩阵 |
| E2E 测试覆盖 | Playwright + Electron 在三平台上运行相同测试用例 |
| WebGL 验证 | 使用 Three.js 官方示例作为验收测试用例 |
| 路径处理 | 统一使用 `path.posix` 风格，避免 Windows 反斜杠问题 |

---

## 九、关键风险与应对

| 风险 | 影响 | 应对策略 |
|------|------|----------|
| 体积过大（~300MB） | 用户下载门槛高 | 使用 electron-builder 的差异更新，减小后续更新体积 |
| 本地服务器安全性 | 恶意 HTML 访问本地文件 | 服务器仅绑定 `127.0.0.1`，限制访问范围在导入目录内 |
| iframe 跨域问题 | HTML 中 fetch 失败 | 本地服务器添加 CORS 头，允许同源请求 |
| 大文件夹性能 | 千级文件卡顿 | 虚拟滚动（react-arborist）、文件树懒加载 |

---

## 十、备选方案（Plan B）

如果 Electron 的体积是不可接受的硬约束，可考虑 **Electrobun**：
- GitHub: https://github.com/blackboardsh/electrobun
- 打包体积仅 ~16MB，使用 Bun + Zig
- 支持 Three.js/Babylon.js 适配器
- 风险：项目较新（2025 年发布），生态不成熟，文档不完善

---

> **下一步：** 请 Review 以上计划，确认选型（Electron vs Tauri vs Electrobun）和功能范围（MVP vs 增强功能），我将开始实施 Phase 1。
