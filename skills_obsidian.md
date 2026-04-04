---

## name: obsidian-markdown description: > Use this skill whenever the user wants to create, format, or improve any Obsidian Markdown note — including daily logs, research notes, benchmark reports, or knowledge-base pages. Triggers include: any mention of "Obsidian", "日志", "工作日誌", "笔记", "日記", or requests to "整理成 Markdown", "放进 Obsidian", "做成好看的格式". Also triggers when the user asks to add charts, diagrams, or visual summaries inside a Markdown file that will be opened in Obsidian. Always use this skill before writing any Dataviewjs code block — it contains critical rules that prevent known rendering failures.

# Obsidian Markdown 笔记与图表制作指南

sk本 Skill 涵盖两部分：

1. **结构规范** — 如何写出清晰、易读的 Obsidian 日志和研究笔记
2. **Dataviewjs 图表规范** — 如何正确渲染 SVG 图表，以及所有已知的踩坑记录

---


## 一、文件结构规范

### 1.1 Frontmatter

每个文件必须以 YAML frontmatter 开头，**每个字段独立一行**，tags 必须用列表格式：

```yaml
---
tags:
  - daily-log
  - engineering
created: 2026-03-18
version: V5.0
status: completed
---
```

> ⚠️ 常见错误：把多个字段写在同一行（例如 `created: 2026-03-18 version: V5.0`）。 这是对话框复制时换行被压缩导致的视觉假象，实际文件里必须保持每行独立。

### 1.2 工作日志结构模板

```markdown
# 📋 工作日誌｜{版本}｜{日期}

> [!success] 今日核心成果
> 一句话总结。

---

## 一、{模块名}
## 二、{模块名}
...
## N、技术栈 / 任务清单 / 遗留问题

---

*日报生成时间：{日期} ｜ 版本：{版本} ｜ 下次目标：{目标}*
```

### 1.3 Callout 类型速查

|类型|用途|
|---|---|
|`[!success]`|成果、通过、完成|
|`[!info]`|背景说明、定位|
|`[!warning]`|注意事项、风险|
|`[!bug]`|已知问题、错误记录|
|`[!abstract]`|核心概念、结论|
|`[!note]`|补充说明|
|`[!question]`|待确认的问题|

### 1.4 内部链接

引用其他笔记用 `[[文件名]]`，例如：`[[main_agent_dashboard]]`

---

## 二、Dataview 使用规范

### 2.1 静态内容不要用 `dataview`

`dataview` 代码块用于**查询 vault 内 notes 的 metadata**，不能显示硬编码的静态字符串。

````markdown
❌ 错误用法（无法渲染）：
```dataview
TABLE WITHOUT ID "Total Tokens" AS 指标, "2,257,471" AS 数值
````

✅ 正确做法：直接用 Markdown 表格

|指标|数值|
|---|---|
|Total Tokens|2,257,471|

````

### 2.2 何时用 `dataviewjs`

当需要**动态生成图表**（条形图、圆饼图、卡片、流程图等）时，使用 `dataviewjs` 代码块。

---

## 三、Dataviewjs SVG 图表规范（关键）

### 3.1 核心渲染规则

Obsidian 的 `dv.el()` 方法会对 HTML 字符串进行 **sanitize（净化）**，
导致 SVG 标签和属性被过滤掉，图表无法显示。

**唯一正确的做法：使用 DOM API 直接创建元素。**

```javascript
// ❌ 错误：用 innerHTML / dv.el() 传入 SVG 字符串
dv.el("div", `<svg>...</svg>`);  // SVG 会被过滤，什么都不显示

// ✅ 正确：用 DOM API 创建 SVG 元素
const NS = "http://www.w3.org/2000/svg";
const svg = document.createElementNS(NS, "svg");
svg.setAttribute("viewBox", "0 0 400 200");
svg.style.cssText = "width:100%;max-width:400px;display:block;";

// 创建元素
const rect = document.createElementNS(NS, "rect");
rect.setAttribute("x", 10);
// ... 设置其他属性

