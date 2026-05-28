# Article Experiment Analysis — 通用实验流程分析方法与交互式定制指南

> 将文献中的实验设计信息结构化为三个可分析维度：**实验配置**、**流程步骤**、**变量关系**，并以交互式HTML进行可视化呈现。

---

## 1. 整体工作流概览

```
PDF文献集 → 文本提取 → 信息抽取 → 结构化归类 → 交互式HTML可视化 → 清理中间产物
```

| 阶段 | 工具/方法 | 产出物 |
|------|----------|--------|
| 文本提取 | PyMuPDF (fitz) 逐页提取纯文本 | `.txt` 中间文件 |
| 信息抽取 | LLM 阅读全文，识别实验相关段落 | 结构化笔记 |
| 归类整理 | 按三个维度人工+LLM协同归类 | 分类数据 |
| 可视化 | 纯HTML/CSS/JS，零依赖 | 单文件 `.html` |
| 清理 | 删除中间 `.txt` 文件 | 仅保留原始PDF + 最终HTML |

---

## 2. 三个核心分析维度

### 2.1 实验配置（Setup）

从论文中抽取三类配置信息，以卡片网格布局呈现：

| 配置类别 | 典型抽取内容 |
|---------|-------------|
| **实验场景** | 环境类型（真实/虚拟/仿真）、场景规模、关键特征、构建工具/平台 |
| **实验设备** | 硬件规格（分辨率/刷新率/传感器）、软件栈、实验室条件、数据采集方式 |
| **参与人员** | 样本量、人口学分布（年龄/性别）、招募方式、分组逻辑、纳入/排除标准 |

**抽取策略**：在论文中定位 Method / Materials / Participants 章节，按上述三类分桶。

### 2.2 实验流程（Procedure）

将实验过程拆解为**离散步骤序列**，以可交互的流程卡片从左至右展示：

- 每张卡片对应一个步骤
- 步骤间以箭头连接，表示时间顺序
- **关键创新**：对有关键数据产出的步骤施加色彩编码（如橙色边框 + "📊 关键数据产出" 徽标），与纯操作步骤形成视觉区分
- 卡片默认为折叠态，点击展开显示该步骤的详细说明

**拆分粒度原则**：
- 一个步骤 = 一个有明确边界的操作单元（参与者做一个连贯的事情）
- 典型拆分点：设备准备 → 练习/熟悉 → 正式实验各阶段 → 问卷/后测 → 结束
- 数据产出步骤的判断标准：步骤中是否产生了后续统计分析所用的原始数据

### 2.3 变量关系（Variables）

以表格形式呈现自变量(IV)、自变量处理方式、因变量(DV)的对应关系：

| 列 | 内容 |
|----|------|
| 自变量 (IV) | 实验中系统操纵的因素名称 |
| 自变量处理方式 | 被试间/被试内设计、水平数量、分组/条件的具体操作 |
| 因变量 (DV) | 测量的结果指标，按类别分组（行为指标/生理指标/主观报告等） |

**表格设计考量**：
- 当一个IV对应多个DV时，使用 `rowspan` 合并单元格以体现设计结构
- DV按逻辑类别分组（如"导航指标"/"眼动指标"/"问卷指标"）
- 对于算法类论文（无人类参与者），改为"消融因素"和"评估指标"

---

## 3. HTML可视化架构

### 3.1 设计原则

- **零外部依赖**：CSS变量管理主题色、纯JS实现交互，无需任何框架或CDN
- **单文件交付**：HTML/CSS/JS全部内嵌，可直接在浏览器打开
- **响应式布局**：grid/flexbox 自适应不同屏幕宽度
- **渐进交互**：核心内容直接可见，详细信息通过点击展开

### 3.2 CSS变量体系（一键换肤）

```css
:root {
  --bg: #f8f9fb;           /* 页面背景 */
  --card-bg: #ffffff;      /* 卡片背景 */
  --text: #2d3436;         /* 主文字色 */
  --muted: #636e72;        /* 次要文字色 */
  --accent: #0984e3;       /* 主题强调色（边框/标题/按钮） */
  --accent2: #6c5ce7;      /* 辅助强调色（渐变） */
  --data-step: #e17055;    /* 数据产出步骤标识色 */
  --data-step-bg: #fff5f3; /* 数据产出步骤背景色 */
  --border: #e0e5ec;       /* 边框色 */
  --radius: 14px;          /* 圆角 */
  --shadow: 0 2px 12px rgba(0,0,0,0.06);
}
```

