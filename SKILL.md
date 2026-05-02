---
name: doc-writer
version: v3.3.1
description: |
  多智能体协同学术/商业文档写作技能，支持单轨/多轨并行/扩轨三种模式，可多支团队同时走不同方向并评选最优解。
  支持 LaTeX 和 Word 双格式输出。

  核心能力：
  - **联网写作（v1.1 新增）**：用户只给主题，AI自动联网搜索最新文献和数据，自动融入文档解决素材痛点
  - 七 Agent 流水线：规划 → 写作 → 内容优化（Doubao-seed-2.0-pro）→ 图表 → 格式优化 → 审查 + 迭代优化循环
  - 三种写作模式：单轨（默认）/ 多轨并行（多方案对比评选）/ 扩轨（多 Agent 并行）
  - 七 Agent 流水线含迭代优化循环：多轮迭代提升文档质量（自我审查 + 同伴互评 + Leader 评估）
  - 六种内置模板：中文学术论文、IEEE英文期刊、项目申报书、查新报告、商业计划书、技术方案文档
  - 图表扩展支持：流程图/折线图/柱状图/饼图/甘特图/Swimlane图、LaTeX TikZ/pgfplots、Word 原生图表
  - 表格样式增强：三线表/斑马纹表格/跨列跨行单元格
  - 多格式输出：LaTeX (pylatex)、Word (python-docx)、编译说明
  - 自动编译：XeLaTeX → BibTeX → XeLaTeX → XeLaTeX
kind: skill
model:
  default: MiniMax M2.7
  per_role:
    planner:
      - MiniMax M2.7
      - doubao-seed-2.0-pro
    writer: MiniMax M2.7
    optimizer:
      - MiniMax M2.7
      - doubao-seed-2.0-pro
    chart-designer: MiniMax M2.7
    formatter: MiniMax M2.7
    reviewer:
      - MiniMax M2.7
      - doubao-seed-2.0-pro
    knowledge-base: MiniMax M2.7
---

# 文档写作助手 (doc-writer)

多智能体协同流水线，通过六个专业 Agent 协作完成高质量学术/商业文档写作。

## 协作式写作模式（v3.2 新增）

> **核心价值**：多人协作写作时，AI 自动协调多个写作角色的输出，保证文档风格一致性和内容连贯性。

### 协作角色定义

| 角色 | 职责 | 输出 |
|------|------|------|
| **架构师** | 设计文档结构、确定章节逻辑 | 大纲 + 格式规范 |
| **撰写者** | 填充各章节内容 | 完整正文 |
| **审稿人** | 检查内容准确性、格式一致性 | 修订意见 |
| **协调者** | 管理写作进度、解决冲突 | 写作状态报告 |

### 协作流程

```python
class CollaborativeWritingMode:
    """协作式写作模式。

    支持多人（多角色）协作写作，
    AI 自动协调各角色输出，保证文档一致性。
    """

    def plan_document(self, topic: str, collaborators: List[str]) -> DocumentPlan:
        """
        规划文档：架构师设计大纲 → 协调者分配章节 → 撰写者分工
        """

    def merge_contributions(self, contributions: Dict[str, str]) -> str:
        """
        合并各角色贡献：自动处理风格差异、内容冲突
        """

    def review_unified(self, full_doc: str) -> ReviewReport:
        """
        统一审稿：审稿人检查合并后文档
        """
```

### 输出格式

```markdown
## 协作写作报告

### 参与角色
- 架构师：✅ 已完成大纲
- 撰写者A：✅ 已完成引言/方法
- 撰写者B：✅ 已完成实验/结果
- 审稿人：✅ 已审稿

### 内容贡献
| 章节 | 贡献者 | 字数 | 状态 |
|------|--------|------|------|
| 引言 | 撰写者A | 320字 | ✅ |
| 方法 | 撰写者A | 580字 | ✅ |
| 实验 | 撰写者B | 420字 | ✅ |
| 结论 | 审稿人整合 | 180字 | ✅ |

### 风格一致性评分
**得分**：92/100（9个术语统一、3处格式调整）
```

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

**与迭代优化的协同**：
- 迭代优化过程中发现的优质表达 → 存入 `references/terminology/custom.json`
- 迭代优化过程中发现的高质量文档片段 → 存入 `references/examples/{type}/`
- 迭代质量评分 < 60 的常见问题 → 整理后存入 `knowledge/COMMON_ISSUES.md`

### 模型选择说明（v0.9 新增）

