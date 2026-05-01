---
name: doc-writer
version: "v0.9.0"
description: |
  多智能体协同学术/商业文档写作技能，支持单轨/多轨并行/扩轨三种模式，可多支团队同时走不同方向并评选最优解。
  支持 LaTeX 和 Word 双格式输出。

  触发场景（满足任一即触发）：
  - 用户要求撰写、修改、润色学术论文（期刊论文、学位论文、会议论文）
  - 用户要求撰写项目申报书/项目介绍书/可行性报告
  - 用户要求撰写科技查新报告
  - 用户要求撰写商业计划书（BP Executive Summary）
  - 用户要求撰写技术方案文档（系统设计文档）
  - 用户要求生成 LaTeX 或 Word 格式的正式长文档
  - 用户上传模板文件（.tex/.cls/.sty/.docx/.dotx）并要求基于模板写作
  - 用户要求将 Markdown 内容转换为 LaTeX 或 Word 格式
  - 用户询问某种文档类型的标准格式规范
  - 用户要求绘制流程图、折线图、柱状图、饼图、甘特图并插入文档
  - 用户要求多角度/多方案并行写作（如同一题目写多个不同版本）

  不触发场景：
  - 简单的问答或短文本生成（少于500字）
  - 代码生成任务（应触发 code skill）
  - 已完成的正式文档的纯翻译需求

  核心能力：
  - 六 Agent 流水线：规划 → 写作 → 内容优化（Doubao-seed-2.0-pro）→ 图表 → 格式优化 → 审查
  - 三种写作模式：单轨（默认）/ 多轨并行（多方案对比评选）/ 扩轨（多 Agent 并行）
  - 六种内置模板：中文学术论文、IEEE英文期刊、项目申报书、查新报告、商业计划书、技术方案文档
  - 图表扩展支持：流程图/折线图/柱状图/饼图/甘特图/Swimlane图、LaTeX TikZ/pgfplots、Word 原生图表
  - 表格样式增强：三线表/斑马纹表格/跨列跨行单元格
  - 多格式输出：LaTeX (pylatex)、Word (python-docx)、编译说明
  - 自动编译：XeLaTeX → BibTeX → XeLaTeX → XeLaTeX
kind: skill
model:
  default: code
  per_role:
    planner: doubao-seed-2.0-pro
    writer: code
    optimizer: doubao-seed-2.0-pro
    chart-designer: code
    formatter: code
    reviewer: doubao-seed-2.0-pro
---

# 文档写作助手 (doc-writer)

多智能体协同流水线，通过六个专业 Agent 协作完成高质量学术/商业文档写作。

## 写作知识库（v0.9 新增）

> 建立持续积累的写作知识库，沉淀领域术语、格式规范、优秀案例，让每次写作都能站在历史积累的基础上。

### 知识库目录结构

```
references/
├── terminology/          # 领域术语库
│   ├── academic.json    # 学术论文术语（中英对照）
│   ├── business.json     # 商业文档术语
│   ├── engineering.json  # 工程/技术文档术语
│   └── custom.json       # 用户自定义术语
├── format_specs/         # 格式规范库
│   ├── academic_paper.md
│   ├── project_proposal.md
│   ├── novelty_report.md
│   ├── business_plan.md
│   └── technical_doc.md
├── examples/             # 优秀案例库
│   ├── academic/
│   ├── project_proposal/
│   ├── novelty_report/
│   └── business_plan/
└── index.md             # 知识库入口
```

### 术语库使用

**JSON 格式**（`references/terminology/academic.json` 示例）：

```json
{
  "transitions": [
    "首先、其次、最后",
    "然而、与此同时、值得注意的是",
    "因此、综上所述、由此可见"
  ],
  "academic_vocab": {
    "观点": ["认为、指出、表明、论证", "本研究"],
    "分析": ["深入探讨、系统论述、详尽阐述"],
    "结论": ["由此可知、基于此、研究表明"]
  },
  "forbidden": ["从而、所以、然后", "大概、可能、左右"]
}
```

**Writer/Optimizer Agent 在写作时应**：
1. 读取对应领域的 `terminology/*.json`
2. 优先使用 `academic_vocab` 中的正式表达
3. 避免使用 `forbidden` 中的口语化词汇
4. 合理使用 `transitions` 中的过渡词提升流畅度

### 格式规范库

每个格式规范文档（`references/format_specs/*.md`）包含：

```markdown
# [文档类型] 格式规范

## 结构要求
- 章节层级、编号规则
- 字数要求（如申报书一般 8000-15000 字）

## 格式规范
- 标题层级、字体字号
- 段落间距、行间距
- 图表标题位置

## 内容要素
- 必须包含的章节
- 每节的写作要点

## 常见错误
- [错误1] 正确做法：...
- [错误2] 正确做法：...
```

### 优秀案例库

每个案例文件（`references/examples/*/*.md`）标注：

```markdown
---
source: [来源]
type: [论文/申报书/查新报告]
score: [1-5]  # 质量评分
tags: [关键词, 关键词]
---
[正文]
```

**Reviewer Agent 在审查时应**：参考优秀案例库中的高分案例，指出当前文档与优秀案例的差距。

### 知识库更新机制

1. **每次写作完成后**，Writer/Optimizer 将新发现的优秀表达存入 `references/terminology/custom.json`
2. **Reviewer 审查时**发现文档质量高，存入 `references/examples/` 对应目录
3. **用户反馈**指正专业术语时，同步更新 `references/terminology/` 相关文件
4. **Knowledge Base Agent**（新增，可选角色）在空闲时整理和维护知识库内容

### 知识库 Agent（可选，v0.9 新增）

当用户指定"启动知识库管理"或"整理写作知识库"时，额外激活知识库 Agent：

**职责**：
- 整理和维护 `references/` 目录下的所有知识库文件
- 将本次写作中的高频优质表达归档到术语库
- 将评分≥4的文档存入优秀案例库
- 清理过期/低质量案例（每季度或积累≥100条时触发）

**触发方式**：
- 用户明确指定："启动知识库管理"
- 当 `references/examples/` 积累 ≥20 个文档时自动触发整理

### 模型选择说明（v0.9 新增）

| Agent | 任务特点 | 推荐模型 | 理由 |
|-------|---------|---------|------|
| planner | 文档结构规划、多方案设计 | `doubao-seed-2.0-pro` | 复杂推理、结构设计 |
| writer | 内容写作、章节填充 | `code` | 专注输出、性价比高 |
| optimizer | 内容优化、表达精炼 | `doubao-seed-2.0-pro` | 复杂推理、文字优化 |
| chart-designer | 图表代码生成 | `code` | 专注代码、效率高 |
| formatter | 格式调整、LaTeX/Word 排版 | `code` | 重复性操作 |
| reviewer | 质量审查、格式检查 | `doubao-seed-2.0-pro` | 逻辑分析、质量把关 |

## 质量与准确性提升（v0.9 新增）

### 真实性校验

**数据核查**（Reviewer Agent 职责）：
- 论文中的数据、统计结果、实验数据必须标注来源
- 申报书中的市场规模、增长率等数据必须注明引用来源和调查方法
- 无法核实的数据须标注"[数据待核实]"并说明数据来源意向

**引用规范**：
- 学术论文：按期刊要求（APA/GB/T 7714）自动格式化参考文献
- 查新报告：使用国标格式，标注检索数据库名称和时间
- 申报书：标注权威机构发布的数据来源

### 完整性校验

**拓扑依赖处理**（Planner/Optimizer 职责）：
在写作开始前，Planner 输出文档结构拓扑图，明确章节间的依赖关系：

```markdown
## 文档拓扑

A(摘要) → B(背景) → C(研究内容) → D(方法) → E(结果) → F(讨论) → G(结论)

依赖关系：
- A 依赖 B+C+D+E+F+G（从全文摘要）
- B 独立
- C 依赖 B（基于背景展开）
- D 依赖 C（基于内容设计方法）
- E 依赖 D（基于方法得出结果）
- F 依赖 E（讨论结果）
- G 依赖 B+C+D+E+F（总结全文）
```

Optimizer 在优化时检查拓扑顺序是否合理，提前发现章节顺序问题。

### 可运行性校验

**代码/算法文档**（Writer 职责）：
- 代码示例必须可直接运行
- 算法描述中的伪代码应与配套实现代码一致
- 技术方案文档中的架构图应与文字描述对应

## 写作模式

> **启动流水线前请指定写作模式：**

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **单轨模式**（默认） | 单一团队，按流水线顺序完成一篇文档 | 明确需求、直接产出 |
| **多轨并行模式** | 2～4支团队**并行**独立撰写同一主题的不同方案/视角/侧重点，最终由 Leader 评选最优或合并多选 | 创新探索、多角度论证、需要对比方案、同一题目多版本 |
| **扩轨模式** | 单队但扩大规模（如2个写作 Agent 并行，负责不同章节组） | 篇幅长、章节多、内容相对独立 |

**指定方式**：在发起任务时说明，例如"用多轨并行模式，写3个不同技术路线的方案"、"用扩轨模式，4个写作 Agent"

若未指定，默认使用**单轨模式**。

**多轨评审维度**：
- 论点新颖性与说服力
- 论证逻辑严密性
- 结构完整性与层次清晰度
- 数据引用与支撑力度
- 语言表达学术化程度

## 模型选择

> **内容优化 Agent 使用哪个模型？** 在启动流水线前请指定：

| 模型 | 适用场景 | 说明 |
|------|---------|------|
| `doubao-seed-2.0-pro` | 计划/分析/设计类文档 | 推理能力强，适合复杂论证结构、学术论文、项目申报书 |
| `code` | 格式转化/结构填充类任务 | 专注代码/格式输出，适合模板套用、格式规范化、技术报告 |

**指定方式**：在发起任务时说明，例如"用 code 模型优化文本"、"用 doubao 模型润色"

若未指定，默认使用 `doubao-seed-2.0-pro`。

## 流水线架构

### 单轨模式（默认）

```
规划Agent ──→ 写作Agent ──→ 内容优化Agent ──→ 图表Agent ──→ 格式优化Agent ──→ 审查Agent
   │              │                  │                  │               │               │
   ↓              ↓                  ↓                  ↓               ↓               ↓
分析模板       填充内容          Doubao-seed-2.0-pro   插入图表       LaTeX/Word      三重审查
制定计划      章节写作           /code（可切换）      TikZ/pgfplots   格式规范化      逻辑/格式/内容
```

### 多轨并行模式