修改 `:root` 中的变量值即可全局换肤。

### 3.3 三大组件结构

#### 组件A：配置信息网格

```html
<div class="setup-grid">           <!-- CSS Grid, auto-fit, min 320px -->
  <div class="setup-block">        <!-- 左侧色条 + 灰色背景 + 圆角 -->
    <h4>类别标题</h4>
    <ul>...</ul>
  </div>
  <!-- 重复2-3个 block -->
</div>
```

#### 组件B：流程卡片

```html
<div class="process-flow">                 <!-- Flexbox 横向滚动 -->
  <div class="process-step" onclick="..."> <!-- 可点击卡片 -->
    <div class="step-number">1</div>       <!-- 圆形编号 -->
    <div class="step-title">步骤名</div>
    <div class="data-badge">📊 关键数据产出</div>  <!-- 仅 data-step 显示 -->
    <div class="step-expand">详细描述</div> <!-- 点击后展开 -->
  </div>
  <div class="step-arrow">→</div>         <!-- 步骤间箭头 -->
  <!-- 重复... -->
</div>
```

**交互逻辑**：点击卡片切换 `.expanded` 类，CSS `max-height` 过渡动画展开/折叠。展开时自动滚动到可见区域。

#### 组件C：变量关系表

```html
<table class="var-table">
  <thead><tr><th>IV</th><th>处理方式</th><th>DV</th></tr></thead>
  <tbody>
    <tr><td rowspan="2">IV名称</td><td>水平1处理</td><td rowspan="2">DV列表</td></tr>
    <tr><td>水平2处理</td></tr>
  </tbody>
</table>
```

### 3.4 导航结构

每篇论文卡片顶部设锚点导航：

```html
<div class="paper-nav">
  <a href="#p1-setup">实验设置</a>
  <a href="#p1-proc">实验流程</a>
  <a href="#p1-var">变量关系</a>
</div>
```

---

## 4. 交互式定制选项

以下选项可通过修改HTML中的对应部分来实现不同程度的定制。

### 4.1 主题色切换（推荐实现方式）

在 `<style>` 前插入一个简易主题切换器：

```html
<!-- 添加到 <body> 开头 -->
<div style="position:fixed; top:16px; right:16px; z-index:999; display:flex; gap:8px;">
  <button onclick="document.documentElement.style.setProperty('--accent','#0984e3');document.documentElement.style.setProperty('--data-step','#e17055')" 
          style="padding:6px 12px; border-radius:20px; border:1px solid #ccc; background:#fff; cursor:pointer;">默认蓝</button>
  <button onclick="document.documentElement.style.setProperty('--accent','#00b894');document.documentElement.style.setProperty('--data-step','#d63031')" 
          style="padding:6px 12px; border-radius:20px; border:1px solid #ccc; background:#fff; cursor:pointer;">翠绿/红</button>
  <button onclick="document.documentElement.style.setProperty('--accent','#6c5ce7');document.documentElement.style.setProperty('--data-step','#fdcb6e')" 
          style="padding:6px 12px; border-radius:20px; border:1px solid #ccc; background:#fff; cursor:pointer;">紫/金</button>
</div>
```

### 4.2 筛选显示模式

添加全局切换按钮控制数据产出步骤的高亮：

```html
<!-- 添加到导航栏 -->
<label style="cursor:pointer; margin-left:16px; font-size:0.85rem;">
  <input type="checkbox" id="highlightData" checked onchange="
    document.querySelectorAll('.process-step').forEach(s => {
      if (!this.checked) s.classList.remove('data-step');
      else if (s.querySelector('.data-badge')) s.classList.add('data-step');
    })
  "> 高亮数据产出步骤
</label>
```

### 4.3 全部展开/折叠流程卡片

```html
<button onclick="document.querySelectorAll('.process-step').forEach(s=>s.classList.add('expanded'))">全部展开</button>
<button onclick="document.querySelectorAll('.process-step').forEach(s=>s.classList.remove('expanded'))">全部折叠</button>
```

### 4.4 论文筛选/搜索

如果论文数量较多，可添加简单的文本筛选：