| Agent | 任务特点 | 推荐模型 | 理由 |
|-------|---------|---------|------|
| planner | 文档结构规划、多方案设计 | `doubao-seed-2.0-pro` | 复杂推理、结构设计 |
| writer | 内容写作、章节填充 | `code` | 专注输出、性价比高 |
| optimizer | 内容优化、表达精炼 | `doubao-seed-2.0-pro` | 复杂推理、文字优化 |
| chart-designer | 图表代码生成 | `code` | 专注代码、效率高 |
| formatter | 格式调整、LaTeX/Word 排版 | `code` | 重复性操作 |
| reviewer | 质量审查、格式检查 | `doubao-seed-2.0-pro` | 逻辑分析、质量把关 |

### 知识库积累与迭代优化协同（v1.0 新增）

每次迭代循环结束后，Agent 自动积累本次经验：

| 迭代结果 | 知识库动作 |
|---------|---------|
| 高分文档片段（≥85分章节） | 存入 `references/examples/{type}/` |
| 低分原因（<70分问题） | 存入 `knowledge/COMMON_ISSUES.md` |
| 改进建议（高频出现） | 更新 `references/terminology/{domain}.json` 的 `academic_vocab` |
| 格式错误（重复出现） | 更新 `references/format_specs/{type}.md` 的"常见错误"部分 |

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

---

## 跨文档知识图谱（v1.7 新增）

> **核心价值**：在写作多份相关文档时，自动维护文档间的知识一致性，确保同一概念在不同文档中表述统一。

### 核心问题

项目中有大量文档（PRD/架构/接口/用户手册），但：
- 同一概念在不同文档中名称不一致（"用户" vs "账户" vs "账号"）
- 同一数据在不同文档中值不匹配（端口号/版本号/配置参数）
- 文档更新后，其他关联文档未同步

### 知识图谱引擎

```python
class DocumentKnowledgeGraph:
    """跨文档知识图谱。建立文档间的概念关系，维护一致性。"""

    def __init__(self):
        self.entities = {}      # {entity_id: {name, type, values, docs}}
        self.relations = []    # [(entity_a, relation, entity_b)]

    def extract_from_doc(self, doc_path: str) -> List[Entity]:
        """
        从文档中提取实体。

        实体类型：
        - 术语（Term）："JWT" / "OAuth2" / "REST"
        - 配置（Config）：端口号/超时时间/版本号
        - 角色（Role）："管理员" / "普通用户" / "访客"
        - 接口（API）："/api/v1/users" / "GET /login"
        - 数据模型（Model）："User" / "Order" / "Product"
        """

    def build_graph(self, doc_paths: List[str]) -> Graph:
        """
        构建跨文档知识图谱。

        1. 遍历所有文档，提取实体
        2. 发现实体间关系（同义/引用/继承/依赖）
        3. 检测跨文档不一致
        4. 生成一致性报告
        """

    def check_consistency(self) -> List[Inconsistency]:
        """
        检测知识不一致。

        Returns:
            [{
                "type": "naming",  # 命名不一致
                "entities": ["用户", "账户", "账号"],
                "docs": ["prd.md", "api.md"],
                "suggestion": "统一使用'用户'"
            }]
        """
```

### 实体提取示例

```markdown
## 从文档中提取的实体

| 实体 | 类型 | 首次出现 | 出现文档 |
|------|------|---------|---------|
| JWT | 术语 | api.md | api.md, auth.md |
| access_token | API | auth.md | auth.md, frontend.md |
| refresh_token | API | auth.md | auth.md |
| 普通用户 | 角色 | rbac.md | rbac.md, prd.md |
| BASE_URL | 配置 | config.md | config.md, test.md |

### 发现的不一致

⚠️ **命名不一致**：同一概念在不同文档中名称不同
- "用户" (prd.md) = "账户" (api.md) = "账号" (user-guide.md)
- → 建议：统一使用「用户」

⚠️ **配置不一致**：同一配置在不同文档中值不同
- BASE_URL = "http://localhost:8080" (config.md)
- BASE_URL = "http://localhost:3000" (test.md)
- → 建议：统一为 8080
```

### 一致性检查工作流

```
用户开始写文档（或完成文档初稿）
    ↓
输入所有相关文档路径
    ↓
build_graph() 构建知识图谱
    ↓
check_consistency() 检测不一致
    ↓
生成一致性报告
    ↓
用户确认后，自动更新各文档中的不一致表述
```

### 与其他文档的协同

```
doc-writer
├── 本次写作 ← 当前正在写的文档
├── 知识图谱 ← 已有的其他文档
└── 一致性检查 ← 跨文档术语/配置统一

当用户说"写接口文档"时：
1. 加载知识图谱（prd.md / 架构文档 / 其他已有文档）
2. 提取关键术语和配置
3. 在写作时自动引用一致的术语
4. 完成后更新知识图谱
```

---

## 多格式联动写作（v1.9 新增）

> **核心价值**：一个核心内容，多种格式输出，无需重复写。写一次，自动生成 Markdown/LaTeX/HTML/Word/PPT。

