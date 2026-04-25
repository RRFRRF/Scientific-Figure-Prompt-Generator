---
name: scientific-figure-prompt
description: Use when generating AI drawing prompts for scientific figures, academic papers, or technical presentations. Triggers on requests for 配图、流程图、架构图、示意图、figure prompt、scientific figure、publication-ready illustration.
user-invocable: true
---

# Scientific Figure Prompt

## Overview

从技术描述直接生成结构化、出版级质量的 AI 绘图提示词，适配 Midjourney / DALL-E / Gemini。

核心机制：

- **交互式风格选择**：7 个预设方案 + 自定义
- **全局风格文档** (`global_style.md`)：一次生成，全篇论文多图复用，保证视觉一致性
- **三阶段内部推理**：Understand → Think → Design — 用户只看到最终 prompt

## When to Use

**Use when:**
- 用户需要为科学论文、技术报告、学术汇报生成配图 prompt
- 用户提到：配图、流程图、架构图、示意图、figure prompt、scientific figure
- 用户需要多张配图共享统一视觉风格

**When NOT to use:**
- 纯艺术创作、非技术类插图、照片处理

## Core Pattern

### Step 1: 收集用户输入

用 `clarify` 依次获取以下信息（可合并为一次询问）：

| 字段 | 必填 | 说明 |
|------|------|------|
| **visualization_goal** | Y | 这张图要讲什么故事？一句话核心信息 |
| **technical_content** | Y | 要可视化的技术内容：系统/算法/概念，关键组件、关系、数据流 |
| **scene_description** | Y | 大致场景描述：几个模块、大致布局方向、重点突出什么 |
| **language** | Y | 标注语言：中文 / 英文 / 中英双语 |
| **global_style** | N | 全局绘图风格 md（已有则粘贴；无则留空，系统会生成） |
| **special_requirements** | N | 特殊约束：必须突出 X、避免 Y、类似某论文风格等 |

如果用户消息中已包含部分信息，只需补充缺失项。

### Step 2: 全局绘图风格处理

这是多图风格一致性的核心机制。一篇论文/汇报通常需要多张配图，必须共享同一套视觉语言。

#### 判断逻辑

```
IF 用户提供了 global_style (完整的 md 格式全局风格文档):
    → 直接使用，跳过风格选择，进入 Step 4

ELIF 用户提供了模糊风格要求 (如 "蓝色系"、"偏简约"):
    → 基于模糊要求 + 默认预设 P1 生成 global_style.md
    → 输出 global_style.md 给用户保存
    → 进入 Step 4

ELSE (用户未提供任何风格信息):
    → 进入 Step 3 风格选择
    → 选择后生成 global_style.md
    → 输出 global_style.md 给用户保存
    → 进入 Step 4
```

#### global_style.md 格式

```markdown
# Global Figure Style — [项目名/论文名]

## Color Palette
- Primary (#XXXXXX): [用途说明，如 "核心模块/主流程"]
- Secondary (#XXXXXX): [用途说明，如 "辅助组件/输出"]
- Accent (#XXXXXX): [用途说明，如 "创新点/重点高亮"]
- Neutral (#XXXXXX): [用途说明，如 "连接线/标注文字"]
- Background (#XXXXXX): [背景色]

## Typography
- Language: [Chinese / English / Bilingual]
- Font: sans-serif (e.g., Inter, Noto Sans)
- Label hierarchy: Module name (14px bold) > Sub-label (12px regular) > Annotation (10px light)

## Shapes & Strokes
- Module: rounded rectangle, 8px radius, 1.5px stroke
- Group boundary: dashed rectangle, 1px stroke, 6px dash
- Arrow: solid, 1.5px, with arrowhead
- Feedback arrow: curved, dashed, 1px
- Icon: minimal line-style, 24px

## Visual Rules
- Shadows: none
- Gradients: none
- 3D effects: none
- Decorative elements: none
- Background: solid color only

## Technical Specs
- Resolution: 4K (3840x2160), 300 DPI
- Default aspect ratio: 16:9 landscape
- Quality: IEEE/ACM publication standard

## Negative Prompt
no watermarks, no blurry, no cluttered, no 3D, no gradients, no shadows, no decorative elements, no stock photo style, no text errors, no overlapping labels, no dark background
```

#### 使用方式

- **第一次调用**：无 global_style → 系统生成并输出 → 用户保存为 `figure_style.md`
- **后续调用**：用户粘贴 `figure_style.md` 内容到 global_style 字段 → 系统直接使用，不再输出风格文档
- **跨图一致性**：同一项目所有图共享同一份 global_style，确保色板、字体、形状、规格完全一致