svg.appendChild(rect);
dv.container.appendChild(svg);  // ✅ 挂载到 container，不是 dv.el()
````

### 3.2 标准代码模板

每个图表块都应遵循以下结构：

````javascript
```dataviewjs
const NS = "http://www.w3.org/2000/svg";

// 1. 定义数据
const data = [ /* ... */ ];

// 2. 定义尺寸
const W = 500, H = 200;

// 3. 创建 SVG 根元素
const svg = document.createElementNS(NS, "svg");
svg.setAttribute("viewBox", `0 0 ${W} ${H}`);
svg.style.cssText = `width:100%;max-width:${W}px;display:block;font-family:sans-serif;`;

// 4. 辅助函数（文字节点）
function t(x, y, content, size, fill, anchor, weight) {
  const el = document.createElementNS(NS, "text");
  el.setAttribute("x", x);
  el.setAttribute("y", y);
  el.setAttribute("font-size", size);
  el.setAttribute("fill", fill);
  el.setAttribute("text-anchor", anchor || "start");
  if (weight) el.setAttribute("font-weight", weight);
  el.textContent = content;
  return el;
}

// 5. 绘制元素（rect / text / line / circle / path）
// ...
svg.appendChild(/* element */);

// 6. 挂载（必须用 dv.container，不能用 dv.el）
dv.container.appendChild(svg);
\```
````

### 3.3 常用元素速查

```javascript
// 矩形
const rect = document.createElementNS(NS, "rect");
rect.setAttribute("x", x);
rect.setAttribute("y", y);
rect.setAttribute("width", w);
rect.setAttribute("height", h);
rect.setAttribute("rx", 6);           // 圆角
rect.setAttribute("fill", "#4f86c6");
rect.setAttribute("opacity", "0.15");

// 直线
const line = document.createElementNS(NS, "line");
line.setAttribute("x1", x1); line.setAttribute("y1", y1);
line.setAttribute("x2", x2); line.setAttribute("y2", y2);
line.setAttribute("stroke", "#888");
line.setAttribute("stroke-width", "1.5");
line.setAttribute("stroke-dasharray", "4,3");  // 虚线

// 圆形
const circle = document.createElementNS(NS, "circle");
circle.setAttribute("cx", cx);
circle.setAttribute("cy", cy);
circle.setAttribute("r", r);
circle.setAttribute("fill", color);

// 路径（饼图扇形）
const path = document.createElementNS(NS, "path");
path.setAttribute("d", `M${cx},${cy} L${x1},${y1} A${r},${r} 0 ${largeArc},1 ${x2},${y2} Z`);
```

### 3.4 颜色规范

优先使用 CSS 变量以适配 Obsidian 亮色/暗色主题：

|用途|CSS 变量|
|---|---|
|正文文字|`var(--text-normal)`|
|次要文字|`var(--text-muted)`|
|强调色|`var(--text-accent)`|
|背景分隔|`var(--background-modifier-border)`|
|主背景|`var(--background-primary)`|

固定色板（适合数据分类）：

```javascript
const COLORS = {
  blue:   "#4f86c6",
  purple: "#9b87d4",
  green:  "#7ec8a4",
  orange: "#f0a500",
  red:    "#e05c5c",
};
```

---

## 四、已知踩坑与解决方案

### 坑 1：`dv.el()` 传入 SVG 字符串无法渲染

**现象**：图表区域空白，没有任何内容。  
**原因**：Obsidian sanitize 机制过滤掉了 HTML 字符串中的 SVG 标签。  
**解决**：改用 `document.createElementNS(NS, tagName)` + `dv.container.appendChild(svg)`。

---

### 坑 2：文字与卡片边框重叠

**现象**：描述文字超出卡片底部，或与下一行元素叠在一起。  
**原因**：卡片高度（`height`）写死，没有为多行文字留足空间。  
**解决**：动态计算高度：

```javascript
const CARD_H = HDR_H + ITEM_START + items.length * ITEM_LINE + PADDING;
```

底部注释（如图例说明）要放在 `CARD_H + offset` 处，不能直接用 `H`：

```javascript
// ❌ 错误：注释刚好贴在卡片底部甚至被遮住
svg.appendChild(t(W/2, H, "说明文字", ...));