### 核心概念

Source Document（源文档）：
- 用结构化标记语言编写（类似 Markdown + YAML frontmatter）
- 包含元数据（作者/日期/版本/标签）
- 格式无关的核心内容

Output Formats（输出格式）：
- Markdown（文档/博客）
- LaTeX/PDF（论文/报告）
- HTML（网页/内网）
- Word（协作/审批）
- PPT（汇报/演示）

### 联动写作流程

```
用户编写源文档（结构化标记）
    ↓
doc-writer 解析源文档内容
    ↓
根据不同格式模板，生成对应输出
    ↓
格式验证 + 交叉引用检查
    ↓
输出：Markdown + LaTeX + HTML + Word + PPT
```

### 源文档结构

```markdown
---
title: 2026年Q1技术报告
author: 技术团队
date: 2026-03-31
format:
  markdown: true
  latex: true
  html: true
  word: true
  ppt: true
---

# 2026年Q1技术报告

## 执行摘要

本季度主要成果：

- 用户增长：120%（[详细数据见附录A](#附录A)）
- 系统稳定性：99.9%
- 技术债务偿还：30%

## 详细分析

### 1. 用户增长分析

[图表：用户增长曲线]

### 2. 技术改进

...
```

### 格式转换器

```python
class FormatTranslater:
    """多格式转换器。将源文档转换为各种格式。"""

    def to_markdown(self, source: SourceDoc) -> str:
        """转换为 Markdown（最接近源格式）。"""

    def to_latex(self, source: SourceDoc) -> str:
        """转换为 LaTeX 源码。包含：
        - 文档类选择（article/report/book）
        - 图表包（graphicx）
        - 数学包（amsmath）
        - 参考文献（biblatex）
        """

    def to_html(self, source: SourceDoc) -> str:
        """转换为 HTML。使用语义化标签 + 响应式 CSS。"""

    def to_word(self, source: SourceDoc) -> bytes:
        """转换为 Word 文档（docx）。用于协作审阅。"""

    def to_ppt(self, source: SourceDoc) -> bytes:
        """转换为 PPT。每章 → 一节幻灯片。"""

    def batch_convert(self, source: SourceDoc, formats: List[str]) -> Dict[str, Any]:
        """批量转换，一次生成所有指定格式。"""
```

### LaTeX 输出示例

```latex
\documentclass[12pt,a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage{graphicx}
\usepackage{amsmath}
\usepackage{biblatex}

\title{2026年Q1技术报告}
\author{技术团队}
\date{2026年3月}

\begin{document}
\maketitle

\section{执行摘要}
本季度主要成果：
\begin{itemize}
    \item 用户增长：120\%
    \item 系统稳定性：99.9\%
\end{itemize}

\section{详细分析}
\subsection{用户增长分析}
参见图表\ref{fig:user-growth}

\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.8\textwidth]{user-growth.pdf}
    \caption{用户增长曲线}
    \label{fig:user-growth}
\end{figure}
```

### 交叉引用同步

```python
def sync_cross_references(self, source: SourceDoc, outputs: Dict[str, Any]):
    """
    同步交叉引用到各格式。

    源文档中的 [图1] → LaTeX 中的 \ref{fig:1}
                         → HTML 中的 <a href="#fig1">
                         → Word 中的 书签引用
                         → PPT 中的 幻灯片链接
    """
```

### 一键生成所有格式

```python
def generate_all_formats(self, source_path: str) -> Dict[str, str]:
    """一键生成所有格式。"""

    source = self.load_source(source_path)

    return {
        "markdown": self.to_markdown(source),
        "latex": self.to_latex(source),
        "pdf": self.compile_latex(source),
        "html": self.to_html(source),
        "word": self.to_word(source),
        "ppt": self.to_ppt(source)
    }

# 使用示例
result = writer.generate_all_formats("quarterly-report.md")
# result 包含所有格式的输出
```

---

## 写作模式

> **启动流水线前请指定写作模式：**

## 迭代优化循环（v1.0 新增）

> 通过多轮迭代不断提升文档质量，每轮迭代包含自我审查、同行评审和 Leader 评估，形成持续改进的正反馈循环。

### 迭代流程

```
[初始版本] → [自我审查] → [同伴互评] → [Leader评估] → [修改优化]
                  ↑                                        |
                  └────────────────────────────────────────┘
                     （如评估分数 < 阈值，继续下一轮）
```

### 迭代参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `max_iterations` | 3 | 最大迭代轮数 |
| `quality_threshold` | 85 | 通过阈值（满分100），低于此值强制进入下一轮 |
| `min_improvement` | 5 | 连续两轮最低改善幅度，低于此值提前终止 |

### 迭代机制详解

#### 每轮迭代包含三个阶段：