```
【并行】规划Agent-A ─→ 写作Agent-A ─→ 内容优化Agent-A ─→ 图表Agent-A ─→ 格式优化Agent-A ─→ 报告-A
【并行】规划Agent-B ─→ 写作Agent-B ─→ 内容优化Agent-B ─→ 图表Agent-B ─→ 格式优化Agent-B ─→ 报告-B
【并行】规划Agent-C ─→ 写作Agent-C ─→ 内容优化Agent-C ─→ 图表Agent-C ─→ 格式优化Agent-C ─→ 报告-C
      ↓ 全部完成后
Leader 对比评审：评选最优方案（可多选合并），输出最终采纳文档
```

> **多轨模式下各团队策略建议**：
> - Team A：主流技术路线，传统方案
> - Team B：创新技术路线，探索性方案
> - Team C：折中/混合方案，兼顾可行性与创新性

### 扩轨模式

```
规划Agent × 1
     ↓ 质量门
写作Agent × 2～4（并行，按章节组分工）
     ↓ 质量门
内容优化Agent × 2（并行）
     ↓ 质量门
图表Agent × 1
     ↓ 质量门
格式优化Agent × 1
     ↓ 质量门
审查Agent × 1
     ↓
Final: 合并各章节，输出完整文档
```

### Agent 职责矩阵

| Agent | 核心职责 | 输出物 | 失败降级 |
|-------|---------|--------|---------|
| **规划Agent** | 解析模板结构 + 制定写作计划 | 章节大纲 + 字数分配 + 写作策略 | 单 Agent 模式（规划+写作合并） |
| **写作Agent** | 按模板格式填充章节内容 | 完整正文（Markdown） | 直接输出大纲级内容 |
| **内容优化Agent** | 调用 Doubao-seed-2.0-pro 或 code 优化文本（可切换模型） | 润色后的 Markdown 正文 | 跳过优化，使用原始内容 |
| **图表Agent** | TikZ 代码生成 / Word 图表插入 | 图表代码块 + 插入位置标记 | 跳过图表步骤，提示用户手动插入 |
| **格式优化Agent** | LaTeX pylatex 脚本 / Word python-docx 脚本 | 格式化的 .tex / .docx 文件 | 输出 Markdown + 转换命令 |
| **审查Agent** | 逻辑完整性 + 格式规范性 + 内容质量 | 审查报告 + 修改建议 | 跳过审查，用户自行检查 |

kind: skill
---

## 第一阶段：模板解析与需求确认

### 模板文件解析逻辑

当用户上传模板文件时，按以下流程处理：

```
用户上传模板 → 检测文件类型 → 解析格式规范 → 提取章节结构 → 加载为写作依据
```

**支持的模板格式：**

| 格式 | 解析方式 | 提取内容 |
|------|---------|---------|
| .tex | 正则提取 `\section{}`, `\chapter{}`, `documentclass` | 章节结构、文档类、宏包 |
| .cls / .sty | 读取类/宏包定义 | 格式规范、命令定义 |
| .docx / .dotx | python-docx + lxml 解析 | 样式定义、章节标题、段落格式 |

**模板解析 Agent 指令：**
```
你是一名模板解析专家。请分析用户提供的模板文件，提取以下信息：

1. 文档类型识别（论文/报告/书籍/其他）
2. 章节结构提取（按层级列出所有标题）
3. 格式规范提取：
   - 文档类/模板名称
   - 字体要求（中文/英文/字号）
   - 页面布局（边距、纸张大小）
   - 行距要求
   - 标题层级样式
   - 表格/图片规范
4. 特殊宏包或命令定义
5. 示例内容片段（用于理解写作风格）

以结构化 JSON 格式输出，示例：
{
  "doc_type": "ctexart",
  "chapters": [{"level": 1, "title": "摘要", "content_hints": ["研究背景", "方法", "结论"]}, ...],
  "format": {
    "paper": "a4paper",
    "font_size": "12pt",
    "line_spacing": "1.5",
    "chinese_font": "宋体",
    "english_font": "Times New Roman"
  },
  "special_commands": ["\\citep{}", "\\figurename{}"]
}
```

### 需求确认清单

当信息不足时，向用户确认以下内容：

```
【文档写作需求确认】

为确保文档质量，请在开始前确认以下信息：

1. 文档类型：
   □ 中文学术论文（ctexart）
   □ 英文期刊论文（IEEEtran）
   □ 项目申报书
   □ 科技查新报告
   □ 商业计划书（BP Executive Summary）
   □ 技术方案文档（系统设计文档）
   □ 其他：__________

2. 目标格式：
   □ LaTeX (.tex)
   □ Word (.docx)
   □ 两者均需要

3. 是否有参考模板：
   □ 有（请上传文件）
   □ 无，使用内置模板

4. 主题领域：__________
5. 大致篇幅：__________ 字
6. 特殊要求：__________

如未明确指定，默认采用：
- 学术论文 → LaTeX
- 商业报告 → Word
- 文档类 → ctexart / IEEEtran
```

kind: skill
---

## 第二阶段：规划 Agent

**角色定义：** 文档架构师，负责分析模板结构并制定详细的写作计划。

**输入契约：**
- 文档类型
- 目标读者
- 参考模板（可选）
- 特殊要求

**输出契约：**
```json
{
  "outline": [
    {
      "level": 1,
      "title": "章节标题",
      "subsections": ["小节1", "小节2"],
      "core_points": ["核心要点1", "核心要点2"],
      "estimated_words": 500,
      "writing_style": "正式/半正式/通俗"
    }
  ],
  "writing_strategy": "整体写作策略说明",
  "word_distribution": {"摘要": 300, "引言": 1000, ...},
  "reference_style": "GB/T 7714 / IEEE / APA",
  "special_requirements": ["图表比例", "公式编号", "附录要求"]
}
```

**规划 Agent 完整指令：**
```
你是一名专业的学术文档架构师。请根据以下信息制定详细的写作计划。

## 输入信息

文档类型：[类型]
主题领域：[领域]
目标读者：[读者群体]
目标格式：[LaTeX/Word]
参考模板：[模板内容，如有]
特殊要求：[要求列表]

## 输出要求

请输出完整的写作计划，包含：

### 1. 文档大纲
按层级列出所有章节，每个章节包含：
- 章节标题（中英文对照，如适用）
- 核心内容要点（3-5个）
- 预估字数
- 推荐写作风格

### 2. 字数分配
为每个章节分配合理字数，确保整体篇幅均衡。

### 3. 写作策略
- 整体论证结构（总-分-总/递进式/并列式）
- 数据/案例使用建议
- 引用风格说明

### 4. 格式要点
- 图表数量及大致位置
- 公式数量（如有）
- 参考文献数量建议

### 5. 风险提示
- 可能遇到的难点
- 需要用户确认的内容

以 JSON 格式返回，结构参见输出契约。
```

kind: skill
---

## 第三阶段：写作 Agent

**角色定义：** 专业文档写作者，负责按计划填充完整内容。

**输入契约：**
- 规划 Agent 输出的大纲
- 用户提供的素材/参考资料
- 写作风格要求

**输出契约：**
- 完整正文内容（Markdown 格式）
- 每个章节独立成块，便于后续处理
- 图表插入占位符 `{{FIGURE:图号:题注}}`、`{{TABLE:表号:标题}}`

**写作 Agent 完整指令：**
```
你是一名专业的学术/商业文档写作者。请根据以下大纲撰写完整文档。

## 写作计划（来自规划Agent）

[粘贴规划Agent的输出JSON]

## 用户素材

[用户提供的参考资料、数据、案例]

## 写作要求

1. **内容充实**：每个章节都要有实质性内容，避免空洞框架
2. **逻辑连贯**：段落之间使用过渡句，章节之间逻辑衔接
3. **数据支撑**：适当使用数据、案例、引用增强说服力
4. **引用规范**：所有论点必须有论文/文献支撑，**禁止随意编造数据**，引用格式 `\citep{key}` 或 `\citet{key}`
5. **格式规范**：
   - 中文使用中文标点，英文/数字使用英文标点
   - 专有名词首次出现需括号标注英文
   - 避免口语化表达
6. **图表占位**：需要在图表位置插入占位符：
   - `{{FIGURE:1:这是一个流程图}}` - 图片占位
   - `{{TABLE:1:这是一个数据表}}` - 表格占位
7. **引用格式**：使用 `\citep{key}` 编号引用，文末 `references.bib` 中必须包含对应条目

## 输出格式

直接输出完整 Markdown 文档，包含所有章节内容。不要仅输出框架，要输出完整正文。

每个主章节以 `# 章节名` 开头，
子章节以 `## 子章节名` 开头。

文档末尾添加 `{{END_OF_DRAFT}}` 标记。
```

kind: skill
---

## 第四阶段：内容优化 Agent

**角色定义：** 语言润色与逻辑强化专家，负责调用 Doubao-seed-2.0-pro 提升文档内容质量。

**输入契约：**
- 写作 Agent 输出的 Markdown 文档
- 文档类型与写作风格要求

**输出契约：**
- 润色优化后的完整 Markdown 正文
- 保留原有 `\citep{key}` 引用和 `{{FIGURE:N:...}}` 等占位符

**内容优化 Agent 完整指令：**
```
你是一名专业的学术文档语言优化专家。请对以下文档进行深度润色和逻辑强化。

## 原始文档

[粘贴内容优化Agent输出的完整Markdown]

## 文档类型：[论文/申报书/查新报告]

## 优化要求

1. **语言润色**：
   - 消除口语化表达，使语言更加正式、学术化
   - 优化句式结构，使表达更加清晰流畅
   - 统一专业术语使用

2. **逻辑强化**：
   - 加强段落之间的逻辑衔接
   - 强化论点论证的严密性
   - 确保章节之间的过渡自然

3. **保留内容**：
   - **必须保留**所有 `\citep{key}` 引用格式
   - **必须保留**所有 `{{FIGURE:N:题注}}`、`{{TABLE:N:标题}}` 占位符
   - **必须保留**加粗关键词格式 `\textbf{...}`
   - **不要改动数据内容**，只优化表述方式

4. **禁止行为**：
   - 禁止添加或删除引用
   - 禁止编造新的数据或结论
   - 禁止改变文档结构

## 输出格式

直接输出优化后的完整 Markdown 文档，末尾添加 `{{END_OF_OPTIMIZED}}` 标记。
```

kind: skill
---

## 第五阶段：图表 Agent

**角色定义：** 图表工程师，负责生成 LaTeX TikZ/pgfplots 代码或 Word 原生图表。

**输入契约：**
- 内容优化 Agent 输出的 Markdown 文档
- 图表类型需求（流程图/折线图/柱状图/饼图/甘特图/Swimlane图/表格/图片）
- 目标格式（LaTeX/Word）

**输出契约：**
- 各图表的完整代码/命令
- 插入位置的行号标记
- 图表文件列表（如有独立图片）

### 图表类型与代码模板

#### F005: LaTeX TikZ 流程图

```latex
% 流程图模板 - 可直接使用
\usepackage{tikz}
\usetikzlibrary{shapes.geometric, arrows, positioning, calc}