// ✅ 正确：在卡片底部加上安全偏移
const H = CARD_H + 20;
svg.appendChild(t(W/2, CARD_H + 15, "说明文字", ...));
```

---

### 坑 3：卡片 Header 圆角漏出背景色

**现象**：Header 矩形与卡片背景之间出现白色缝隙，底部圆角区域颜色不连续。  
**原因**：`rx` 圆角导致矩形四角都被裁切，header 底部出现空隙。  
**解决**：在 header 下方叠加一个无圆角的矩形填补缝隙：

```javascript
const hdr = document.createElementNS(NS, "rect");
hdr.setAttribute("rx", 8);
hdr.setAttribute("height", HDR_H);
// ...
svg.appendChild(hdr);

// 补丁：填补 header 底部圆角造成的缝隙
const fix = document.createElementNS(NS, "rect");
fix.setAttribute("x", x);
fix.setAttribute("y", HDR_H - 10);   // 略微上移，覆盖圆角区域
fix.setAttribute("width", colW);
fix.setAttribute("height", 10);
fix.setAttribute("fill", color);
fix.setAttribute("opacity", "0.8");  // 与 header 相同
svg.appendChild(fix);
```

---

### 坑 4：`dataview` 代码块显示静态内容

**现象**：`dataview` 块渲染出「No results」或完全空白。  
**原因**：`dataview` 只能查询 vault metadata，不能显示硬编码字符串。  
**解决**：改用普通 Markdown 表格，或改用 `dataviewjs` 动态生成。

---

### 坑 5：SVG viewBox 宽度不够，文字被裁切

**现象**：右侧的文字标签（如数值、说明）显示不全。  
**原因**：`viewBox` 宽度写死，没有为标签预留足够空间。  
**解决**：宽度动态计算，并在最右侧内容之后加 padding：

```javascript
const totalW = labelW + chartW + 100;  // 100 为右侧 padding
svg.setAttribute("viewBox", `0 0 ${totalW} ${H}`);
```

---

### 坑 6：多卡片布局总宽度计算错误

**现象**：最后一张卡片超出 viewBox，被截断。  
**原因**：总宽度 = `cols * colW + (cols-1) * gap`，漏算了间距。  
**解决**：

```javascript
const W = dims.length * colW + (dims.length - 1) * gap;
// 不要写成 dims.length * (colW + gap)，最后一列后面没有 gap
```

---

## 五、图表类型选型指南

|需求|推荐类型|说明|
|---|---|---|
|分类占比|圆饼图|适合 2–6 个分类，用扇形 + 图例|
|数量对比|横向条形图|左侧标签、右侧数值，适合 4–8 项|
|状态清单|行列卡片|带色条/badge，适合带描述的列表|
|流程/架构|横向流程图|方框 + 箭头，3–5 个节点|
|多维分类|纵向卡片组|每列一个分类，适合 3–5 列|
|进度/比率|进度条|单条填充，适合单一比率展示|
|数据表格|SVG 表格|带 header 背景色和交替行色|

---

## 六、完整图表示例：横向条形图

````javascript
```dataviewjs
const NS = "http://www.w3.org/2000/svg";
const data = [
  { label: "System Prompt", value: 94, color: "#4f86c6" },
  { label: "User Input",    value: 6,  color: "#f0a500" },
  { label: "Tools",         value: 0,  color: "#e05c5c" },
];

const barH = 28, gap = 10, labelW = 160, chartW = 200, padding = 80;
const W = labelW + chartW + padding;
const H = data.length * (barH + gap) + 30;
const maxVal = Math.max(...data.map(d => d.value));

const svg = document.createElementNS(NS, "svg");
svg.setAttribute("viewBox", `0 0 ${W} ${H}`);
svg.style.cssText = `width:100%;max-width:${W}px;display:block;font-family:sans-serif;`;