**阶段 1：自我审查（Self-Review）**
- Writer 重新阅读自己的输出，对照 `references/format_specs/` 中的格式规范
- 标注自己发现的问题：逻辑漏洞、表达不清、数据不一致
- 输出：问题列表 + 自评分数（满分100）

**阶段 2：同伴互评（Peer Review）**
- 扩轨模式下，同章节的另一个 Writer 对输出进行交叉评审
- 多轨模式下，其他轨道团队中的 Writer 对当前输出进行评审
- 输出：评审意见 + 评分

**阶段 3：Leader 评估（Leader Assessment）**
- Leader 综合 Writer 自评 + 同伴互评，给出综合评分
- 若评分 < 质量阈值，进入下一轮迭代
- 若评分 ≥ 质量阈值，或达到最大迭代次数，退出迭代循环
- Leader 输出：综合评分 + 改进建议

### 迭代质量评估维度

| 维度 | 权重 | 评审依据 |
|------|------|---------|
| 内容完整性 | 25% | 格式规范库（format_specs）中的内容要素是否齐全 |
| 逻辑连贯性 | 25% | 文档拓扑依赖图，章节逻辑是否顺畅 |
| 表达准确性 | 20% | 术语库（terminology）规范程度，禁用词出现次数 |
| 数据可靠性 | 15% | 数据是否有来源标注，是否可核实 |
| 格式规范性 | 15% | 字体/图表/引用格式是否符合规范 |

### 提前终止条件

满足以下任一条件，迭代提前终止：

1. **质量达标**：综合评分 ≥ 质量阈值（默认85分）
2. **收敛停滞**：连续两轮改善幅度 < 最小改善值（默认5分）
3. **已达上限**：迭代次数达到最大轮数（默认3轮）
4. **用户干预**：用户主动要求停止迭代

### 迭代报告格式

```markdown
## 迭代报告 — 第 [N] 轮

### 自我审查
- 自评分数：[X]/100
- 发现问题：[问题列表]

### 同伴互评
- 评审人：[角色]
- 评审分数：[X]/100
- 评审意见：[...]

### Leader 综合评估
- 综合评分：[X]/100
- 维度得分：内容完整性 [X]/25 | 逻辑连贯性 [X]/25 | 表达准确性 [X]/20 | 数据可靠性 [X]/15 | 格式规范性 [X]/15
- 改进建议：[...]
- 决策：[继续迭代 / 通过，退出循环]
```

### 迭代循环中的 Agent 协同

| 阶段 | 执行者 | 输入 | 输出 |
|------|--------|------|------|
| 自我审查 | Writer | 当前版本 + format_specs | 问题列表 + 自评分 |
| 同伴互评 | 其他 Writer（扩轨/多轨模式） | 当前版本 + 术语库 | 评审意见 + 评分 |
| Leader评估 | Leader | 自评 + 同伴互评 + 格式规范 | 综合评分 + 决策 |
| 修改优化 | 原 Writer | Leader 改进建议 | 修订版本 |

### 与现有写作模式的关系

- **单轨模式** + 迭代优化：Writer 写完初稿 → 进入迭代循环 → 最终定稿
- **多轨并行模式** + 迭代优化：各轨分别迭代优化 → Leader 评选最优 → 最终定稿
- **扩轨模式** + 迭代优化：各章节并行迭代 → 合并整稿 → 最终定稿

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **单轨模式**（默认） | 单一团队，按流水线顺序完成一篇文档；含迭代优化循环（v1.0）| 明确需求、追求高质量定稿 |
| **多轨并行模式** | 2～4支团队**并行**独立撰写同一主题的不同方案/视角/侧重点，最终由 Leader 评选最优或合并多选 | 创新探索、多角度论证、需要对比方案、同一题目多版本 |
| **扩轨模式** | 单队但扩大规模（如2个写作 Agent 并行，负责不同章节组）；同伴互评在扩轨模式下天然支持 | 篇幅长、章节多、内容相对独立 |

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

## 联网写作模块（v1.1 新增）

> **核心价值**：用户只需要给主题，AI自动联网搜索最新文献和数据，自动融入文档。解决用户"有主题没素材"的痛点。

### 联网写作激活条件

联网写作在以下情况激活：
- 用户说"帮我搜一下最新的[领域]文献"
- 用户说"根据这个主题搜索相关研究"
- 文档规划阶段发现章节缺少数据支撑

### 搜索策略

| 文档类型 | 搜索目标 | 数据来源 |
|---------|---------|---------|
| 学术论文 | 最新研究成果、统计数据、实验数据 | arXiv、Google Scholar、Semantic Scholar |
| 项目申报书 | 市场规模、行业报告、政策文件 | 中国知网、万方数据、国家统计局 |
| 查新报告 | 专利检索、技术对比、文献分析 | 国家知识产权局、IEEE Xplore、ScienceDirect |
| 商业计划书 | 市场数据、行业趋势、竞品分析 | 艾瑞咨询、CBNData、QuestMobile |