% 样式定义
\tikzstyle{startstop} = [rectangle, rounded corners, minimum width=3cm, minimum height=1cm, text centered, draw=black, fill=blue!20]
\tikzstyle{process} = [rectangle, minimum width=3cm, minimum height=1cm, text centered, draw=black, fill=yellow!20]
\tikzstyle{decision} = [diamond, minimum width=3cm, minimum height=1.5cm, text centered, draw=black, fill=green!20]
\tikzstyle{arrow} = [thick,->,>=stealth]

\begin{tikzpicture}[node distance=1.5cm]

% 节点定义
\node(start) [startstop] {开始};
\node(proc1) [process, below of=start, yshift=-0.5cm] {步骤1};
\node(proc2) [process, below of=proc1, yshift=-0.5cm] {步骤2};
\node(decision1) [decision, below of=proc2, yshift=-1cm] {条件判断};
\node(proc3a) [process, below left of=decision1, xshift=-2cm] {分支A};
\node(proc3b) [process, below right of=decision1, xshift=2cm] {分支B};
\node(stop) [startstop, below of=decision1, yshift=-2cm] {结束};

% 连接线
\draw [arrow] (start) -- (proc1);
\draw [arrow] (proc1) -- (proc2);
\draw [arrow] (proc2) -- (decision1);
\draw [arrow] (decision1) -- node[anchor=east] {是} (proc3a);
\draw [arrow] (decision1) -- node[anchor=west] {否} (proc3b);
\draw [arrow] (proc3a) -| ($(proc2)+(2,0)$) -- (proc2);
\draw [arrow] (proc3b) -- (stop);
\draw [arrow] (proc3a) |- ($(stop)+(-2,0)$);

\end{tikzpicture}
```

#### F005: LaTeX pgfplots 折线图

```latex
% 折线图模板 - 可直接使用
\usepackage{pgfplots}
\pgfplotsset{width=10cm, height=6cm, compat=1.18}

\begin{figure}[htbp]
\centering
\begin{tikzpicture}
\begin{axis}[
  xlabel={X轴标签},
  ylabel={Y轴标签},
  xmin=0, xmax=10,
  ymin=0, ymax=100,
  legend pos=north east,
  grid=major,
  grid style={dashed, gray!30},
  title={图表标题}
]

% 数据系列1
\addplot[mark=o, color=blue, line width=2pt] coordinates {
  (0, 10)
  (2, 25)
  (4, 45)
  (6, 65)
  (8, 82)
  (10, 95)
};
\addlegendentry{系列1};

% 数据系列2
\addplot[mark=square, color=red, line width=2pt] coordinates {
  (0, 15)
  (2, 30)
  (4, 50)
  (6, 68)
  (8, 78)
  (10, 88)
};
\addlegendentry{系列2};

\end{axis}
\end{tikzpicture}
\caption{折线图题注}\label{fig:line}
\end{figure}
```

#### F005: LaTeX pgfplots 柱状图

```latex
% 柱状图模板 - 可直接使用
\usepackage{pgfplots}
\usepackage{pgfplotstable}
\pgfplotsset{width=10cm, height=6cm, compat=1.18}

\begin{figure}[htbp]
\centering
\begin{tikzpicture}
\begin{axis}[
  ybar,
  bar width=0.6cm,
  symbolic x coords={A类, B类, C类, D类, E类},
  xtick=data,
  xlabel={类别},
  ylabel={数值},
  legend pos=north east,
  ymin=0,
  grid=major,
  grid style={dashed, gray!30},
  title={柱状图标题}
]

\addplot[fill=blue!60] coordinates {
  (A类, 45)
  (B类, 62)
  (C类, 53)
  (D类, 78)
  (E类, 35)
};
\addlegendentry{2024年数据};

\addplot[fill=red!60] coordinates {
  (A类, 38)
  (B类, 55)
  (C类, 48)
  (D类, 65)
  (E类, 30)
};
\addlegendentry{2023年数据};

\end{axis}
\end{tikzpicture}
\caption{柱状图题注}\label{fig:bar}
\end{figure}
```

#### F008: LaTeX 三线表

```latex
% 三线表模板 - 可直接使用
\usepackage{booktabs}
\usepackage{multirow}

\begin{table}[htbp]
\centering
\caption{表格标题}\label{tab:example}
\begin{tabular}{ccccl}
\toprule
\multirow{2}*{项目} & \multicolumn{2}{c}{实验组} & \multicolumn{2}{c}{对照组} \\
\cmidrule(lr){2-3} \cmidrule(lr){4-5}
          & 均值   & 标准差   & 均值   & 标准差 \\
\midrule
指标1     & 85.6  & 3.2     & 78.3  & 4.1   \\
指标2     & 92.1  & 2.8     & 86.5  & 3.5   \\
指标3     & 78.9  & 4.5     & 71.2  & 5.2   \\
\bottomrule
\end{tabular}
\end{table}
```

#### F007: LaTeX 图片插入

```latex
% 图片插入模板 - 可直接使用
\usepackage{graphicx}

\begin{figure}[htbp]
\centering
includegraphics[width=0.8\textwidth]{image.pdf}
\caption{图片题注}\label{fig:demo}
\end{figure}

% 子图示例
\usepackage{subcaption}
\begin{figure}[htbp]
\centering
\begin{subcaptionbox}{0.45\textwidth}{
  \includegraphics[width=\textwidth]{fig1.pdf}
  \caption{子图1}
  \label{fig:sub1}
}
\begin{subcaptionbox}{0.45\textwidth}{
  \includegraphics[width=\textwidth]{fig2.pdf}
  \caption{子图2}
  \label{fig:sub2}
}
\end{subcaption}
```

#### F009: LaTeX pgf-pie 饼图

```latex
% 饼图模板 - 可直接使用
\usepackage{pgf-pie}

\begin{figure}[htbp]
\centering
\begin{tikzpicture}
\pie[
  rotate=180,
  color={blue!60, red!60, green!60, orange!60, purple!60},
  text=legend,
  explode=0.1,
  style={font=\footnotesize},
]{
  35/产品A,
  25/产品B,
  20/产品C,
  12/产品D,
  8/其他
}
\end{tikzpicture}
\caption{饼图题注}\label{fig:pie}
\end{figure}
```

#### F010: LaTeX pgfgantt 甘特图

```latex
% 甘特图模板 - 可直接使用
\usepackage{pgfgantt}
\usepackage{xcolor}

\begin{figure}[htbp]
\centering
\begin{ganttchart}[
  hgrid,
  vgrid,
  time slot format=isodate,
  group incomplete node=black,
  milestone incomplete node=black,
]{2024-01-01}{2024-06-30}
  \gantttitlecalendar{year, month=name} \\
  \ganttgroup{项目整体}{2024-01-01}{2024-06-30} \\
  \ganttbar{阶段一：需求分析}{2024-01-01}{2024-02-15} \\
  \ganttbar{阶段二：设计开发}{2024-02-16}{2024-05-15} \\
  \ganttlinkedbar{阶段三：测试验收}{2024-05-16}{2024-06-15} \\
  \ganttmilestone{项目上线}{2024-06-30}
\end{ganttchart}
\caption{甘特图题注}\label{fig:gantt}
\end{figure}
```

#### F011: LaTeX Swimlane 跨职能流程图

```latex
% Swimlane（跨职能流程图）模板 - 可直接使用
\usepackage{tikz}
\usetikzlibrary{shapes.geometric, arrows, positioning, fit}

\begin{figure}[htbp]
\centering
\begin{tikzpicture}[
  lane/.style={rectangle, minimum height=1.5cm, minimum width=14cm, draw=black, fill=none},
  task/.style={rectangle, rounded corners, minimum width=2.5cm, minimum height=0.8cm, draw=black, fill=white},
  arrow/.style={thick,->,>=stealth}
]

% 泳道定义
\node[lane] (lane1) at (0, 0) {业务部门};
\node[lane] (lane2) at (0, -1.8) {技术部门};
\node[lane] (lane3) at (0, -3.6) {运维部门};

% 任务节点
\node[task] (t1) at (2, 0) {提交需求};
\node[task] (t2) at (2, -1.8) {技术评估};
\node[task] (t3) at (6, -1.8) {方案设计};
\node[task] (t4) at (10, -1.8) {开发实现};
\node[task] (t5) at (6, -3.6) {部署上线};

% 连接线
\draw[arrow] (t1) -- (t2);
\draw[arrow] (t2) -- (t3);
\draw[arrow] (t3) -- (t4);
\draw[arrow] (t4) -- ++(0,-1) -| (t5);
\draw[arrow] (t5) -- ++(2,0) node[anchor=west] {完成};

\end{tikzpicture}
\caption{Swimlane流程图题注}\label{fig:swimlane}
\end{figure}
```

#### F012: LaTeX 斑马纹表格

```latex
% 斑马纹表格模板 - 可直接使用
\usepackage[table]{xcolor}
\usepackage{booktabs}

\rowcolors{1}{gray!10}{white}  % 奇数行灰色，偶数行白色

\begin{table}[htbp]
\centering
\caption{斑马纹表格标题}\label{tab:zebra}
\begin{tabular}{clccc}
\toprule
序号 & 项目名称 & 数值1 & 数值2 & 备注 \\
\midrule
1    & 项目A    & 85.6  & 92.1  & 优秀   \\
2    & 项目B    & 78.3  & 86.5  & 良好   \\
3    & 项目C    & 92.1  & 95.8  & 卓越   \\
4    & 项目D    & 65.4  & 71.2  & 合格   \\
5    & 项目E    & 88.7  & 90.3  & 优秀   \\
\bottomrule
\end{tabular}
\end{table}
```

#### F013: LaTeX 跨列跨行复杂表格

```latex
% 跨列跨行表格模板 - 可直接使用
\usepackage{booktabs}
\usepackage{multirow}

\begin{table}[htbp]
\centering
\caption{跨列跨行表格标题}\label{tab:complex}
\begin{tabular}{cc|ccc|c}
\toprule
\multirow{2}{*}{维度} & \multirow{2}{*}{指标} & \multicolumn{3}{c}{测量结果} & \multirow{2}{*}{评价} \\
\cmidrule{3-5}
                       &            & 数值1    & 数值2    & 数值3     &            \\
