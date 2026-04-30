---
name: doc-writer
version: v0.3
description: |
  多智能体协同学术/商业文档写作技能，支持 LaTeX 和 Word 双格式输出。

  触发场景（满足任一即触发）：
  - 用户要求撰写、修改、润色学术论文（期刊论文、学位论文、会议论文）
  - 用户要求撰写项目申报书/项目介绍书/可行性报告
  - 用户要求撰写科技查新报告
  - 用户要求生成 LaTeX 或 Word 格式的正式长文档
  - 用户上传模板文件（.tex/.cls/.sty/.docx/.dotx）并要求基于模板写作
  - 用户要求将 Markdown 内容转换为 LaTeX 或 Word 格式
  - 用户询问某种文档类型的标准格式规范
  - 用户要求绘制流程图、折线图、柱状图并插入文档

  不触发场景：
  - 简单的问答或短文本生成（少于500字）
  - 代码生成任务（应触发 code skill）
  - 已完成的正式文档的纯翻译需求

  核心能力：
  - 六 Agent 流水线：规划 → 写作 → 内容优化（Doubao-seed-2.0-pro）→ 图表 → 格式优化 → 审查
  - 四种内置模板：中文学术论文、IEEE英文期刊、项目申报书、查新报告
  - 图表生成：LaTeX TikZ/pgfplots 流程图/折线图/柱状图、Word 原生图表
  - 多格式输出：LaTeX (pylatex)、Word (python-docx)、编译说明
  - 自动编译：XeLaTeX → BibTeX → XeLaTeX → XeLaTeX
---

# 文档写作助手 (doc-writer)

多智能体协同流水线，通过五个专业 Agent 协作完成高质量学术/商业文档写作。

## 流水线架构

```
规划Agent ──→ 写作Agent ──→ 内容优化Agent ──→ 图表Agent ──→ 格式优化Agent ──→ 审查Agent
   │              │                  │                  │               │               │
   ↓              ↓                  ↓                  ↓               ↓               ↓
分析模板       填充内容          Doubao-seed-2.0-pro   插入图表       LaTeX/Word      三重审查
制定计划      章节写作           语言润色/逻辑强化      TikZ/pgfplots   格式规范化      逻辑/格式/内容
```

### Agent 职责矩阵

| Agent | 核心职责 | 输出物 | 失败降级 |
|-------|---------|--------|---------|
| **规划Agent** | 解析模板结构 + 制定写作计划 | 章节大纲 + 字数分配 + 写作策略 | 单 Agent 模式（规划+写作合并） |
| **写作Agent** | 按模板格式填充章节内容 | 完整正文（Markdown） | 直接输出大纲级内容 |
| **内容优化Agent** | 调用 Doubao-seed-2.0-pro 优化文本 | 润色后的 Markdown 正文 | 跳过优化，使用原始内容 |
| **图表Agent** | TikZ 代码生成 / Word 图表插入 | 图表代码块 + 插入位置标记 | 跳过图表步骤，提示用户手动插入 |
| **格式优化Agent** | LaTeX pylatex 脚本 / Word python-docx 脚本 | 格式化的 .tex / .docx 文件 | 输出 Markdown + 转换命令 |
| **审查Agent** | 逻辑完整性 + 格式规范性 + 内容质量 | 审查报告 + 修改建议 | 跳过审查，用户自行检查 |

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

---

## 第五阶段：图表 Agent

**角色定义：** 图表工程师，负责生成 LaTeX TikZ/pgfplots 代码或 Word 原生图表。

**输入契约：**
- 内容优化 Agent 输出的 Markdown 文档
- 图表类型需求（流程图/折线图/柱状图/表格/图片）
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
识别图表类型（流程图/折线图/柱状图/表格/普通图片）
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

1. **图表类型判断**：流程图/折线图/柱状图/表格/普通图片
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

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 文档页眉}
\fancyfoot[C]{\thepage}