### 搜索执行流程

```
规划 Agent 输出大纲
    ↓
检测到章节需要数据支撑（数据支撑标记：{{NEED_DATA:章节名}}）
    ↓
联网搜索 Agent 启动：
1. 分析章节主题，确定搜索关键词
2. 调用 Exa API 或 Google Search API 获取相关内容
3. 过滤低质量/过期内容（保留近3年内文献）
4. 提取关键数据：数值、引用、结论
5. 生成搜索报告（包含来源链接）
    ↓
写作 Agent 使用搜索结果填充章节内容
    ↓
标注数据来源（自动生成参考文献条目）
```

### 搜索报告格式

```markdown
## 搜索报告 — [章节名]

### 搜索关键词
- 主关键词：xxx
- 辅助关键词：xxx, xxx

### 搜索结果（Top 5）

| # | 标题 | 来源 | 年份 | 关键数据 |
|---|------|------|------|---------|
| 1 | [论文/报告标题] | [期刊/网站] | 2024 | [关键数据摘要] |
| 2 | ... | ... | ... | ... |

### 数据提取

**数值数据**：
- [关键数据1]：来源 [1]
- [关键数据2]：来源 [2]

**引用结论**：
- [结论1]：来源 [3]
- [结论2]：来源 [4]

### 自动生成的参考文献

[1] 作者. 标题. 期刊, 年份. DOI: xxx
[2] ...
```

### 注意事项

1. **数据可信度分级**：
   - A级（直接使用）：核心期刊、权威机构发布、政府统计数据
   - B级（谨慎使用）：普通期刊、行业报告、媒体数据
   - C级（仅供参考）：博客、论坛、非正式渠道

2. **数据标注要求**：
   - 所有数据必须标注来源
   - 无法核实的数据标注"[数据待核实]"
   - 禁止编造数据

3. **搜索频率控制**：
   - 每个章节最多搜索3次（避免无限循环）
   - 搜索结果超过10条时，只提取最相关的5条

### 联网写作示例

**用户输入**：
```
帮我写一篇关于"人工智能在医疗诊断中的应用"的学术论文，没有素材，你帮我搜。
```

**AI响应**：
```
已检测到联网写作需求，正在搜索最新文献...

## 搜索报告 — 研究背景

### 搜索关键词
- 主关键词：人工智能 医疗诊断
- 辅助关键词：AI diagnosis, medical AI, deep learning healthcare

### 搜索结果（已提取关键数据）

| 来源 | 年份 | 关键数据 |
|------|------|---------|
| Nature Medicine | 2024 | AI辅助诊断准确率达95%，超过人类医生平均水平 |
| The Lancet | 2023 | 深度学习在影像诊断中降低漏诊率40% |
| 中国人工智能学会 | 2024 | 中国AI医疗市场2025年预计突破500亿元 |

已将搜索结果融入论文大纲，正在开始写作...
```

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
% 使用 \renewcommand 修改摘要标题样式
% 正文内容直接写在 \begin{abstract}...\end{abstract} 之间
% 示例见下方完整模板

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
% 使用 \renewcommand 修改摘要标题样式
% 正文内容直接写在 \begin{abstract}...\end{abstract} 之间

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
% 使用 \renewcommand 修改摘要标题样式
% 正文内容直接写在 \begin{abstract}...\end{abstract} 之间

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

## 摘要格式模板（F013-补充）

> 摘要使用标准 `\begin{abstract}...\end抽象环境`，通过 `\renewcommand{\abstractname}` 修改标题样式，内容直接写在环境中。

### 摘要格式代码

```latex
% ===== 摘要标题样式 =====
% 修改摘要标题为四号黑体加粗
renewcommand{\abstractname}{\bfseries \zihao{4} 摘要}

% ===== 摘要正文（标准环境，直接填入内容）=====
\begin{document}

\begin{center}
\thispagestyle{empty}
\vspace*{2cm}

% 文档标题
\mytitle{文档标题}

\vspace{1.5cm}
\mybody{团队编号：\underline{\hspace{6cm}}}

\vspace{1cm}
\mybody{作品编号：\underline{\hspace{6cm}}}
\vspace{3cm}

\end{center}

% ===== 摘要（与封面同页）=====
\begin{center}
\textbf{摘\quad 要}
\end{center}

\begin{abstract}
\mybody{摘要正文内容，支持 \textbf{加粗} 表示关键词。}

\mybody{\textbf{关键词：}关键词1；关键词2；关键词3}
\end{abstract}

\newpage
\tableofcontents
\newpage

% ===== 正文章节 =====
...
```