\midrule
\cellcolor{blue!20}区域A & 均值       & 85.6    & 92.1    & 88.9     & 优秀       \\
                        & 标准差     & 3.2     & 2.8     & 4.1      & —         \\
\cellcolor{green!20}区域B & 均值       & 78.3    & 86.5    & 82.4     & 良好       \\
                        & 标准差     & 4.1     & 3.5     & 3.8      & —         \\
\bottomrule
\end{tabular}
\end{table}
```

> **图表模板说明**：以上模板可直接复制使用，数据部分按实际内容替换即可。LaTeX 图表使用 `\usepackage{booktabs}`（三线表）、`\usepackage{pgf-pie}`（饼图）、`\usepackage{pgfgantt}`（甘特图）、`\usepackage{multirow}`（跨行表格）。

#### F006: Word 原生图表（Python 脚本）

```python
# /// script
# dependencies = ['python-docx']
# ///

from docx.shared import Inches, Pt, Cm, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.chart import XL_CHART_TYPE, XL_LEGEND_POSITION
from docx.enum.table import WD_TABLE_ALIGNMENT
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def add_line_chart(doc, title, x_label, y_label, series_data):
    """插入折线图

    Args:
        doc: python-docx Document 对象
        title: 图表标题
        x_label: X轴标签
        y_label: Y轴标签
        series_data: list of (series_name, [(x1,y1), (x2,y2), ...])
    """
    chart = doc.add_chart(XL_CHART_TYPE.LINE, width=Inches(5), height=Inches(3))
    chart.has_title = True
    chart.chart_title.text_frame.text = title

    # 设置图例
    chart.has_legend = True
    chart.legend.position = XL_LEGEND_POSITION.RIGHT

    # 写入数据
    chart_data = chart._element.chart.plot.series[0]
    # 注意：实际使用需要填充数据表

    return chart

def add_bar_chart(doc, title, categories, values, series_name="数据"):
    """插入柱状图"""
    chart = doc.add_chart(XL_CHART_TYPE.COLUMN_CLUSTERED, width=Inches(5), height=Inches(3))
    chart.has_title = True
    chart.chart_title.text_frame.text = title
    chart.has_legend = True
    chart.legend.include_in_layout = False

    # 填充数据
    chart.chart_data.data_range
    # ... 具体数据填充逻辑

    return chart

def add_table(doc, headers, rows, caption=None):
    """插入标准三线表

    Args:
        doc: Document 对象
        headers: 表头列表
        rows: 数据行列表
        caption: 表格标题（可选）
    """
    table = doc.add_table(rows=len(rows)+1, cols=len(headers))
    table.style = 'Table Grid'

    # 表头
    header_cells = table.rows[0].cells
    for i, h in enumerate(headers):
        header_cells[i].text = h
        # 设置表头样式（加粗、居中、底边线）
        for p in header_cells[i].paragraphs:
            p.alignment = WD_ALIGN_PARAGRAPH.CENTER
            for run in p.runs:
                run.font.bold = True

    # 数据行
    for row_idx, row_data in enumerate(rows):
        row_cells = table.rows[row_idx + 1].cells
        for col_idx, cell_text in enumerate(row_data):
            row_cells[col_idx].text = str(cell_text)

    # 添加表格标题
    if caption:
        cap_para = doc.add_paragraph(caption, style='Caption')

    return table
```

#### F006: Word 图片插入（Python 脚本）

```python
# /// script
# dependencies = ['python-docx']
# ///

def add_picture_with_caption(doc, image_path, caption_text, width=Inches(5)):
    """插入图片并添加题注

    Args:
        doc: Document 对象
        image_path: 图片路径（本地或URL）
        caption_text: 题注文本
        width: 图片宽度
    """
    # 插入图片
    run = doc.add_paragraph().add_run()
    run.add_picture(image_path, width=width)

    # 添加题注
    caption = doc.add_paragraph()
    caption.style = 'Caption'
    caption.add_run(caption_text)

    return caption

# 示例调用
# doc = Document()
# add_picture_with_caption(doc, 'figure1.png', '图 1 实验结果对比', width=Inches(5))
```

### 图表 Agent 工作流程

```
遍历写作输出的 {{FIGURE:N:题注}} 和 {{TABLE:N:标题}} 占位符
    ↓
识别图表类型（流程图/折线图/柱状图/饼图/甘特图/Swimlane图/表格/普通图片）
    ↓
生成对应格式的图表代码
    ↓
输出：{chart_id, type, code, insert_line}
```

**图表 Agent 完整指令：**
```
你是一名专业的图表工程师。请根据文档中的图表占位符生成对应的图表代码。

## 文档内容

[粘贴写作Agent输出的完整文档]

## 目标格式：[LaTeX/Word]

## 图表占位符列表

识别文档中所有 {{FIGURE:N:题注}} 和 {{TABLE:N:标题}} 标记。

## 输出要求

对于每个图表占位符，输出：

1. **图表类型判断**：流程图/折线图/柱状图/饼图/甘特图/Swimlane图/表格/普通图片
2. **数据提取**：从上下文提取图表所需的数据
3. **代码生成**：根据目标格式生成对应代码

### LaTeX 输出格式
```latex
% === 图 [编号] ===
% 位置：章节 [X]
% 类型：[流程图/折线图/柱状图/表格]
[完整的LaTeX代码]
```

### Word 输出格式
```python
def insert_chart_[编号](doc):
    """插入图 [编号]：题注
    类型：[流程图/折线图/柱状图/表格]
    """
    # Python 代码
    pass
```

## 代码要求

1. LaTeX 代码必须完整可编译，包含必要的包声明
2. Word 代码使用 python-docx API，函数化设计
3. 代码中包含详细注释
4. 为图表分配合理的数据（如上下文未提供，使用示例数据并在注释中说明）

完成后列出所有生成的图表代码及插入位置。
```

kind: skill
---

## 第六阶段：格式优化 Agent

**角色定义：** 格式工程师，负责将 Markdown 内容转换为 LaTeX 或 Word 格式。

**输入契约：**
- 内容优化 Agent 输出的完整 Markdown
- 图表 Agent 输出的图表代码
- 目标格式（LaTeX/Word）
- 文档类型

**输出契约：**
- 格式化的 .tex 或 .docx 文件路径
- 编译/生成说明
- 依赖文件列表

### 内置格式模板

#### F002-1: 中文学术论文（ctexart）

**LaTeX 格式规范：**
```latex
% 文档类声明
\documentclass[12pt,a4paper]{ctexart}

% ===== 页面设置 =====
\usepackage{geometry}
\geometry{top=2.8cm, bottom=2.5cm, left=3cm, right=2.6cm}
\setlength{\headheight}{14.5pt}

% ===== 数学公式 =====
\usepackage{amsmath}
\usepackage{amssymb}

% ===== 图片 =====
\usepackage{graphicx}
\usepackage{float}
\usepackage{subfigure}

% ===== 表格 =====
\usepackage{booktabs}
\usepackage{tabularx}
\usepackage{multirow}
\usepackage{makecell}
\usepackage{array}
\usepackage{threeparttable}

% ===== 参考文献 =====
\usepackage[nottoc]{tocbibind}
\usepackage[numbers,sort&compress,super,square]{natbib}

% ===== 超链接（无颜色可跳转） =====
\usepackage{hyperref}
\hypersetup{
    bookmarks=true,
    bookmarksnumbered=true,
    hidelinks
}
\usepackage{bookmark}

% ===== 标题格式 =====
\usepackage{titlesec}
\usepackage{titletoc}
\titleformat{\section}{\bfseries\heiti\fontsize{14pt}{18pt}\selectfont}{\thesection}{1em}{}
\titleformat{\subsection}{\bfseries\kaishu\fontsize{13pt}{16pt}\selectfont}{\thesubsection}{1em}{}
\titleformat{\subsubsection}{\bfseries\fontsize{12pt}{15pt}\selectfont}{\thesubsubsection}{1em}{}

% ===== 目录 =====
\makeatletter
\def\@dotsep{3.5}
\makeatother

% ===== 排版间距 =====
\setlength{\parindent}{2em}
\setlength{\parskip}{0pt}
\linespread{1.67}

% ===== 图表编号 =====
\usepackage{chngcntr}
\counterwithout{figure}{section}
\counterwithout{table}{section}

% ===== 自定义命令 =====
\def\mytitle#1{{\fontsize{18pt}{22pt}\bfseries\selectfont #1}}
\def\mybody#1{{\fontsize{12pt}{15pt}\selectfont #1}}

% ===== 摘要格式 =====
\renewenvironment{abstract}{%
  \thispagestyle{empty}
  \vspace*{0.5cm}
  \begin{center}
    \vspace*{1.5cm}
    \mytitle{论文标题}\\[1em]
    {\heiti\zihao{3} 文档类型}\\[2em]
    {\kaishu\zihao{-4} 作者信息}\\[0.5em]
    {\kaishu\zihao{-4} 日期}
  \end{center}
  \vspace*{1.5cm}
  \begin{center}
    {\heiti\zihao{3}\textbf{摘\quad 要}}
  \end{center}
  \begin{center}
  \parbox{0.9\textwidth}{
  \kaishu\zihao{-4}
  摘要正文内容...
  \\[2ex]
  \noindent\textbf{关键词：}关键词1；关键词2；关键词3
  }
  \end{center}
  \vspace*{\fill}
  \newpage
}{}

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 文档页眉}
\fancyfoot[C]{\thepage}

% ===== 参考文献标题 =====
renewcommand{\bibname}{参考文献}

\begin{document}

% ===== 封面 + 摘要（同一页）=====
\begin{ abstract }

% ===== 目录 =====
\tableofcontents
\newpage

% ===== 正文章节 =====

% ===== 参考文献 =====
\nocite{*}
\bibliographystyle{plainnat}
\bibliography{references}

\vspace*{2cm}
\begin{flushright}
\textbf{编写人}: 作者 \\
\textbf{日期}: 2026年4月
\end{flushright}

\end{document}
```

**Word 格式规范：**
| 元素 | 格式要求 |
|------|---------|
| 纸张 | A4 |
| 页边距 | 上2.8cm，下2.5cm，左3.0cm，右2.6cm |
| 标题1 | 黑体，三号，居中 |
| 标题2 | 黑体，四号，左对齐 |
| 正文 | 宋体，12pt，1.67倍行距 |
| 首行缩进 | 2字符 |
| 英文/数字 | Times New Roman，12pt |
| 图表题注 | 宋体，五号，居中 |
| 关键词 | 加粗 |

#### F002-2: IEEE 英文期刊论文

**LaTeX 格式规范：**
```latex
% IEEEtran 文档类
\documentclass[journal, a4paper, 10pt, onecolumn]{IEEEtran}