```html
<input type="text" id="paperFilter" placeholder="搜索论文标题或作者..." 
  oninput="document.querySelectorAll('.paper-card').forEach(card => {
    const show = card.textContent.toLowerCase().includes(this.value.toLowerCase());
    card.style.display = show ? '' : 'none';
  })"
  style="width:100%; padding:10px 16px; border:1px solid var(--border); border-radius:24px; font-size:0.95rem; margin-bottom:24px;">
```

### 4.5 变量表行折叠（长表格压缩）

当某篇论文的因变量特别多时，可将详细的DV列默认折叠：

```html
<!-- 在变量表的 DV 单元格中 -->
<td>
  <details>
    <summary><strong>展开查看所有因变量 (共N项)</strong></summary>
    <strong>行为指标：</strong>...<br>
    <strong>眼动指标：</strong>...<br>
    <strong>主观报告：</strong>...
  </details>
</td>
```

### 4.6 导出为PDF（打印样式）

在CSS中添加 `@media print` 规则，自动展开所有折叠内容并移除交互装饰：

```css
@media print {
  .process-step { max-width: none; }
  .process-step .step-expand { max-height: none !important; padding: 12px 14px !important; }
  .step-arrow { display: none; }
  .process-flow { flex-wrap: wrap; overflow-x: visible; }
  .paper-nav { display: none; }
}
```

### 4.7 LLM驱动的领域与视角自动推荐

**核心理念**：不预设固定的实验类型模板，而是让LLM在读取PDF全文后，根据论文实际内容**动态推断**适用的分析领域和可选视角，并将推荐结果作为交互式选项嵌入HTML。

#### 4.7.1 推荐生成流程

```
PDF全文 → LLM阅读 → 两阶段推荐 → HTML中的领域/视角选择器
```

**阶段一：领域推荐（Domain Recommendation）**

LLM分析论文集后，基于以下信号自动归纳领域标签：

| 信号来源 | LLM提取内容 | 示例 |
|---------|-------------|------|
| 论文的研究范式 | 实验设计类型、对照逻辑 | 被试间对照实验、消融研究、纵向追踪 |
| 数据采集手段 | 设备、传感器、测量工具 | VR头显+眼动、运动捕捉、问卷量表 |
| 因变量的性质 | 测量的是什么类型的指标 | 行为绩效（时间/错误）、生理信号（眼动/ pupil）、主观报告（Likert） |
| 学科术语聚类 | 高频方法论关键词 | "between-subjects"、"space syntax"、"presence questionnaire" |
| 参与者特征 | 人群类型、样本设计 | 健康成年人、临床人群、跨年龄段、AI agent评估 |

LLM据此输出**领域推荐**，例如从本次5篇论文中自动推断出：

```json
{
  "recommendedDomains": [
    {
      "label": "VR人因实验",
      "description": "使用沉浸式/非沉浸式VR对比不同技术条件对用户行为的影响",
      "matchingPapers": [1, 4, 5],
      "configFocus": ["VR硬件对比", "沉浸度/临场感", "模拟器晕动症控制", "虚拟场景保真度"]
    },
    {
      "label": "空间认知行为实验",
      "description": "通过受控导航/寻路任务研究人类空间信息加工机制",
      "matchingPapers": [1, 2, 4],
      "configFocus": ["任务复杂度层级", "空间线索操纵", "个体差异变量", "策略分类编码方案"]
    },
    {
      "label": "心理测量与个体差异研究",
      "description": "使用标准化测试量表对被试进行预分类后比较组间行为差异",
      "matchingPapers": [2, 4],
      "configFocus": ["认知风格/能力前测工具", "聚类/分组方法", "组间均衡性检验"]
    },
    {
      "label": "智能导航系统算法评估",
      "description": "在标准基准数据集上对AI导航系统进行定量评估与消融分析",
      "matchingPapers": [3],
      "configFocus": ["数据集规模与划分", "基准方法选择", "消融维度设计", "评估指标集"]
    },
    {
      "label": "自适应界面可用性研究",
      "description": "根据环境/用户特征动态调整信息呈现方式并测量认知负荷",
      "matchingPapers": [5],
      "configFocus": ["信息呈现类型", "环境特征量化（space syntax等）", "眼动认知负荷指标"]
    }
  ]
}
```

**阶段二：视角推荐（Perspective Recommendation）**

在同一领域内，LLM根据论文中实际出现的分析维度，推荐**可切换的分析视角**：