### 格式要点

| 项目 | 配置值 |
|------|--------|
| 摘要标题 | `\bfseries \zihao{4} 摘要`（四号黑体加粗） |
| 摘要正文 | `\begin抽象环境}` 内直接写内容 |
| 关键词 | `\mybody{\textbf{关键词：}关键词1；关键词2}` |
| 摘要与封面 | 同页（`\thispagestyle{empty}` 控制封面无页眉页脚） |
| 标题样式 | `\mytitle{}`（18pt 加粗居中） |
| 占位符 | `团队编号：\underline{\hspace{6cm}}` |

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


## 📌 附录：完整格式配置（来自参考模板）

> 以下为可直接复用的完整 LaTeX 模板配置，提取自高质量参考文件。包含完整页面设置、字体、图片、表格、参考文献、超链接、标题格式、页眉页脚，可直接作为中文学术论文/申报书的排版基础。

### 完整模板配置

```latex
\documentclass[12pt,a4paper]{ctexart}

% ===== 页面设置 =====
\usepackage{geometry}
\geometry{top=2.8cm, bottom=2.5cm, left=3cm, right=2.6cm}
\setlength{\headheight}{14.5pt}
\usepackage{adjustbox}

% ===== 数学公式 =====
\usepackage{amsmath}
\usepackage{amssymb}

% ===== 图片 =====
\usepackage{graphicx}
\usepackage{float}
\usepackage{subcaption}

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
    % ========== 电子版（彩色可跳转） ==========
    % colorlinks=true,
    % linkcolor=blue,
    % filecolor=magenta,
    % urlcolor=cyan,
    % ========== 打印版（黑色无颜色） ==========
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

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 页眉内容}
\fancyfoot[C]{\thepage}

\begin{document}

% ===== 封面（申报书/论文封面模板） =====
\vspace*{0.5cm}
\begin{center}
\thispagestyle{empty}
\vspace*{2cm}

\mytitle{文档标题}

\vspace{1.5cm}
\mybody{团队编号：\underline{\hspace{6cm}}}

\vspace{1cm}
\mybody{作品编号：\underline{\hspace{6cm}}}
\vspace{3cm}

\end{center}

% ===== 摘要（与封面同页）=====
\mybody{摘要}
\mybody{摘要正文内容，支持 **加粗** 表示关键词。}

\mybody{\textbf{关键词：}关键词1；关键词2；关键词3}

\newpage
\tableofcontents
\newpage

% ===== 正文章节 =====
\section{章节标题}
\mybody{正文内容，使用 \mybody 命令保证统一字号。}

\section{参考文献}
\nocite{*}
\bibliographystyle{plainnat}
\bibliography{references}

\end{document}
```

### 格式要点总结

| 项目 | 配置值 |
|------|--------|
| 文档类 | `[12pt,a4paper]{ctexart}` |
| 页边距 | 上2.8cm，下2.5cm，左3cm，右2.6cm |
| **section** | `\fontsize{14pt}{18pt}` 黑体加粗 |
| **subsection** | `\fontsize{13pt}{16pt}` 楷体加粗 |
| **subsubsection** | `\fontsize{12pt}{15pt}` 加粗 |
| 正文 | `\mybody{}` 命令（统一 12pt） |
| 行距 | `\linespread{1.67}` |
| 首行缩进 | `\setlength{\parindent}{2em}` |
| **超链接** | `hidelinks`（无颜色无框） |
| 页码 | 居中底部（fancyfoot） |
| 页眉 | 居中显示（fancyhead） |
| 图表编号 | 独立编号（不随章节，counterwithout） |
| 参考文献 | `plainnat`，上标编号 |
| 封面 | 含标题/团队编号/作品编号占位符 |
| 图片包 | `subcaption`（非 subfigure） |
| 表格包 | `threeparttable`（含 tablenotes 注） |

### 参考文献格式

```latex
\bibliographystyle{plainnat}
\bibliography{references}
% 需配合 \nocite{*} 或具体 \cite{key} 使用
```

### 目录生成

```latex
\tableofcontents  % 在 \begin{document} 后，摘要之后
```

**目录控制策略（确保目录控制在一页内）：**

当目录条目在 1页～1.5页 之间时，通过以下方式压缩至一页（优先级从高到低）：

1. **删除短 subsection**：将内容较少的 subsection 直接合并到 parent section 正文
2. **减少 subsection 数量**：每个 section 保留不超过2个 subsection
3. **删除 subsubsection 层级**：目录只保留 section 和 subsection 两级
4. **收紧行距**：`\linespread{1.3}` 临时覆盖正文行距
5. **减少 dotsep**：`\def\@dotsep{2}` 减少目录点线间距