### Step 3: 风格选择（交互式）

仅在 Step 2 中用户未提供 global_style 时执行。

用 `clarify` 提供预设方案，同时允许用户自定义。选择后自动生成 `global_style.md` 并输出给用户保存。

## Quick Reference

### 7 种预设方案

| 编号 | 预设名 | 适用场景 | 关键特征 |
|------|--------|----------|----------|
| **P1** | AI/SE 学术白底简约 | 论文/汇报配图（默认） | 白底、浅色系、无渐变、圆角矩形、1.5px 描边、无 3D、无阴影、sans-serif |
| **P2** | Nature/Science 深色 | 顶刊 Figure | 浅灰底(#F5F5F5)、蓝绿主色系、细线连接、subfigure 编号 |
| **P3** | 系统架构图 | 微服务/系统设计 | 分层布局、模块色块区分、虚线边界、数据流箭头 |
| **P4** | 数据流水线 | ETL/Pipeline/Workflow | 左→右流向、阶段色带、图标+标签、并行分支 |
| **P5** | 循环/反馈图 | 训练循环/迭代优化 | 环形布局、反馈箭头、收敛指示、中心核心 |
| **P6** | 网络拓扑 | 分布式系统/通信图 | 节点-边模型、辐射/网状、协议标注、层次分组 |
| **P7** | 自定义 | 用户完全自定义 | 用户描述风格偏好 |

**默认选 P1**（AI/SE 学术白底简约）。

### 预设参数映射

#### P1: AI/SE 学术白底简约（默认）

```yaml
background: "#FFFFFF"
color_palette:
  primary: "#4A90D9"
  secondary: "#7EC8A0"
  accent: "#E8855B"
  neutral: "#8C8C8C"
  background: "#FFFFFF"
style:
  medium: clean vector illustration, flat design
  shapes: rounded rectangles (8px radius), 1.5px stroke
  icons: minimal line-style, 24px
  shadows: none
  gradients: none
  effects_3d: none
  typography: sans-serif, clear hierarchy
tech_specs:
  resolution: 4K (3840x2160), 300 DPI
  aspect_ratio: 16:9 landscape
  quality: IEEE/ACM publication standard
negative_prompt: no watermarks, no blurry, no cluttered, no 3D, no gradients, no shadows, no decorative elements, no stock photo style, no text errors, no overlapping labels, no dark background
```

#### P2: Nature/Science 深色

```yaml
background: "#F5F5F5"
color_palette:
  primary: "#2B6CB0"
  secondary: "#38A169"
  accent: "#D69E2E"
  neutral: "#718096"
  background: "#F5F5F5"
style:
  medium: clean vector, subtle line weight variation
  shapes: rounded rectangles (4px radius), 1px stroke
  subfigure_numbering: true
  shadows: none
  gradients: none
tech_specs:
  resolution: 4K, 300 DPI
  aspect_ratio: "3:2"
  quality: Nature/Science publication standard
negative_prompt: no watermarks, no blurry, no 3D, no gradients, no decorative, no text errors
```

#### P3: 系统架构图

```yaml
background: "#FFFFFF"
color_palette:
  primary: "#3182CE"
  secondary: "#805AD5"
  accent: "#DD6B20"
  neutral: "#A0AEC0"
  background: "#FFFFFF"
style:
  medium: clean vector, layered layout
  shapes: rounded rectangles, dashed boundaries for groups
  connections: solid arrows for data flow, dashed for dependencies
  shadows: none
  gradients: none
tech_specs:
  resolution: 4K, 300 DPI
  aspect_ratio: 16:9
  quality: IEEE publication standard
negative_prompt: no watermarks, no blurry, no 3D, no gradients, no decorative, no text errors
```

#### P4: 数据流水线

```yaml
background: "#FFFFFF"
color_palette:
  primary: "#3B82F6"
  secondary: "#10B981"
  accent: "#F59E0B"
  neutral: "#6B7280"
  background: "#FFFFFF"
style:
  medium: clean vector, left-to-right flow
  shapes: rounded rectangles with stage color bands
  connections: solid arrows, parallel branches
  shadows: none
  gradients: none
tech_specs:
  resolution: 4K, 300 DPI
  aspect_ratio: 16:9 landscape
  quality: IEEE publication standard
negative_prompt: no watermarks, no blurry, no 3D, no gradients, no decorative, no text errors
```

#### P5: 循环/反馈图

```yaml
background: "#FFFFFF"
color_palette:
  primary: "#6366F1"
  secondary: "#14B8A6"
  accent: "#EF4444"
  neutral: "#94A3B8"
  background: "#FFFFFF"
style:
  medium: clean vector, circular/radial layout
  shapes: rounded rectangles, curved feedback arrows
  shadows: none
  gradients: none
tech_specs:
  resolution: 4K, 300 DPI
  aspect_ratio: 1:1 square
  quality: IEEE publication standard
negative_prompt: no watermarks, no blurry, no 3D, no gradients, no decorative, no text errors
```

#### P6: 网络拓扑

```yaml
background: "#FFFFFF"
color_palette:
  primary: "#2563EB"
  secondary: "#7C3AED"
  accent: "#DC2626"
  neutral: "#9CA3AF"
  background: "#FFFFFF"
style:
  medium: clean vector, node-edge graph
  shapes: circles for nodes, lines for edges, grouped by hierarchy
  shadows: none
  gradients: none
tech_specs:
  resolution: 4K, 300 DPI
  aspect_ratio: 16:9
  quality: IEEE publication standard
negative_prompt: no watermarks, no blurry, no 3D, no gradients, no decorative, no text errors
```

#### P7: 自定义

由用户描述风格偏好，AI 衡量后生成参数集。需确保输出格式与上述预设一致。

## Implementation

### Step 4: 三阶段推理生成 Prompt

内部执行，不输出中间过程。

#### Phase 1: UNDERSTAND

- 提取 core story（输入→处理→输出？对比？层次？循环？）
- 列出 key entities 和 relationships
- 识别 innovation/highlight 点
- 确定受众（reviewers / general / technical）和载体（paper / slides / poster）

#### Phase 2: THINK

- 从 7 种图类型中选择最匹配的，给出理由：
  System Architecture / Flowchart-Pipeline / Layered / Cycle-Loop / Comparison / Radial-HubSpoke / Swimlane
- 设计视觉层级：FIRST（最大/最醒目）→ SECOND（支撑）→ de-emphasize（必要但不焦点）
- 布局方向：L→R（流程）、T→B（层次）、radial（循环）
- 创新点位置：黄金分割位 / 中心 / 终端

#### Phase 3: DESIGN

基于 Phase 1-2 结论 + 用户选定预设参数，生成结构化 prompt。

### Step 5: 输出格式

**情况 A：用户提供了 global_style（后续调用）** — 只输出绘图 prompt，不输出 global_style.md。

**情况 B：用户未提供 global_style（首次调用）** — 先输出 `global_style.md`（供用户保存），再输出绘图 prompt。

绘图 prompt 格式：

```
=== SCENE DESCRIPTION ===
[2-3 句话描述：图类型 + 主体 + 核心视觉特征 + 风格]
[核心故事/流向]
[整体构图]

=== VISUAL ELEMENTS ===
[Group 1 Name]:
• [Element]: [shape], [color], [position], [label]

[Group 2 Name]:
• [Element]: [shape], [color], [position], [label]

[Connections]:
• [Source] → [Target]: [arrow style], [label]

=== COMPOSITION & LAYOUT ===
• Direction: [left-to-right / top-to-bottom / radial]
• Focal point: [视觉焦点]
• Grouping: [元素分组方式]
• White space: [留白位置]

=== COLOR PALETTE ===
• Primary (#XXXXXX): [用途]
• Secondary (#XXXXXX): [用途]
• Accent (#XXXXXX): [用途]
• Neutral (#XXXXXX): [用途]
• Background (#XXXXXX)

=== STYLE SPECIFICATIONS ===
• Medium: [from preset]
• Aesthetic: professional, technical, publication-ready
• Shapes: [from preset]
• Icons: [from preset]
• Typography: [language] labels, sans-serif
• Shadows: none
• Gradients: none

=== TECHNICAL SPECS ===
• Resolution: [from preset]
• Aspect ratio: [from preset]
• Quality: [from preset]

=== NEGATIVE PROMPT ===
[from preset]
```

### Step 6: 自检

输出前内部验证：
- 核心故事是否一眼可读？
- 创新点是否视觉突出？
- 用户所有需求是否覆盖？
- 颜色是否为浅色系、无渐变（P1 默认约束）？
- 标注语言是否匹配用户选择？
- Prompt 是否足够具体可复现？

### 示例 1：首次调用（无 global_style）

**输入：**
```
visualization_goal: 展示 RAG 系统如何通过检索增强减少 LLM 幻觉
technical_content: Query→Embedding→Vector DB检索→Context拼接→LLM生成→验证。关键创新：双路检索（稀疏+稠密）+ 自适应重排序
scene_description: 左到右流水线，中间突出双路检索模块，5-6个组件
language: 英文
global_style: (未提供)
preset: P1
```

**输出（先输出 global_style.md，再输出 prompt）：**

```markdown
# Global Figure Style — RAG System Paper

## Color Palette
- Primary (#4A90D9): core modules / main pipeline stages
- Secondary (#7EC8A0): output / generation components
- Accent (#E8855B): innovation highlights / key differentiator
- Neutral (#8C8C8C): connection lines / annotations
- Background (#FFFFFF): white

## Typography
- Language: English
- Font: sans-serif (Inter)
- Label hierarchy: Module name (14px bold) > Sub-label (12px regular) > Annotation (10px light)

## Shapes & Strokes
- Module: rounded rectangle, 8px radius, 1.5px stroke
- Group boundary: dashed rectangle, 1px stroke, 6px dash
- Arrow: solid, 1.5px, with arrowhead
- Feedback arrow: curved, dashed, 1px
- Icon: minimal line-style, 24px

## Visual Rules
- Shadows: none
- Gradients: none
- 3D effects: none
- Decorative elements: none
- Background: solid color only

## Technical Specs
- Resolution: 4K (3840x2160), 300 DPI
- Default aspect ratio: 16:9 landscape
- Quality: IEEE/ACM publication standard

## Negative Prompt
no watermarks, no blurry, no cluttered, no 3D, no gradients, no shadows, no decorative elements, no stock photo style, no text errors, no overlapping labels, no dark background
```

---

```
=== SCENE DESCRIPTION ===
A left-to-right pipeline diagram showing a Retrieval-Augmented Generation (RAG) system, with the dual-path retrieval module visually emphasized at the center.
The diagram illustrates how user queries flow through embedding, dual-path retrieval (sparse + dense), adaptive reranking, context assembly, and LLM generation with verification.
Clean flat design on white background, with the innovation module highlighted in warm orange.

=== VISUAL ELEMENTS ===
Input Stage:
• User Query: rounded rectangle, #4A90D9, leftmost, "Query"
• Query Embedding: rounded rectangle, #4A90D9, second, "Embedding"

Retrieval Stage (Highlighted):
• Sparse Retriever: rounded rectangle, #E8855B, center-upper, "Sparse (BM25)"
• Dense Retriever: rounded rectangle, #E8855B, center-lower, "Dense (Vector)"
• Adaptive Reranker: rounded rectangle, #E8855B, center-right, "Adaptive Reranking"

Generation Stage:
• Context Assembly: rounded rectangle, #7EC8A0, right-center, "Context Assembly"
• LLM Generator: rounded rectangle, #7EC8A0, right, "LLM Generation"
• Verifier: rounded rectangle, #7EC8A0, rightmost, "Verification"

Connections:
• Query → Embedding: solid arrow, ""
• Embedding → Sparse Retriever: solid arrow, "sparse path"
• Embedding → Dense Retriever: solid arrow, "dense path"
• Sparse Retriever → Reranker: solid arrow, ""
• Dense Retriever → Reranker: solid arrow, ""
• Reranker → Context Assembly: solid arrow, "ranked docs"
• Context Assembly → LLM Generator: solid arrow, "augmented prompt"
• LLM Generator → Verifier: solid arrow, "output"

=== COMPOSITION & LAYOUT ===
• Direction: left-to-right
• Focal point: dual-path retrieval cluster (center, orange accent)
• Grouping: Input (blue) | Retrieval (orange, highlighted) | Generation (green)
• White space: generous padding between stages, 20px between groups

=== COLOR PALETTE ===
• Primary (#4A90D9): input stage modules
• Secondary (#7EC8A0): generation stage modules
• Accent (#E8855B): retrieval innovation (dual-path + reranker)
• Neutral (#8C8C8C): connection lines, labels
• Background (#FFFFFF)

=== STYLE SPECIFICATIONS ===
• Medium: clean vector illustration, flat design
• Aesthetic: professional, technical, publication-ready
• Shapes: rounded rectangles (8px radius), 1.5px stroke
• Icons: minimal line-style, 24px
• Typography: English labels, sans-serif, clear hierarchy
• Shadows: none
• Gradients: none

=== TECHNICAL SPECS ===
• Resolution: 4K (3840x2160), 300 DPI
• Aspect ratio: 16:9 landscape
• Quality: IEEE/ACM publication standard

=== NEGATIVE PROMPT ===
no watermarks, no blurry, no cluttered, no 3D, no gradients, no shadows, no decorative elements, no stock photo style, no text errors, no overlapping labels, no dark background
```

### 示例 2：后续调用（有 global_style）

**输入：**
```
visualization_goal: 展示 Verifier 模块如何检测并过滤幻觉内容
technical_content: LLM输出→事实校验(知识库比对)→置信度评分→阈值过滤→高置信输出/低置信回退。关键：多粒度校验（句子级+实体级）
scene_description: 分支结构，上方是正常输出路径，下方是回退路径，中间是校验模块
language: 英文
global_style: (粘贴之前生成的 Global Figure Style — RAG System Paper)
```

**输出（只输出 prompt，不重复输出 global_style.md）：**

```
=== SCENE DESCRIPTION ===
A branching verification diagram showing how the Verifier module detects and filters hallucinated content, with the multi-granularity check visually emphasized.
The diagram illustrates LLM output flowing through fact verification (KB comparison), confidence scoring, and threshold-based routing — high-confidence passes through, low-confidence triggers fallback.
Clean flat design on white background, verification module highlighted in warm orange, consistent with project style.

=== VISUAL ELEMENTS ===
Verification Core (Highlighted):
• Fact Verifier: rounded rectangle, #E8855B, center, "Fact Verification (KB)"
• Sentence-level Check: rounded rectangle, #E8855B, center-upper, "Sentence-level"
• Entity-level Check: rounded rectangle, #E8855B, center-lower, "Entity-level"
• Confidence Scorer: rounded rectangle, #E8855B, center-right, "Confidence Scoring"

High-confidence Path:
• Threshold Gate (pass): diamond, #4A90D9, right, "Score >= theta"
• Final Output: rounded rectangle, #7EC8A0, far-right, "High-confidence Output"

Low-confidence Path:
• Threshold Gate (fail): diamond, #8C8C8C, below-right, "Score < theta"
• Fallback: rounded rectangle, #8C8C8C, bottom-right, "Fallback / Re-query"

Connections:
• LLM Output → Fact Verifier: solid arrow, ""
• Fact Verifier → Sentence-level Check: solid arrow, ""
• Fact Verifier → Entity-level Check: solid arrow, ""
• Sentence-level Check → Confidence Scorer: solid arrow, ""
• Entity-level Check → Confidence Scorer: solid arrow, ""
• Confidence Scorer → Threshold Gate (pass): solid arrow, ">= theta"
• Confidence Scorer → Threshold Gate (fail): solid arrow, "< theta"
• Threshold Gate (pass) → Final Output: solid arrow, ""
• Threshold Gate (fail) → Fallback: solid arrow, ""

=== COMPOSITION & LAYOUT ===
• Direction: left-to-right with vertical branch at threshold
• Focal point: multi-granularity verification cluster (center, orange accent)
• Grouping: Verification (orange) | Pass path (blue→green) | Fallback path (gray)
• White space: 20px between groups, branch fork clearly separated

=== COLOR PALETTE ===
• Primary (#4A90D9): threshold gate (pass)
• Secondary (#7EC8A0): final output
• Accent (#E8855B): verification core (multi-granularity)
• Neutral (#8C8C8C): fallback path, threshold gate (fail)
• Background (#FFFFFF)

=== STYLE SPECIFICATIONS ===
• Medium: clean vector illustration, flat design
• Aesthetic: professional, technical, publication-ready
• Shapes: rounded rectangles (8px radius), 1.5px stroke; diamonds for decision
• Icons: minimal line-style, 24px
• Typography: English labels, sans-serif, clear hierarchy
• Shadows: none
• Gradients: none

=== TECHNICAL SPECS ===
• Resolution: 4K (3840x2160), 300 DPI
• Aspect ratio: 16:9 landscape
• Quality: IEEE/ACM publication standard

=== NEGATIVE PROMPT ===
no watermarks, no blurry, no cluttered, no 3D, no gradients, no shadows, no decorative elements, no stock photo style, no text errors, no overlapping labels, no dark background
```

## Common Mistakes

| 错误 | 修正 |
|------|------|
| 使用深色/渐变/3D 效果 | P1 默认禁止；白底、浅色系、flat design |
| 输出 Phase 1/2 推理过程 | 用户只要最终 prompt，不输出中间推理 |
| 让用户手动填模板文件 | 直接交互式收集信息 |
| 生成模糊色值（如 "light blue"） | 必须给 hex 值（如 #4A90D9） |
| 遗漏 negative prompt | 每次输出必须包含 |
| 用户提供了 global_style 时重新生成风格文档 | 直接使用，不重新生成 |
| 首次调用跳过 global_style.md 输出 | 未提供时必须生成并输出 |
| global_style.md 留占位符 | 所有值必须填充具体值 |
| 多轮追问同一信息 | 尽量一次 clarify 搞定 |