\begin{document}

% ===== 封面（含标题、日期、摘要）=====
\thispagestyle{empty}
\vspace*{0.5cm}
\begin{center}
\vspace*{2cm}
\mytitle{论文标题}
\vspace{1cm}
\mybody{作者姓名~~单位名称~~邮箱}
\vspace{1cm}
\mybody{2026年4月}
\end{center}
\vspace{2cm}
\renewenvironment.Abstract}{\par\begin{center}\textbf{\fontsize{16pt}{20pt}\selectfont 摘要}\end{center}\par}{\par}
\begin.Abstract}
本文摘要内容...
\textbf{关键词：}关键词1；关键词2；关键词3
\end\Abstract}
\newpage

% ===== 目录（摘要页后紧跟，目录内容过多时合并小节）=====
\tableofcontents
\newpage

% ===== 正文章节 =====

% 参考文献
\bibliographystyle{plainnat}
\bibliography{references}

% 附录
\appendix
\section{附录A}

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

\begin{document}

% ===== 封面（含标题、日期、摘要）=====
\thispagestyle{empty}
\vspace*{0.5cm}
\begin{center}
\vspace*{2cm}
\mytitle{论文标题}
\vspace{1cm}
\mybody{作者姓名~~单位名称~~邮箱}
\vspace{1cm}
\mybody{2026年4月}
\end{center}
\vspace{2cm}
\renewenvironment.Abstract}{\par\begin{center}\textbf{\fontsize{16pt}{20pt}\selectfont 摘要}\end{center}\par}{\par}
\begin.Abstract}
本文摘要内容...
\textbf{关键词：}关键词1；关键词2；关键词3
\end\Abstract}
\newpage

% ===== 目录（摘要页后紧跟，目录内容过多时合并小节）=====
\tableofcontents
\newpage

% ===== 正文章节 =====

% ===== 参考文献 =====
\newpage
\thispagestyle{empty}
\vspace*{2cm}
\bibliographystyle{plainnat}
\bibliography{references}

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
| 结构 | 报告封面→查新内容与要求→检索范围与策略→检索结果→查新结论→附件 |
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

% ===== 页眉页脚 =====
\usepackage{fancyhdr}
\pagestyle{fancy}
\fancyhf{}
\fancyhead[C]{\mybody 页眉内容}
\fancyfoot[C]{\thepage}

\begin{document}

% ===== 封面（含标题、日期、摘要）=====
\thispagestyle{empty}
\vspace*{0.5cm}
\begin{center}
\vspace*{2cm}
\mytitle{论文标题}
\vspace{1cm}
\mybody{作者姓名~~单位名称~~邮箱}
\vspace{1cm}
\mybody{2026年4月}
\end{center}
\vspace{2cm}
\renewenvironment.Abstract}{\par\begin{center}\textbf{\fontsize{16pt}{20pt}\selectfont 摘要}\end{center}\par}{\par}
\begin.Abstract}
本文摘要内容...
\textbf{关键词：}关键词1；关键词2；关键词3
\end\Abstract}
\newpage

% ===== 目录（摘要页后紧跟，目录内容过多时合并小节）=====
\tableofcontents
\newpage

\section{查新内容与要求}
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
\newpage
\thispagestyle{empty}
\vspace*{2cm}
\bibliographystyle{plainnat}
\bibliography{references}

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

当目录条目略超一页时，通过以下方式压缩（优先级从高到低）：

1. **删除 subsection 层级的独立标题**：将 `section` 下的短内容 subsection 直接合并到 parent section 正文中，不单独列条目
2. **减少 subsection 数量**：每个 section 保留不超过2个 subsection 列出
3. **删除 subsubsection 层级**：只保留 section 和 subsection 两级目录
4. **收紧行距**：`\linespread{1.3}` 临时覆盖正文行距
5. **减少 dotsep**：`\def\@dotsep{2}` 减少目录点线间距