### 封面页结构（申报书/比赛文档）

```latex
% 封面居中区域
\mytitle{文档标题}           % 18pt 加粗居中
\mybody{团队编号：\underline{\hspace{6cm}}}
\mybody{作品编号：\underline{\hspace{6cm}}}

% 摘要区（与封面同页）
\mybody{摘要}
\mybody{正文...}
\mybody{\textbf{关键词：}关键词1；关键词2；关键词3}
```

### 三线表标准格式

```latex
\usepackage{booktabs}
\usepackage{threeparttable}

\begin{table}[htbp]
\centering
\begin{threeparttable}
\caption{表格标题}\label{tab:example}
\begin{tabular}{cccc}
\toprule
项目 & 指标1 & 指标2 & 备注 \\
\midrule
数据1 & 85.6 & 92.1 & 优秀 \\
数据2 & 78.3 & 86.5 & 良好 \\
\bottomrule
\end{tabular}
\begin{tablenotes}
\item 注：表格注释内容
\end{tablenotes}
\end{threeparttable}
\end{table}
```

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

---

## 协作式写作工作流（v2.0 新增）

> **核心价值**：多人协作写文档时，自动协调各方输入，管理文档版本，解决冲突，保持一致性。

### 协作架构

```
用户A（技术负责人）─────┐
用户B（产品经理）─────┼──→ 协作总线 ──→ 文档状态机
用户C（法务审核）─────┘         ↓
                         文档服务器
```

### 协作角色

| 角色 | 权限 | 典型操作 |
|------|------|---------|
| **主编** | 全部权限 | 分配章节、审核、定稿 |
| **作者** | 编写自己章节 | 写稿、提交修改 |
| **审核** | 审核权限 | 提意见、批准/打回 |
| **读者** | 只读 | 阅读、评论 |

### 文档状态机

```python
class DocumentStateMachine:
    """协作文档状态机。管理文档生命周期状态。"""

    STATES = {
        "draft": "草稿",
        "in_review": "审核中",
        "revision": "修改中",
        "approved": "已批准",
        "published": "已发布",
        "archived": "已归档"
    }

    TRANSITIONS = {
        ("draft", "submit"): "in_review",
        ("in_review", "approve"): "approved",
        ("in_review", "request_changes"): "revision",
        ("revision", "submit"): "in_review",
        ("approved", "publish"): "published",
        ("published", "archive"): "archived"
    }

    def transition(self, doc_id: str, action: str) -> str:
        """
        执行状态转换。

        检查权限 → 执行转换 → 通知相关人
        """
```

### 协作消息

```python
class CollaborationBus:
    """协作消息总线。实时同步多人写作状态。"""

    # 实时事件
    EVENTS = {
        "section.locked": "某章节被锁定，其他作者无法编辑",
        "section.updated": "某章节有新内容",
        "comment.added": "有人添加了评论",
        "review.requested": "审核请求发出",
        "review.completed": "审核完成",
        "doc.approved": "文档被批准",
        "conflict.detected": "检测到版本冲突"
    }

    def broadcast(self, doc_id: str, event: str, payload: Dict):
        """广播协作事件到所有相关方。"""
```

### 冲突解决

```python
class CollaborativeConflictResolver:
    """协作冲突解决器。自动处理多人编辑冲突。"""

    def detect_conflicts(self, doc_id: str) -> List[Conflict]:
        """
        检测版本冲突。

        冲突类型：
        1. 同一段落被多人同时修改
        2. 引用章节被删除/修改
        3. 图片/附件被替换
        """

    def resolve(self, conflict: Conflict, strategy: str = "merge") -> Resolution:
        """
        解决冲突。

        策略：
        - "merge"：尝试合并（适用于无直接冲突的修改）
        - "author_choice"：让作者选择保留哪个版本
        - "latest_wins"：最新版本覆盖
        - "manual"：人工介入解决
        """
```

### 协作工作流示例

```markdown
## 协作写作流程 — 产品需求文档

### 参与人
- 主编：张三（技术负责人）
- 作者：李四（产品经理）
- 作者：王五（设计师）
- 审核：赵六（法务）

### 协作时间线

| 时间 | 事件 | 动作 |
|------|------|------|
| 09:00 | 李四创建文档 | 文档状态：draft |
| 09:30 | 李四完成第一章 | 提交审核请求 |
| 10:00 | 赵六审核第一章 | 提出3条修改意见 |
| 10:30 | 李四修改 | 状态：revision → 再次提交 |
| 11:00 | 赵六批准第一章 | 状态：in_review → approved |
| 14:00 | 王五完成配图 | 通知李四关联 |
| 16:00 | 全部章节完成 | 张三最终审核 |
| 17:00 | 文档发布 | 状态：approved → published |

### 实时通知

📢 李四：「第一章已提交审核」
📝 赵六 评论第一章：建议补充竞品对比
✅ 赵六：「第一章已批准」
⚠️ 系统冲突检测：第三章和第五章都引用了"图1"，已自动修复引用
🎉 张三：「文档已发布」
```