```json
{
  "recommendedPerspectives": [
    {
      "id": "by_tech",
      "label": "按技术条件对比",
      "description": "比较不同实验条件/组别在同一任务上的表现差异",
      "applicableWhen": "论文包含明确的组间对照设计",
      "visualHint": "配置卡片按条件分组着色"
    },
    {
      "id": "by_dv_category",
      "label": "按因变量类别分层",
      "description": "按行为指标/生理指标/主观报告三个层级分别审视实验设计",
      "applicableWhen": "论文测量了跨层级的多类因变量",
      "visualHint": "变量表按DV类别分区着色"
    },
    {
      "id": "by_procedure_phase",
      "label": "按实验阶段拆分",
      "description": "将流程卡片按准备期/练习期/正式期/后期分组，关注各阶段的数据贡献",
      "applicableWhen": "实验有明显的多阶段结构",
      "visualHint": "流程卡片增加阶段分隔带"
    },
    {
      "id": "by_participant_group",
      "label": "按被试群体分段",
      "description": "当实验涉及多个差异化人群时，切换为每类人群独立展示其配置和结果",
      "applicableWhen": "论文包含年龄段/临床/专家-新手等多群体比较",
      "visualHint": "每组人群一个tab页"
    },
    {
      "id": "by_stimulus_condition",
      "label": "按刺激/操纵条件筛选",
      "description": "以自变量水平为筛选器，仅展示特定条件组合下的实验配置和预期结果",
      "applicableWhen": "论文有2个以上因子设计",
      "visualHint": "顶部增加条件筛选下拉菜单"
    }
  ]
}
```

#### 4.7.2 HTML中的交互式选择器实现

根据LLM返回的推荐结果，在HTML页面顶部动态渲染选择器：

```html
<!-- LLM推荐的选择器区域，由JS动态生成 -->
<div class="domain-perspective-selector" style="
  display:flex; gap:16px; align-items:center; flex-wrap:wrap;
  padding:16px 20px; background:#f0f3f8; border-radius:12px; margin-bottom:24px;
">
  <div>
    <label style="font-size:0.82rem; font-weight:700; color:var(--muted); display:block; margin-bottom:4px;">
      📂 分析领域
    </label>
    <select id="domainSelect" onchange="applyDomainFilter(this.value)" 
      style="padding:8px 28px 8px 12px; border-radius:8px; border:1px solid var(--border); font-size:0.9rem; background:#fff; cursor:pointer;">
      <option value="all">全部论文 (5篇)</option>
      <option value="vr_human_factors">VR人因实验 (3篇)</option>
      <option value="spatial_cognition">空间认知行为实验 (3篇)</option>
      <option value="psychometrics">心理测量与个体差异 (2篇)</option>
      <option value="ai_evaluation">智能导航系统算法评估 (1篇)</option>
      <option value="adaptive_ui">自适应界面可用性 (1篇)</option>
    </select>
  </div>

  <div>
    <label style="font-size:0.82rem; font-weight:700; color:var(--muted); display:block; margin-bottom:4px;">
      🔍 分析视角
    </label>
    <select id="perspectiveSelect" onchange="applyPerspective(this.value)"
      style="padding:8px 28px 8px 12px; border-radius:8px; border:1px solid var(--border); font-size:0.9rem; background:#fff; cursor:pointer;">
      <option value="default">默认：完整展示</option>
      <option value="by_tech">按技术条件对比</option>
      <option value="by_dv_category">按因变量类别分层</option>
      <option value="by_procedure_phase">按实验阶段拆分</option>
      <option value="by_participant_group">按被试群体分段</option>
      <option value="by_stimulus_condition">按操纵条件筛选</option>
    </select>
  </div>

  <div id="perspectiveHint" style="font-size:0.78rem; color:var(--muted); flex-basis:100%; margin-top:4px;">
    💡 当前领域由LLM根据论文内容自动推断，视角选项来自论文中实际存在的分析维度
  </div>
</div>
```

#### 4.7.3 视角切换的JS行为定义