function t(x, y, content, size, fill, anchor, weight) {
  const el = document.createElementNS(NS, "text");
  el.setAttribute("x", x); el.setAttribute("y", y);
  el.setAttribute("font-size", size); el.setAttribute("fill", fill);
  el.setAttribute("text-anchor", anchor || "start");
  if (weight) el.setAttribute("font-weight", weight);
  el.textContent = content; return el;
}

data.forEach((d, i) => {
  const y = i * (barH + gap);
  const bw = maxVal > 0 ? Math.round((d.value / maxVal) * chartW) : 0;

  // Label
  svg.appendChild(t(labelW - 8, y + barH/2 + 4, d.label, "12", "var(--text-normal)", "end"));

  // Bar
  if (bw > 0) {
    const bar = document.createElementNS(NS, "rect");
    bar.setAttribute("x", labelW); bar.setAttribute("y", y);
    bar.setAttribute("width", bw); bar.setAttribute("height", barH);
    bar.setAttribute("rx", 4); bar.setAttribute("fill", d.color);
    bar.setAttribute("opacity", "0.85");
    svg.appendChild(bar);
  }

  // Value label
  svg.appendChild(t(labelW + bw + 6, y + barH/2 + 4, `${d.value}%`, "11", "var(--text-muted)"));
});

dv.container.appendChild(svg);
\```
````

---

## 七、配色设计原则：单一主色调系统

### 7.1 核心原则

**一份报告只用一套主色调。** 颜色不是装饰，是信息——滥用颜色等于没有颜色。

|错误做法|正确做法|
|---|---|
|每个图表各自用 4–5 种颜色|全文统一一个 ACCENT 主色|
|用颜色区分「不同类别」|用颜色标记「需要注意的状态」|
|header 用彩色背景|用细线条或轻透明度做结构|

### 7.2 推荐色彩变量声明

在每个 `dataviewjs` 块顶部统一声明，全文保持一致：

```javascript
const ACCENT = "#3b6ea5";                              // 主色：深蓝，用于关键数字、强调线、accent dot
const MUTED  = "var(--text-muted)";                    // 次要文字、说明、单位
const NORMAL = "var(--text-normal)";                   // 正文文字、标签
const BORDER = "var(--background-modifier-border)";    // 边框、分隔线、背景条
```

### 7.3 颜色使用规则

**只在以下三种情况使用 ACCENT 颜色：**

1. **关键数字** — 通过率、核心指标、需要读者第一眼看到的数值
2. **高亮状态** — 对比组中需要突出的那一项（如「动态测试」vs「静态测试」）
3. **结构锚点** — 卡片顶部 accent bar、列表左侧 dot、流程图中的核心节点

**其他所有元素用中性灰：**

- 背景条：`BORDER`（opacity 0.4–0.5）
- 普通边框：`BORDER`（stroke-width 1）
- 次要文字：`MUTED`
- 普通标签：`NORMAL`

### 7.4 卡片设计模式（轻量版）

避免彩色 header 背景，改用「顶部细线 + 无色卡片」：

```javascript
// ❌ 旧模式：彩色 header 背景，视觉噪音大
const hdr = document.createElementNS(NS, "rect");
hdr.setAttribute("fill", "#4f86c6");  // 每张卡片颜色不同 → 很吵

// ✅ 新模式：只用顶部 3–4px 细线作为视觉锚点
const bd = document.createElementNS(NS, "rect");
bd.setAttribute("fill", "none");
bd.setAttribute("stroke", BORDER);       // 边框用中性灰
svg.appendChild(bd);