% 常用宏包
\usepackage{amsmath, amssymb}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{cite}
\usepackage{hyperref}

% 标题区域
\title{论文标题}
\author{
    \IEEEauthorblockN{作者姓名}
    \IEEEauthorblockA{所属机构\\ 城市，国家\\ 邮箱}
}

\begin{document}

\maketitle

% 摘要
\begin{abstract}
This paper presents...
\textbf{Index Terms}— keyword1, keyword2, keyword3
\end{abstract}

% 正文
\section{Introduction}
\section{Related Work}
\section{Proposed Method}
\section{Experimental Results}
\section{Conclusion}

% 参考文献
\bibliographystyle{IEEEtran}
\bibliography{references}

\end{document}
```

**Word 格式规范：**
| 元素 | 格式要求 |
|------|---------|
| 纸张 | A4 |
| 页边距 | 上下2.54cm，左右3.17cm |
| 字体 | Times New Roman，10pt |
| 行距 | 单倍行距 |
| 标题 | 居中，加粗 |

#### F002-3: 项目申报书

**格式规范：**
| 元素 | 要求 |
|------|------|
| 纸张 | A4 |
| 页边距 | 上2.8cm，下2.5cm，左3.0cm，右2.6cm |
| 标题 | 黑体，18pt，居中 |
| 正文 | 12pt，1.67倍行距 |
| 首行缩进 | 2字符 |
| 超链接 | 无颜色无框，可跳转 |
| 参考文献 | natbib，上标编号 |

**LaTeX 结构模板：**
```latex
\documentclass[12pt,a4paper]{ctexart}

% ===== 页面设置 =====
\usepackage{geometry}
\geometry{top=2.8cm, bottom=2.5cm, left=3cm, right=2.6cm}
\setlength{\headheight}{14.5pt}

% ===== 数学公式 =====
\usepackage{amsmath}
\usepackage{amssymb}

% ===== 图片 =====
\usepackage{graphicx}
\usepackage{float}
\usepackage{subfig}

% ===== 表格 =====
\usepackage{booktabs}
\usepackage{tabularx}
\usepackage{multirow}
\usepackage{makecell}
\usepackage{array}
\usepackage{threeparttable}

% ===== 参考文献 =====
\usepackage[nottoc]{tocbibind}
\usepackage[numbers,sort&compress,super,square]{natbib}

% ===== 超链接（无颜色可跳转） =====
\usepackage{hyperref}
\hypersetup{
    bookmarks=true,
    bookmarksnumbered=true,
    hidelinks,
}
\usepackage{bookmark}

% ===== 标题格式 =====
\usepackage{titlesec}
\usepackage{titletoc}
\titleformat{\section}{\bfseries\heiti\fontsize{14pt}{18pt}\selectfont}{\thesection}{1em}{}
\titleformat{\subsection}{\bfseries\kaishu\fontsize{13pt}{16pt}\selectfont}{\thesubsection}{1em}{}
\titleformat{\subsubsection}{\bfseries\fontsize{12pt}{15pt}\selectfont}{\thesubsubsection}{1em}{}

% ===== 目录 =====
\makeatletter
\def\@dotsep{3.5}
\makeatother

% ===== 排版间距 =====
\setlength{\parindent}{2em}
\setlength{\parskip}{0pt}
\linespread{1.67}

% ===== 图表编号 =====
\usepackage{chngcntr}
\counterwithout{figure}{section}
\counterwithout{table}{section}

% ===== 自定义命令 =====
\def\mytitle#1{{\fontsize{18pt}{22pt}\bfseries\selectfont #1}}
\def\mybody#1{{\fontsize{12pt}{15pt}\selectfont #1}}

% ===== 摘要格式 =====
\renewenvironment{ abstract }{%
  \thispagestyle{empty}
  \vspace*{0.5cm}
  \begin{center}
    \vspace*{1.5cm}
    \mytitle{项目名称}\\[1em]
    {\heiti\zihao{3} 项目申报书}\\[2em]
    {\kaishu\zihao{-4} 申报单位/团队}\\[0.5em]
    {\kaishu\zihao{-4} 日期}
  \end{center}
  \vspace*{1.5cm}
  \begin{center}
    {\heiti\zihao{3}\textbf{摘\quad 要}}
  \end{center}
  \begin{center}
  \parbox{0.9\textwidth}{
  \kaishu\zihao{-4}
  摘要正文内容...
  \\[2ex]
  \noindent\textbf{关键词：}关键词1；关键词2；关键词3
  }
  \end{center}
  \vspace*{\fill}
  \newpage
}{}

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 项目申报书}
\fancyfoot[C]{\thepage}

% ===== 参考文献标题 =====
renewcommand{\bibname}{参考文献}

\begin{document}

% ===== 封面 + 摘要（同一页）=====
\begin{ abstract }

% ===== 目录 =====
\tableofcontents
\newpage

% ===== 正文章节 =====

% ===== 参考文献 =====
\nocite{*}
\bibliographystyle{plainnat}
\bibliography{references}
\newpage

\vspace*{2cm}
\begin{flushright}
\textbf{编写人}: 作者 \\
\textbf{日期}: 2026年4月
\end{flushright}

\end{document}
```

#### F002-4: 科技查新报告

**格式规范：**
| 元素 | 要求 |
|------|------|
| 纸张 | A4 |
| 页边距 | 上2.8cm，下2.5cm，左3.0cm，右2.6cm |
| 标题 | 黑体，18pt，居中 |
| 正文 | 12pt，1.67倍行距 |
| 首行缩进 | 2字符 |
| 超链接 | 无颜色无框，可跳转 |
| 参考文献 | natbib，上标编号 |
| 结构 | 报告封面→摘要→目录→查新内容与要求→检索范围与策略→检索结果→查新结论→附件→参考文献 |
| 查新点 | 编号列出，每个查新点独立段落 |
| 密切相关文献 | 需要详细分析对比 |

**LaTeX 结构模板：**
```latex
\documentclass[12pt,a4paper]{ctexart}

% ===== 页面设置 =====
\usepackage{geometry}
\geometry{top=2.8cm, bottom=2.5cm, left=3cm, right=2.6cm}
\setlength{\headheight}{14.5pt}

% ===== 数学公式 =====
\usepackage{amsmath}
\usepackage{amssymb}

% ===== 图片 =====
\usepackage{graphicx}
\usepackage{float}
\usepackage{subfigure}

% ===== 表格 =====
\usepackage{booktabs}
\usepackage{tabularx}
\usepackage{multirow}
\usepackage{makecell}
\usepackage{array}
\usepackage{threeparttable}

% ===== 参考文献 =====
\usepackage[nottoc]{tocbibind}
\usepackage[numbers,sort&compress,super,square]{natbib}

% ===== 超链接（无颜色可跳转） =====
\usepackage{hyperref}
\hypersetup{
    bookmarks=true,
    bookmarksnumbered=true,
    hidelinks
}
\usepackage{bookmark}

% ===== 标题格式 =====
\usepackage{titlesec}
\usepackage{titletoc}
\titleformat{\section}{\bfseries\heiti\fontsize{14pt}{18pt}\selectfont}{\thesection}{1em}{}
\titleformat{\subsection}{\bfseries\kaishu\fontsize{13pt}{16pt}\selectfont}{\thesubsection}{1em}{}
\titleformat{\subsubsection}{\bfseries\fontsize{12pt}{15pt}\selectfont}{\thesubsubsection}{1em}{}

% ===== 目录 =====
\makeatletter
\def\@dotsep{3.5}
\makeatother

% ===== 排版间距 =====
\setlength{\parindent}{2em}
\setlength{\parskip}{0pt}
\linespread{1.67}

% ===== 图表编号 =====
\usepackage{chngcntr}
\counterwithout{figure}{section}
\counterwithout{table}{section}

% ===== 自定义命令 =====
\def\mytitle#1{{\fontsize{18pt}{22pt}\bfseries\selectfont #1}}
\def\mybody#1{{\fontsize{12pt}{15pt}\selectfont #1}}

% ===== 摘要格式 =====
\renewenvironment{ abstract }{%
  \thispagestyle{empty}
  \vspace*{0.5cm}
  \begin{center}
    \vspace*{1.5cm}
    \mytitle{查新项目名称}\\[1em]
    {\heiti\zihao{3} 查新报告}\\[2em]
    {\kaishu\zihao{-4} 查新机构}\\[0.5em]
    {\kaishu\zihao{-4} 日期}
  \end{center}
  \vspace*{1.5cm}
  \begin{center}
    {\heiti\zihao{3}\textbf{摘\quad 要}}
  \end{center}
  \begin{center}
  \parbox{0.9\textwidth}{
  \kaishu\zihao{-4}
  摘要正文内容...
  \\[2ex]
  \noindent\textbf{关键词：}关键词1；关键词2；关键词3
  }
  \end{center}
  \vspace*{\fill}
  \newpage
}{}

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 查新报告}
\fancyfoot[C]{\thepage}

% ===== 参考文献标题 =====
renewcommand{\bibname}{参考文献}

\begin{document}

% ===== 封面 + 摘要（同一页）=====
\begin{ abstract }

% ===== 目录 =====
\tableofcontents
\newpage

\section{查新目的}
\subsection{查新项目名称}
\subsection{查新目的}
\subsection{查新点}
\subsection{查新要求}

\section{检索范围与策略}
\subsection{检索数据库}
\subsection{检索词}
\subsection{检索式}

\section{检索结果}
\subsection{文献检索}
\subsection{密切相关文献分析}

\section{查新结论}

\section{附件}

% ===== 参考文献 =====
\nocite{*}
\bibliographystyle{plainnat}
\bibliography{references}
\newpage

\vspace*{2cm}
\begin{flushright}
\textbf{编写人}: 作者 \\
\textbf{审核人}: 审核人 \\
\textbf{日期}: 2026年4月
\end{flushright}

\end{document}
```

### 格式优化 Agent 工作流程

```
接收 Markdown 内容 + 图表代码 + 目标格式
    ↓
选择/加载格式模板
    ↓
执行 pylatex 或 python-docx 脚本生成文档
    ↓
自动执行 XeLaTeX 编译（xelatex → bibtex → xelatex → xelatex）
    ↓
验证输出文件完整性
    ↓
输出：文件路径 + 编译/使用说明
```

**格式优化 Agent 完整指令（LaTeX）：**
```
你是一名 LaTeX 格式工程师。请将 Markdown 文档转换为符合规范的 LaTeX 文档。

## 输入信息

文档类型：[类型]
格式模板：[模板名称]

## 写作内容

[粘贴内容优化Agent输出的完整Markdown]