```javascript
const perspectiveBehaviors = {
  default: {
    apply() { /* 恢复默认布局 */ },
    hint: "展示每篇论文的完整实验配置、全流程步骤和全部变量关系"
  },
  by_tech: {
    apply() {
      // 将同篇论文中不同技术条件（如HMD vs Desktop）的配置块并列着色
      document.querySelectorAll('.setup-block').forEach(block => {
        if (block.dataset.condition === 'hmd') block.style.borderLeftColor = '#0984e3';
        if (block.dataset.condition === 'desktop') block.style.borderLeftColor = '#00b894';
      });
    },
    hint: "以实验条件为轴重新组织配置卡片，相同任务不同条件的块并排对比"
  },
  by_dv_category: {
    apply() {
      // 在变量表中按DV类别插入分隔行，并用不同底色区分行为层/生理层/主观层
      document.querySelectorAll('.var-table td:last-child').forEach(td => {
        td.innerHTML = td.innerHTML
          .replace(/<strong>(.*?)：<\/strong>/g, 
            '<span class="dv-layer-tag">$1</span>');
      });
    },
    hint: "因变量按行为指标(蓝)、生理指标(绿)、主观报告(黄)三层着色标记"
  },
  by_procedure_phase: {
    apply() {
      // 在流程卡片间插入阶段分隔标签（准备期 | 正式期 | 后期）
      const phases = [
        { after: 2, label: '准备期' },
        { after: 4, label: '正式实验期' },
        { after: 99, label: '实验后期' }
      ];
      // 动态插入分隔条...
    },
    hint: "流程卡片按实验阶段分组，每组之间插入阶段标签分隔带"
  },
  by_participant_group: {
    apply() {
      // 为每组参与者创建tab切换，每个tab内独立展示该组相关的配置和变量
      // 隐藏不相关论文
    },
    hint: "按被试群体（如儿童/年轻人/老年人）分tab独立展示，隐藏不含该群体的论文"
  },
  by_stimulus_condition: {
    apply() {
      // 以自变量水平构建下拉筛选，仅展示匹配条件组合的变量关系
    },
    hint: "通过顶部下拉菜单选择特定自变量水平组合，页面仅展示匹配的实验配置"
  }
};

function applyPerspective(id) {
  const behavior = perspectiveBehaviors[id];
  if (behavior) {
    behavior.apply();
    document.getElementById('perspectiveHint').innerHTML = '💡 ' + behavior.hint;
  }
}
```

#### 4.7.4 LLM推荐Prompt的设计要点

推动LLM进行领域和视角推荐的Prompt结构：

```
你刚读完了N篇学术论文的全文。现在请完成以下两步：

【第一步：领域推荐】
分析这些论文在实验方法上的共性和差异。不要使用预设的学科分类，
而是根据论文中实际使用的研究范式、设备工具、数据采集方式和
分析逻辑，归纳出3-5个"方法学领域"标签。

对每个领域标签，请说明：
  - 标签名称（简洁，4-8个字）
  - 为什么这个标签能覆盖其中某几篇论文（列出匹配的论文编号）
  - 在这个领域下，实验配置应重点关注哪些维度

【第二步：视角推荐】
识别这些论文中实际存在的分析维度，推荐2-4个可切换的分析视角。
每个视角必须满足：至少2篇论文可以应用该视角进行重新审视。

对每个视角，请说明：
  - 视角名称
  - 适用条件（什么情况下应该选择这个视角）
  - 切换后页面的视觉变化建议

输出格式：JSON，方便直接注入HTML的JS代码中。
```

#### 4.7.5 本次5篇论文的实际推荐结果

以本次文献集为例，LLM在实际读取PDF后自动生成的推荐（即上述JSON的真实来源）：

| 推荐领域 | 覆盖论文 | 推断依据（来自PDF实际内容） |
|---------|---------|--------------------------|
| VR人因实验 | #1, #4, #5 | 三篇均以VR为实验媒介，系统比较了不同VR技术(HMD/Desktop/Mobile)或不同VR呈现方式对用户行为和体验的影响 |
| 空间认知行为实验 | #1, #2, #4 | 三篇的核心任务均为寻路/导航，操纵了空间线索可得性（地标/几何/地图/标志），测量了路径选择和策略偏好 |
| 心理测量与个体差异 | #2, #4 | 两篇均使用标准化前测工具对被试进行分类（SCST空间认知风格测试、19项视认知综合评估），然后比较组间差异 |
| 智能导航系统评估 | #3 | 基于VLN-CE标准基准，采用消融实验设计（移除注意力模块/改变阈值/改变地图尺寸），报告SR/SPL/NE等标准指标 |
| 自适应界面可用性 | #5 | 该论文根据环境特征(space syntax指标)动态调整信息类型(符号/照片/3D仿真)，通过眼动数据推断认知负荷 |