### 版本控制

```python
class DocumentVersionControl:
    """文档版本控制。完整的变更历史。"""

    def commit(self, doc_id: str, author: str, changes: str):
        """提交一个版本。"""

    def diff(self, doc_id: str, v1: str, v2: str) -> str:
        """对比两个版本的差异。"""

    def rollback(self, doc_id: str, version: str):
        """回滚到指定版本。"""

    def branch(self, doc_id: str, branch_name: str) -> str:
        """创建文档分支（用于大版本重构）。"""
```

### 实时协作协议

```yaml
collaboration:
  realtime:
    protocol: "websocket"
    sync_interval: "1s"
    conflict_detection: "operational_transform"

  permissions:
    default_role: "reader"
    roles:
      - name: "editor"
        can_edit: true
        can_comment: true
      - name: "reviewer"
        can_review: true
        can_comment: true
      - name: "owner"
        can_manage: true
        can_delete: true
```

## 文档质量评分（v1.2 新增）

> **核心价值**：文档生成后自动多维度评分（可读性/完整性/引用准确性/结构/专业性），输出具体改进建议，避免交付低质量文档。

### 评分维度

| 维度 | 权重 | 评估方法 | 评分标准 |
|------|------|---------|---------|
| **可读性** | 20% | 句长/词汇难度/段落长度 | 简单(90-100)/中等(70-89)/较难(50-69)/困难(<50) |
| **完整性** | 25% | 章节覆盖率/图表数量/引用数量 | 完整(90-100)/基本完整(70-89)/缺失较多(<70) |
| **引用准确性** | 20% | 参考文献DOI验证/引用格式/数据来源 | 准确(90-100)/基本准确(70-89)/需核实(<70) |
| **结构合理性** | 20% | 逻辑流/层次清晰度/过渡衔接 | 清晰(90-100)/基本清晰(70-89)/需调整(<70) |
| **专业性** | 15% | 术语一致性/行业规范/格式规范 | 专业(90-100)/基本专业(70-89)/非正式(<70) |

### 评分工作流

```
Step 1: 多维度自动评分
- 统计句长/词汇频率 → 可读性分数
- 检查章节完整性 → 完整性分数
- 验证引用DOI格式 → 引用准确性分数
- 分析结构逻辑 → 结构分数
- 检测术语一致性 → 专业性分数

Step 2: 加权综合评分
综合分数 = Σ(维度权重 × 维度分数) / 100

Step 3: 生成改进建议
- 列出每个维度的具体问题
- 按优先级排序（低分维度优先）
- 给出具体修改位置（章节/段落）
- 提供参考改进方案

Step 4: 输出评分报告
```

### 评分报告格式

```markdown
## 文档质量评分报告

### 综合评分
**总分：78/100**（中等偏上）

### 各维度得分
| 维度 | 得分 | 等级 | 问题摘要 |
|------|------|------|---------|
| 可读性 | 85 | 良好 | 句长略长，部分段落超过8句 |
| 完整性 | 72 | 及格 | 缺少第4章的实验数据 |
| 引用准确性 | 80 | 良好 | 1个引用格式不规范 |
| 结构合理性 | 75 | 及格 | 第2章与第3章过渡不够流畅 |
| 专业性 | 88 | 优秀 | 术语使用一致，格式规范 |

### 改进建议（按优先级）

1. **[完整性] 补充实验数据** — 位置：第4章，图4-2附近
   → 建议：添加原始数据表格或引用数据来源

2. **[结构] 优化第2→3章过渡** — 位置：第2章末尾/第3章开头
   → 建议：添加过渡段落，解释两章的逻辑关系

3. **[可读性] 缩短长句段落** — 位置：第5章，第3段
   → 建议：将超过10句的段落拆分为2-3个段落

### 文档建议
- 综合评分78，建议进行小幅修订后发布
- 引用格式修正可在终稿前完成
```

### 评分阈值

| 综合分 | 状态 | 处理方式 |
|--------|------|---------|
| ≥90 | 优秀 | 直接发布，无需修改 |
| 75-89 | 良好 | 小幅修订后发布 |
| 60-74 | 及格 | 需要针对性改进 |
| <60 | 不及格 | 阻断，需全面修订 |

### 调用示例

```
用户：帮我写一份技术方案文档，然后评分
→ doc-writer生成文档初稿
→ 自动触发评分 → 输出78分报告 + 3条改进建议
→ 用户确认后修改 → 重新评分 → 通过后发布
```