## 图表代码

[粘贴图表Agent输出的LaTeX代码]

## 输出要求

1. 使用 python_execute 工具执行 Python 脚本
2. 使用 pylatex 库生成 LaTeX 文档
3. 输出完整的 .tex 文件内容
4. 输出编译说明（需要安装的依赖包、编译命令）

## pylatex 使用参考

```python
# /// script
# dependencies = ['pylatex']
# ///

from pylatex import Document, Section, Subsection, Command
from pylatex.math import Math, Alignat
from pylatex.packages import Booktabs, Hyperref
from pylatex import NoEscape

def create_latex_doc(content_dict):
    doc = Document('article')
    doc.preamble.append(Command('usepackage', 'ctex'))
    doc.preamble.append(Command('usepackage', 'graphicx'))
    doc.preamble.append(Command('usepackage', 'booktabs'))

    # 添加标题
    doc.append(NoEscape(r'\title{' + content_dict['title'] + r'}'))
    doc.append(NoEscape(r'\author{' + content_dict['author'] + r'}'))
    doc.append(NoEscape(r'\date{\today}'))
    doc.append(NoEscape(r'\maketitle'))

    # 添加摘要
    with doc.create(Section('摘要')):
        doc.append(content_dict['abstract'])
        doc.append(NoEscape(r'\\textbf{关键词}：' + content_dict['keywords']))

    # 添加正文章节
    for section in content_dict['sections']:
        with doc.create(Section(section['title'])):
            doc.append(section['content'])

    doc.generate_tex('output')
    return 'output.tex'
```

## 注意事项

1. 确保中文字体正确配置
2. 图表代码需要正确插入到对应章节
3. 生成文件保存到用户工作目录
## 自动编译 LaTeX（F015-增强）

生成 .tex 文件后，使用 Bash 工具自动执行 XeLaTeX 编译，生成 PDF 文件。

### 编译步骤

```bash
cd <输出目录>
xelatex document.tex
xelatex document.tex
```

### 成功标志

- 生成同名 .pdf 文件
- 无 fatal error（warnings 正常）

### 失败处理

- 如果 xelatex 命令不存在，尝试 `latexmk -xelatex document.tex`
- 如果仍然失败，保留 .tex 文件并说明"请手动编译"
- 在输出中明确告知 PDF 是否生成成功

```

**格式优化 Agent 完整指令（Word）：**
```
你是一名 Word 格式工程师。请将 Markdown 文档转换为符合规范的 Word 文档。

## 输入信息

文档类型：[类型]
格式规范：[规范详情]

## 写作内容

[粘贴内容优化Agent输出的完整Markdown]

## 图表代码

[粘贴图表Agent输出的Word Python代码]

## 输出要求

1. 使用 python_execute 工具执行 Python 脚本
2. 使用 python-docx 库生成 Word 文档
3. 输出完整的 .docx 文件
4. 输出使用说明

## python-docx 使用参考

```python
# /// script
# dependencies = ['python-docx']
# ///

from docx import Document
from docx.shared import Pt, Inches, Cm, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.style import WD_STYLE_TYPE

def set_chinese_format(run, font_name='宋体', size=12, bold=False):
    """设置中文字体格式"""
    run.font.name = font_name
    run.font.size = Pt(size)
    run.font.bold = bold
    run._element.rPr.rFonts.set(qn('w:eastAsia'), font_name)

def apply_heading_style(doc, level, text):
    """应用标题样式"""
    heading = doc.add_heading(text, level=level)
    for run in heading.runs:
        if level == 1:
            set_chinese_format(run, '黑体', 16, bold=True)
        elif level == 2:
            set_chinese_format(run, '黑体', 14, bold=True)
        else:
            set_chinese_format(run, '宋体', 12, bold=True)
    heading.alignment = WD_ALIGN_PARAGRAPH.CENTER if level == 1 else WD_ALIGN_PARAGRAPH.LEFT
    return heading

def create_word_doc(content_dict):
    doc = Document()

    # 设置默认段落样式
    style = doc.styles['Normal']
    font = style.font
    font.name = 'Times New Roman'
    font.size = Pt(12)

    # 添加标题
    title = doc.add_heading(content_dict['title'], 0)
    title.alignment = WD_ALIGN_PARAGRAPH.CENTER

    # 添加作者信息
    author_para = doc.add_paragraph(content_dict['author'])
    author_para.alignment = WD_ALIGN_PARAGRAPH.CENTER

    # 添加正文
    for section in content_dict['sections']:
        apply_heading_style(doc, 1, section['title'])
        doc.add_paragraph(section['content'])

    doc.save('output.docx')
    return 'output.docx'
```

## 注意事项

1. 严格遵循格式规范表中的字体、字号、行距要求
2. 表格使用三线表样式
3. 图片插入后添加题注
4. 生成文件保存到用户工作目录
## 自动编译 LaTeX（F015-增强）

生成 .tex 文件后，使用 Bash 工具自动执行 XeLaTeX 编译，生成 PDF 文件。

### 编译步骤

```bash
cd <输出目录>
xelatex document.tex
xelatex document.tex
```

### 成功标志

- 生成同名 .pdf 文件
- 无 fatal error（warnings 正常）

### 失败处理

- 如果 xelatex 命令不存在，尝试 `latexmk -xelatex document.tex`
- 如果仍然失败，保留 .tex 文件并说明"请手动编译"
- 在输出中明确告知 PDF 是否生成成功

```

kind: skill
---

## 第七阶段：审查 Agent

**角色定义：** 资深审查员，负责对文档进行三重审查（逻辑/格式/内容）。

**输入契约：**
- 格式化后的文档（.tex / .docx）
- 原始需求文档
- 格式规范

**输出契约：**
```json
{
  "review_result": "通过 / 需要修改",
  "issues": [
    {
      "type": "逻辑问题 / 格式问题 / 内容问题",
      "severity": "严重 / 一般 / 轻微",
      "location": "章节/段落位置",
      "description": "问题描述",
      "suggestion": "修改建议"
    }
  ],
  "overall_assessment": "整体评价"
}
```

### F011: 逻辑审查要点

- 章节结构是否完整合理
- 论证逻辑是否严密
- 段落之间过渡是否自然
- 是否存在自相矛盾的内容

### F012: 格式审查要点

- LaTeX：编译是否成功、警告/错误数量
- Word：样式是否正确应用、页眉页脚、目录
- 图表编号是否连续、题注位置是否正确
- 参考文献格式是否统一

### F013: 内容审查要点

- 核心观点是否突出
- 数据/案例是否准确
- 语言表达是否清晰专业
- 是否有错别字/病句

**审查 Agent 完整指令：**
```
你是一名资深学术/商业文档审查员。请对生成的文档进行全面审查。

## 审查文档

[文档内容或文件路径]

## 原始需求

[需求文档内容]

## 格式规范

[格式规范详情]

## 审查维度

### 1. 逻辑审查（F011）
- 章节结构是否完整：摘要→关键词→引言→方法→实验→结论→参考文献
- 论证逻辑是否严密
- 段落之间过渡是否自然
- 前后表述是否一致

### 2. 格式审查（F012）
- LaTeX：编译是否成功、是否存在警告或错误
- Word：样式应用是否正确、页眉页脚是否规范
- 图表编号是否连续
- 参考文献格式是否统一（GB/T 7714 / IEEE）

### 3. 内容审查（F013）
- 核心观点是否明确突出
- 数据/案例是否准确可靠
- 语言表达是否清晰专业
- 是否有错别字、语病

## 输出格式

以 JSON 格式输出审查结果：

```json
{
  "review_result": "通过 / 需要修改",
  "issues": [
    {
      "type": "逻辑问题 / 格式问题 / 内容问题",
      "severity": "严重 / 一般 / 轻微",
      "location": "具体位置",
      "description": "问题描述",
      "suggestion": "修改建议"
    }
  ],
  "overall_assessment": "整体评价及改进建议"
}
```

如果审查通过，在 overall_assessment 中说明"文档质量达标，可直接使用"。
如果需要修改，列出所有问题及修改建议。
```

kind: skill
---

## F014: 多格式输出

### 输出格式组合

| 场景 | 主要输出 | 附带输出 |
|------|---------|---------|
| 仅 LaTeX | .tex 文件 + .pdf | 编译说明 |
| 仅 Word | .docx 文件 | 使用说明 |
| 双格式 | .tex + .docx | 各自的编译/使用说明 |
| 带图表 | 主文件 + 图片文件夹 | 图片目录说明 |

### 编译说明模板（F015）

**LaTeX 编译说明：**
````markdown
## LaTeX 文档编译说明

### 环境要求
- **TeX 发行版**：TeX Live 2020+ / MacTeX / MiKTeX
- **编辑器推荐**：VS Code + LaTeX Workshop / TeXstudio / Overleaf（在线）

### 依赖宏包
本文档使用以下宏包（TeX Live/MacTeX 默认安装，如缺失请使用 tlmgr 安装）：
- ctex（中文支持）
- geometry（页面设置）
- amsmath, amssymb, amsthm（数学公式）
- graphicx（图片插入）
- booktabs（三线表）
- hyperref（超链接）
- pgfplots（数据图表）
- tikz（流程图）
- setspace（行距控制）

### 编译命令（默认 XeLaTeX）

**默认使用 XeLaTeX 引擎**（支持完整中文，无需额外配置）：

```bash
cd <文档目录>
xelatex document.tex    # 第一次编译（生成 .aux 包含 bibdata）
bibtex document        # 生成参考文献（使用 plainnat.bst）
xelatex document.tex   # 第二次（写入参考文献）
xelatex document.tex   # 第三次（交叉引用稳定）
```

**含参考文献的完整编译流程：**
```bash
xelatex document.tex
bibtex document
xelatex document.tex
xelatex document.tex
```

**常见问题**

1. **中文无法编译**：确保使用 `xelatex` 或 `lualatex` 引擎，不要使用 `pdflatex`
2. **图表不显示**：当前文档使用 TikZ/pgfplots 绘制矢量图表，无需外部图片文件
3. **交叉引用警告**：运行两次 xelatex 后自动消除
4. **参考文献格式**：使用 `\bibliographystyle{plainnat}` + `\bibliography{references}` 需先运行 bibtex
````

**Word 使用说明：**
````markdown
## Word 文档使用说明
### 兼容性
- 建议使用 Microsoft Word 2016+ 或 WPS Office
- 开启"修订模式"以便跟踪修改

### 目录生成
1. 将光标放在需要插入目录的位置
2. 引用 → 目录 → 选择样式
3. 更新目录：右键目录 → 更新域

### 图表题注
- 插入图片后，右键 → 插入题注
- 题注格式：图 1. 描述 / 表 1. 描述
````

kind: skill
---

## 错误处理与降级策略

### 错误处理矩阵

| 错误类型 | 错误表现 | 处理策略 |
|---------|---------|---------|
| 模板解析失败 | 无法提取格式规范 | 使用内置默认模板，询问用户格式偏好 |
| 规划 Agent 失败 | 超时或输出格式错误 | 降级为单 Agent 模式，直接根据需求生成大纲 |
| 写作 Agent 失败 | 内容空洞或逻辑混乱 | 输出框架级内容，用户填充具体内容 |
| 图表 Agent 失败 | 代码无法执行 | 跳过图表，标记占位符位置，提示用户手动插入 |
| 格式优化失败 | pylatex/python-docx 报错 | 回退到 Markdown 输出，提供 Pandoc 转换命令 |
| 审查失败 | 审查超时 | 跳过审查，用户自行检查质量 |

### 降级执行流程

```
多 Agent 流水线执行中...
    ↓