const accentBar = document.createElementNS(NS, "rect");
accentBar.setAttribute("height", "3");   // 仅 3px 细线
accentBar.setAttribute("fill", ACCENT);  // 全文统一同一个 ACCENT
svg.appendChild(accentBar);
```

### 7.5 高亮对比模式

当需要在一组数据中突出某一项时，用「其余项降灰 + 目标项显色」：

```javascript
bars.forEach((b) => {
  const fillColor = b.highlight ? ACCENT : BORDER;   // 高亮项用 ACCENT，其余用灰
  const opacity   = b.highlight ? "1" : "0.5";       // 高亮项不透明，其余半透明
  const weight    = b.highlight ? "bold" : null;     // 高亮项文字加粗
  // ...
});
```

### 7.6 何时可以用多色

**只有一种情况允许多色**：表示「互斥状态」，且状态本身有固定语义：

|状态|颜色|说明|
|---|---|---|
|通过 / 成功|`#7ec8a4`（绿）|固定语义，读者有预期|
|失败 / 错误|`#e05c5c`（红）|固定语义，读者有预期|
|待定 / 进行中|`#f0a500`（橙）|固定语义，读者有预期|

除此之外，分类颜色、图表装饰色一律用单一 ACCENT + 透明度变化。

---

## 八、完整模板：单色系工作日志图表

以下是一个完整的单色系条形图示例，可直接复用：

````javascript
```dataviewjs
const NS = "http://www.w3.org/2000/svg";
const ACCENT = "#3b6ea5";
const MUTED  = "var(--text-muted)";
const NORMAL = "var(--text-normal)";
const BORDER = "var(--background-modifier-border)";

const bars = [
  { label: "项目 A", sub: "说明文字", value: 89,   highlight: false },
  { label: "项目 B", sub: "说明文字", value: 93,   highlight: false },
  { label: "项目 C", sub: "重点项目", value: 97.7, highlight: true  },
];

const barH = 36, gap = 10, lw = 140, chartW = 260, padR = 60;
const W = lw + chartW + padR, H = bars.length * (barH + gap);

const svg = document.createElementNS(NS, "svg");
svg.setAttribute("viewBox", `0 0 ${W} ${H}`);
svg.style.cssText = `width:100%;max-width:${W}px;display:block;font-family:sans-serif;`;

function t(x, y, content, size, fill, anchor, weight) {
  const el = document.createElementNS(NS, "text");
  el.setAttribute("x", x); el.setAttribute("y", y);
  el.setAttribute("font-size", size); el.setAttribute("fill", fill);
  el.setAttribute("text-anchor", anchor || "start");
  if (weight) el.setAttribute("font-weight", weight);
  el.textContent = content; return el;
}

bars.forEach((b, i) => {
  const y = i * (barH + gap);
  const bw = Math.round((b.value / 100) * chartW);

  // 左侧标签（双行）
  svg.appendChild(t(lw - 10, y + barH / 2 - 4, b.label, "13", NORMAL, "end", b.highlight ? "bold" : null));
  svg.appendChild(t(lw - 10, y + barH / 2 + 11, b.sub, "10", MUTED, "end"));

  // 背景条
  const bg = document.createElementNS(NS, "rect");
  bg.setAttribute("x", lw); bg.setAttribute("y", y);
  bg.setAttribute("width", chartW); bg.setAttribute("height", barH);
  bg.setAttribute("rx", 4); bg.setAttribute("fill", BORDER);
  bg.setAttribute("opacity", "0.4");
  svg.appendChild(bg);

  // 填充条：高亮项用 ACCENT，其余用灰
  const fill = document.createElementNS(NS, "rect");
  fill.setAttribute("x", lw); fill.setAttribute("y", y);
  fill.setAttribute("width", bw); fill.setAttribute("height", barH);
  fill.setAttribute("rx", 4);
  fill.setAttribute("fill", b.highlight ? ACCENT : BORDER);
  fill.setAttribute("opacity", b.highlight ? "1" : "0.7");
  svg.appendChild(fill);

  // 数值标签
  svg.appendChild(t(
    lw + bw + 8, y + barH / 2 + 5,
    `${b.value}%`,
    "13", b.highlight ? ACCENT : NORMAL,
    "start", b.highlight ? "bold" : null
  ));
});

dv.container.appendChild(svg);
\```
````

---

_Skill 版本：1.1 ｜ 基于 2026-03 实践经验整理_