| 推荐视角 | 视角来源（PDF中的实际分析维度） | 至少2篇可用的证据 |
|---------|-------------------------------|-----------------|
| 按技术条件对比 | 论文#1直接对比HMD vs Desktop，#4隐含Quest2与前人PC VR研究的差异 | #1, #4, #5 |
| 按因变量类别分层 | 所有论文均收集了行为+生理(眼动/头部追踪)+主观报告三类DV | #1, #2, #4, #5 |
| 按被试群体分段 | #2明确分儿童/年轻人/老年人三组，#4分Landmark/Route/Survey三组 | #1, #2, #4 |
| 按操纵条件筛选 | #2的地标vs几何条件，#5的照片vs符号vs3D仿真类型 | #2, #3, #5 |
| 按实验阶段拆分 | 所有论文均有准备→练习→正式→后测的多阶段结构 | 全部5篇 |

---

## 5. 工作流关键决策点

### 5.1 PDF文本提取工具选择

| 工具 | 优势 | 劣势 | 适用场景 |
|------|-----|------|---------|
| **PyMuPDF (fitz)** | 速度快、保留段落结构 | 对扫描版PDF需要OCR | 原生数字PDF（大多数学术论文） |
| pdfplumber | 表格提取能力强 | 较慢 | 含表格数据的论文 |
| pdftotext (poppler) | 命令行友好 | Windows安装复杂 | Linux/Mac环境 |

### 5.2 信息抽取策略

- **机器可自动抽取**：通过正则/关键词匹配提取样本量、年龄范围、设备型号等结构化数据
- **LLM辅助抽取**：阅读全文后按三个维度输出结构化摘要，人工核验关键数据
- **人工最终确认**：核对数字（样本量、年龄均值、设备参数）与原文一致

### 5.3 流程步骤的拆分粒度

- 过粗（如"实验阶段"→"分析阶段"）：丢失可用信息，流程卡片无意义
- 过细（如"打开软件"→"点击按钮"→"输入ID"）：信息噪音大，卡片数量爆炸
- **推荐粒度**：以"参与者完成一个有意义操作单元"为基准，单篇论文5-8个步骤

### 5.4 数据产出步骤的判定

一个步骤被标记为"关键数据产出"当且仅当：
1. 该步骤生成了后续正式统计推断所用的原始数据
2. 数据不是纯管理性质（签到、编号等不算）

---

## 6. 扩展性：从5篇到N篇

当论文数量增长时，建议的改进方向：

1. **分离数据与展示**：将每篇论文的结构化信息存入JSON，HTML通过JS动态渲染
2. **模板化卡片**：为不同实验类型（行为实验/算法评估/调查研究）预设卡片模板
3. **批量对比视图**：增加跨论文对比模式——例如并排比较所有论文的参与者规模或变量设计
4. **添加标签系统**：为每篇论文打标签（如"VR"/"眼动"/"疏散"），支持多维筛选
5. **生成静态站点**：将JSON + HTML模板通过简单的构建脚本生成可部署的静态页面

### 示例JSON Schema

```json
{
  "papers": [
    {
      "id": 1,
      "title": "...",
      "authors": "...",
      "venue": "...",
      "year": 2022,
      "tags": ["VR", "wayfinding", "between-subjects"],
      "setup": {
        "scene": { "type": "virtual", "description": "..." },
        "equipment": [{ "name": "HTC Vive", "specs": "..." }],
        "participants": { "n": 70, "age_range": "17-64", "groups": ["HMD", "Desktop"] }
      },
      "procedure": [
        { "step": 1, "title": "导引", "description": "...", "has_data_output": false },
        { "step": 2, "title": "练习", "description": "...", "has_data_output": false }
      ],
      "variables": [
        { "iv": "VR技术", "manipulation": "被试间...", "dvs": ["行进时间", "头部旋转", "..."] }
      ]
    }
  ]
}
```

---

## 7. 总结

将实验论文的系统性分析方法归纳为三个普适维度（配置-流程-变量），配合交互式HTML可视化，可以在以下场景复用：

- **文献综述**：快速建立多篇论文实验方法的横向比较视图
- **实验复现准备**：在正式复现前结构化解构原始实验设计
- **实验设计参考**：为新实验的方案设计提供方法学模板库
- **教学材料**：将论文的实验方法部分转化为适合课堂展示的交互式材料

核心工作流（PDF→文本→抽取→归类→HTML→清理）跨学科通用，仅需调整三个分析维度的侧重点即可适配不同领域。