某个 Agent 失败
    ↓
判断失败类型（可重试 / 不可重试）
    ↓
可重试：等待30秒后重试，最多3次
    ↓
不可重试或重试失败
    ↓
触发降级策略
    ↓
输出：降级模式说明 + 部分成果 + 待完成任务清单
```

### 降级模式说明

**单 Agent 模式：**
```
当前处于降级模式，仅使用写作 Agent。

功能限制：
- 跳过图表生成步骤（需用户手动插入）
- 输出 Markdown 格式而非 LaTeX/Word
- 跳过自动审查（需用户自行检查）

可用命令：
- /retry：重新尝试完整流水线
- /format：重新执行格式优化步骤
- /review：单独执行审查步骤
```

kind: skill
---

## 交互流程指南

### 完整交互流程

```
用户发起请求
    ↓
Skill 判断触发
    ↓
需求确认（信息不足时）
    ↓
执行五 Agent 流水线
    ↓
各阶段进度汇报
    ↓
最终输出 + 编译/使用说明
    ↓
用户反馈 → 迭代修改（可选）
```

### 对话交互模板

**初次触发：**
```
我已接收到您的文档写作需求。

📄 文档类型：[类型]
📦 目标格式：[格式]
📝 预估篇幅：[字数]

如需调整或有参考模板，请现在说明。
确认后我将启动五步流水线开始写作。
```

**各阶段完成：**
```
✅ 规划阶段完成
   已生成 [N] 章节大纲，预估总字数 [X]

🔄 写作阶段进行中
   正在撰写：[当前章节]

✅ 写作阶段完成
   已生成完整正文，共 [N] 字

🔄 内容优化阶段进行中
   正在调用 Doubao-seed-2.0-pro 润色文本...

✅ 内容优化阶段完成
   文本已完成语言润色与逻辑强化

🔄 图表阶段进行中
   正在生成 [N] 个图表...

🔄 格式优化阶段进行中
   正在转换为 [格式]...
```

**最终输出：**
```
✅ 文档生成完成！

📁 输出文件：[文件路径]
📊 图表数量：[N] 个
📏 总字数：[X] 字

编译/使用说明已随文件附上。
您可以通过 /review 命令进行质量审查，
或通过 /modify 命令进行局部修改。
```

kind: skill
---

## Python 依赖说明

### 核心依赖

```python
# /// script
# dependencies = [
#     'pylatex>=1.4.6',
#     'python-docx>=1.1.0',
#     'jinja2>=3.1.2',
#     'matplotlib>=3.7.0',
#     'pillow>=10.0.0',
#     'lxml>=4.9.0'
# ]
# ///
```

### pylatex 使用场景

- 生成 LaTeX 文档（Article/Report/Book 文档类）
- 添加章节、公式、表格、图表
- 自动处理包依赖

### python-docx 使用场景

- 生成 Word 文档
- 应用样式和格式
- 插入图表和表格

### jinja2 使用场景

- 模板渲染（用户自定义模板）
- 批量生成相似文档

### matplotlib 使用场景

- 生成图表数据图片
- 为 pgfplots 提供数据支持

kind: skill
---

## 附录：完整格式规范速查表

### 中文学术论文

| 项目 | LaTeX | Word |
|------|-------|------|
| 文档类 | `[12pt,a4paper]{ctexart}` | - |
| 纸张 | a4paper | A4 |
| 页边距 | 上2.8cm，下2.5cm，左3cm，右2.6cm | 上2.8cm，下2.5cm，左3cm，右2.6cm |
| 正文字号 | 12pt | 12pt |
| 行距 | 1.67倍 | 1.67倍 |
| 中文字体 | 黑体/楷体/宋体 | 宋体/黑体 |
| 英文字体 | Times New Roman | Times New Roman |
| section | `\fontsize{14pt}{18pt}` 黑体加粗 | 标题1样式 |
| subsection | `\fontsize{13pt}{16pt}` 楷体加粗 | 标题2样式 |
| 首行缩进 | 2字符 | 2字符 |
| 超链接 | hidelinks | - |
| 图表环境 | figure[htbp]+TikZ/pgfplots | 插入图片+题注 |
| 表格环境 | table[htbp]+booktabs 三线表 | 表格+三线表样式 |
| 参考文献 | `plainnat`+上标编号 | - |
| 摘要格式 | `\renewenvironment{abstract}` 居中加粗 | - |
| 封面+摘要 | 同页 | 同页 |

### 项目申报书

| 项目 | 要求 |
|------|------|
| 纸张 | A4 |
| 页边距 | 上2.8cm，下2.5cm，左3cm，右2.6cm |
| 标题 | 黑体，18pt，居中 |
| 正文 | 12pt，1.67倍行距 |
| section | 黑体，14pt，加粗 |
| subsection | 楷体，13pt，加粗 |
| 首行缩进 | 2字符 |
| 超链接 | 无颜色无框，可跳转 |
| 参考文献 | `plainnat`，上标编号 |
| 封面+摘要 | 同页 |
| 参考文献 | 单独一页 |

### 查新报告

| 项目 | 要求 |
|------|------|
| 纸张 | A4 |
| 页边距 | 上2.8cm，下2.5cm，左3cm，右2.6cm |
| 标题 | 黑体，18pt，居中 |
| 正文 | 12pt，1.67倍行距 |
| 首行缩进 | 2字符 |
| 结构 | 封面→查新内容→检索策略→检索结果→结论→附件 |
| 查新点 | 编号列出，独立段落 |
| 密切相关文献 | 详细分析对比表 |
| 附件 | 检索式、相关文献全文 |
| 参考文献 | `plainnat`，上标编号 |

## 📌 附录：LaTeX 格式规范（必须遵守）

### 章节命令使用规则

**正确用法**（自动编号，不写"一、二、三"）：
```latex
\section{项目背景}          % 自动编号为"1 项目背景"
\subsection{研究现状}       % 自动编号为"1.1 研究现状"
```

**错误用法**（禁止手动编号）：
```latex
\section{一、项目背景}    % ❌ 错误：不要在标题里写数字编号
\section{项目背景与意义}   % ✅ 正确：直接写标题
```

### 目录生成

在 \begin{document} 后添加：
```latex
\tableofcontents  % 生成目录
```

### 摘要格式

```latex
\section*{摘要}              % 星号命令：不编号
\addcontentsline{toc}{section}{摘要}  % 手动加入目录

正文内容...

\noindent\textbf{关键词}：关键词1；关键词2
```

### 编译命令（默认 XeLaTeX）

```bash
xelatex document.tex  % 第一次
xelatex document.tex  % 第二次
```


## 📌 附录：LaTeX 格式规范（必须遵守）

### 页面格式

```latex
% hyperref - 无框无颜色，仅可跳转
\hypersetup{
    colorlinks=false,
    pdfborder={0 0 0}
}

% 页码居中底部
\fancypagestyle{plain}{
    \fancyhf{}\fancyfoot[C]{\thepage}\fancyhead[C]{}
    \renewcommand{\headrulewidth}{0pt}
}
\pagestyle{plain}
```

### 标题页格式

```latex
\begin{center}
\vspace*{2cm}
{\heiti\zihao{1}\textbf{文档标题}}\\[1em]
{\heiti\zihao{3} 副标题}\\[3em]
{\kaishu\zihao{-4} 日期}
\end{center}

% 摘要居中区域
\vspace*{\fill}
\begin{center}
{\heiti\zihao{3}\textbf{摘\quad 要}}\\[1ex]
\parbox{0.9\textwidth}{
\kaishu\zihao{-4}
摘要正文...
\\[2ex]
\noindent\textbf{关键词}：关键词1；关键词2
}
\end{center}
\vspace*{\fill}
```

### 图表规范

- **流程图**：使用 TikZ，节点水平排列居中
- **柱状图**：使用 pgfplots，symbolic x coords
- **表格**：使用 booktabs 三线表（\toprule/\midrule/\bottomrule）

### 字号层级规范

```latex
% 统一字号 + 层级区分
\ctexset{
    % 统一正文字号（小四）
    section={
        format=\heiti\zihao{-4}\bfseries,
        beforeskip=2ex,
        afterskip=1ex
    },
    % subsection 加大（四号）
    subsection={
        format=\heiti\zihao{4}\bfseries,
        beforeskip=1.5ex,
        afterskip=0.8ex
    },
    % subsubsection 再加大（三号）
    subsubsection={
        format=\heiti\zihao{3}\bfseries,
        beforeskip=1ex,
        afterskip=0.5ex
    }
}

% 行间距（建议1.8）
\setstretch{1.8}
```

% 参考文献字号（不要用 scriptsize）
\begin{thebibliography}{99}
\small  % 使用 \small 或 \footnotesize，不要用 \scriptszie
\item ...
\end{thebibliography}

% 关键词加粗
\textbf{卷积神经网络}、\textbf{深度学习}、\textbf{医学影像}
```

### 编译命令

```bash
xelatex document.tex
xelatex document.tex
```


## 📌 附录：LaTeX 完整格式配置（来自参考模板）

### 完整模板配置

```latex
\documentclass[12pt,a4paper]{ctexart}

% ===== 页面设置 =====
\usepackage{geometry}
\geometry{top=2.8cm, bottom=2.5cm, left=3cm, right=2.6cm}
\setlength{\headheight}{14.5pt}

% ===== 数学公式 =====
\usepackage{amsmath}
\usepackage{amssymb}

% ===== 图片 =====
\usepackage{graphicx}
\usepackage{float}
\usepackage{subfigure}

% ===== 表格 =====
\usepackage{booktabs}
\usepackage{tabularx}
\usepackage{multirow}
\usepackage{makecell}
\usepackage{array}
\usepackage{threeparttable}

% ===== 参考文献 =====
\usepackage[nottoc]{tocbibind}
\usepackage[numbers,sort&compress,super,square]{natbib}

% ===== 超链接（无颜色可跳转） =====
\usepackage{hyperref}
\hypersetup{
    bookmarks=true,
    bookmarksnumbered=true,
    hidelinks
}
\usepackage{bookmark}

% ===== 标题格式 =====
\usepackage{titlesec}
\usepackage{titletoc}
\titleformat{\section}{\bfseries\heiti\fontsize{14pt}{18pt}\selectfont}{\thesection}{1em}{}
\titleformat{\subsection}{\bfseries\kaishu\fontsize{13pt}{16pt}\selectfont}{\thesubsection}{1em}{}
\titleformat{\subsubsection}{\bfseries\fontsize{12pt}{15pt}\selectfont}{\thesubsubsection}{1em}{}

% ===== 目录 =====
\makeatletter
\def\@dotsep{3.5}
\makeatother

% ===== 排版间距 =====
\setlength{\parindent}{2em}
\setlength{\parskip}{0pt}
\linespread{1.67}

% ===== 图表编号 =====
\usepackage{chngcntr}
\counterwithout{figure}{section}
\counterwithout{table}{section}

% ===== 自定义命令 =====
\def\mytitle#1{{\fontsize{18pt}{22pt}\bfseries\selectfont #1}}
\def\mybody#1{{\fontsize{12pt}{15pt}\selectfont #1}}

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 页眉内容}
\fancyfoot[C]{\thepage}
```

### 格式要点总结

| 项目 | 配置值 |
|------|--------|
| 文档类 | `[12pt,a4paper]{ctexart}` |
| 页边距 | 上2.8cm，下2.5cm，左3cm，右2.6cm |
| **section** | `\fontsize{14pt}{18pt}` 黑体加粗 |
| **subsection** | `\fontsize{13pt}{16pt}` 楷体加粗 |
| **subsubsection** | `\fontsize{12pt}{15pt}` 加粗 |
| 正文 | `\fontsize{12pt}{15pt}` |
| 行距 | `\linespread{1.67}` |
| 首行缩进 | `\setlength{\parindent}{2em}` |
| **超链接** | `hidelinks`（无颜色无框） |
| 页码 | 居中底部 |
| 页眉 | 居中显示内容 |
| 图表编号 | 独立编号（不随章节） |
| 参考文献 | `natbib`，上标编号 |

### 参考文献格式

```latex
\bibliographystyle{plainnat}
\bibliography{references}
```

### 目录生成

```latex
\tableofcontents  % 在 \begin{document} 后
```

**目录控制策略（确保目录控制在一页内）：**

当目录条目在 1页～1.5页 之间时，通过以下方式压缩至一页（优先级从高到低）：

1. **删除 subsection 层级的独立标题**：将 `section` 下的短内容 subsection 直接合并到 parent section 正文中，不单独列条目
2. **减少 subsection 数量**：每个 section 保留不超过2个 subsection 列出
3. **删除 subsubsection 层级**：只保留 section 和 subsection 两级目录
4. **收紧行距**：`\linespread{1.3}` 临时覆盖正文行距
5. **减少 dotsep**：`\def\@dotsep{2}` 减少目录点线间距

当目录条目略超一页时，优先使用第 1 步和第 2 步压缩内容。

kind: skill
---

## 第八阶段：质量自检循环（结果交付前必须执行）

> **目的**：在最终交付结果之前，自动检测并修复 LaTeX 语法错误、宏包缺失、格式不符合等问题，形成自检→修复→再检的闭环。
>
> **触发时机**：格式优化 Agent 生成 .tex 文件后，审查 Agent 开始前。

### 自检循环流程

```
生成 .tex 文件
    ↓
【自检】执行 XeLaTeX 编译，捕获错误输出
    ↓
有错误？
  ├─ YES → 【诊断】解析错误类型（语法/宏包/引用/格式）
  │          ↓
  │    【修复】自动修复问题（最多3次迭代）
  │          ↓
  │    【再检】重新编译验证
  │          ↓
  │    仍失败？→ 记录问题，提示用户手动处理
  │          ↓
  └─ NO → 【输出】验证通过，交付 PDF + .tex
```

### 自检检查项（F016）

| 检查项 | 检测方式 | 失败表现 | 修复策略 |
|--------|---------|---------|---------|
| LaTeX 语法错误 | `xelatex` 编译输出 | `! Error:` / `! LaTeX Error:` | 定位行号，修正语法 |
| 宏包缺失 | 编译输出 | `! LaTeX Error: File 'xxx.sty' not found` | 添加 `\usepackage{xxx}` |
| 参考文献缺失 | `bibtex` 输出 | `Warning: empty bibliography` | 检查 `\nocite{*}` 和 bib 文件 |
| 引用 key 不存在 | `bibtex` 输出 | `Database file doesn't contain key` | 修正或补全 key |
| 图表环境错误 | 编译输出 | `! Missing $ inserted` / `Runaway definition` | 修正 TikZ/pgfplots 代码 |
| 字体不可用 | 编译输出 | `Font ... not loadable` | 替换为可用中文字体 |
| 目录超长 | 编译后 PDF | 目录超过1.5页 | 执行目录压缩策略（F017） |
| 摘要位置错误 | PDF 检查 | 摘要不在封面页 | 重排 abstract 环境 |
| 编译超时/挂起 | 进程监控 | 编译超过60秒无响应 | `pkill -f xelatex`，跳过编译 |

### 错误诊断与自动修复策略（F016-详细）

#### 错误1：宏包缺失
```
检测：xelatex 输出包含 "File 'xxx.sty' not found"
修复：在导言区添加 \usepackage{xxx}
      常用缺失宏包自动补全列表：
      - geometry → \usepackage{geometry}
      - hyperref → \usepackage{hyperref}
      - booktabs → \usepackage{booktabs}
      - natbib   → \usepackage{natbib}
      - fancyhdr → \usepackage{fancyhdr}
      - titlesec → \usepackage{titlesec}
      - titletoc → \usepackage{titletoc}
      - pgfplots → \usepackage{pgfplots}
      - tikz     → \usetikzlibrary{shapes,arrows,positioning}
```

#### 错误2：语法错误
```
检测：xelatex 输出包含 "! <line>" 格式错误
修复：
  - ! Too many }'s → 查找多余的 }，删除或补充配对的 {
  - ! Missing $ inserted → 数学符号在正文模式，补上 $ 或改用文字
  - ! Undefined control sequence → 命令拼写错误或宏包未加载
  - ! Runaway definition → 检查是否有未闭合的 { 或 verbatim 环境
```

#### 错误3：参考文献问题
```
检测：bibtex 输出包含 "Warning" 或 "doesn't contain key"
修复：
  - 缺少 \nocite{*} → 在 \bibliography{references} 前添加
  - key 不存在 → 在 references.bib 中补全缺失条目
  - BibTeX 未运行 → 补全编译流程：xelatex → bibtex → xelatex × 2
```

#### 错误4：图表代码错误
```
检测：xelatex 输出在 tikz/pgfplots 环境报错
修复：
  - 坐标格式错误 → 检查 addplot coordinates {} 语法
  - 节点定义缺失 → 补全 \node 命令
  - 库未加载 → 添加对应 \usetikzlibrary{}
```

#### 错误5：字体错误
```
检测：xelatex 输出包含 "Font ... not loadable"
修复：
  - XeLaTeX 下中文字体错误 → 确保使用 ctexart/XeLaTeX
  - 英文字体错误 → 使用 Times New Roman / Arial 等系统可用字体
```

### 目录压缩策略（F017）

当目录条目在 1页～1.5页 之间时，执行以下压缩（优先级从高到低）：

1. **删除短 subsection 独立标题**：将内容较少的 subsection 直接合并到 parent section
2. **减少 subsection 数量**：每个 section 最多保留2个 subsection 条目
3. **删除 subsubsection 层级**：目录只保留 section 和 subsection 两级
4. **收紧行距**：`\setlength{\baselineskip}{1.2em}` 临时覆盖
5. **减少 dotsep**：`\def\@dotsep{2}` 减少目录点线间距

### 自检 Agent 完整指令

```
你是一名 LaTeX 质量工程师。请对生成的 .tex 文件执行自检循环，在交付前确保文档可正常编译。

## 待检测文件

[文件路径或完整 .tex 内容]

## 自检流程

### Step 1：编译检测
使用 Bash 工具执行以下编译流程：

```bash
cd <文件所在目录>
# 清理旧编译产物
rm -f *.aux *.bbl *.blg *.log *.out

# 第一次编译
xelatex document.tex 2>&1 | tee compile_log1.txt

# BibTeX 处理
bibtex document 2>&1 | tee bibtex_log.txt

# 第二、三次编译
xelatex document.tex 2>&1 | tee compile_log2.txt
xelatex document.tex 2>&1 | tee compile_log3.txt
```

### Step 2：错误分析
分析 compile_log*.txt 中的错误：

1. **宏包缺失**：搜索 "not found" 或 "File 'xxx.sty'"
2. **语法错误**：搜索 "! " 开头行，提取行号和错误信息
3. **BibTeX 问题**：搜索 "Warning" 或 "doesn't contain key"
4. **字体问题**：搜索 "not loadable" 或 "Font"

### Step 3：自动修复
根据错误类型执行修复（最多3次迭代）：

```
迭代 1：修复语法错误和缺失宏包
  → 修改 .tex 文件，添加缺失宏包，修正明显语法错误
  → 重新编译验证

迭代 2：修复引用和图表问题
  → 检查并补全 references.bib 条目
  → 修正 TikZ/pgfplots 代码

迭代 3：修复格式问题
  → 目录压缩、字体替换等
```

### Step 4：输出结果

修复完成后，输出：

```json
{
  "status": "success / partial / failed",
  "iterations": 2,
  "issues_found": ["问题1", "问题2"],
  "issues_fixed": ["已修复问题列表"],
  "issues_remaining": ["未解决问题列表"],
  "pdf_generated": true,
  "final_message": "说明文字"
}
```

如果修复后仍无法编译，说明未解决的问题，告知用户需要手动处理的具体位置。

## 注意事项

1. **不要跳过编译**：每次修复后必须重新编译验证
2. **最多3次迭代**：3次后仍失败则停止循环，输出已知问题
3. **保留原始文件**：修复前先备份 .tex 文件
4. **杀死挂起进程**：如果编译超过60秒无响应，使用 `pkill -f xelatex` 终止
5. **完整编译流程**：即使有警告，只要无 fatal error 就认为通过
```
