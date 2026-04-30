# CHANGELOG

## [Unreleased]

---

## v0.5 — 2026-04-30

### 新增
- **三种写作模式**：单轨模式（默认）/ 多轨并行模式 / 扩轨模式
- **多轨并行模式**：2～4支团队并行独立撰写同一主题的不同方案/视角/侧重点，Leader 评选最优或合并多选
- **扩轨模式**：单队扩大规模（多个写作 Agent 并行，按章节组分工）
- **多轨评审维度**：论点新颖性、论证逻辑性、结构完整性、数据支撑、语言学术化
- 触发场景新增"多角度/多方案并行写作"
- version: v0.4 → v0.5

---

## v0.4 — 2026-04-30

### 修复
- 修复摘要格式：使用正确的 `\renewenvironment{ abstract }` 环境，不再使用 `\section{摘要}`
- 修复 `\begin.Abstract}` 为 `\begin{ abstract }`
- 参考文献使用 `\bibliographystyle{plainnat}` + `\bibliography{references}`
- 添加 `\usepackage[nottoc]{tocbibind}` 使参考文献出现在目录中
- 添加 `\renewcommand{\bibname}{参考文献}` 在 `\begin{document}` 前

### 变更
- 封面和摘要在同一页
- 目录放在摘要页之后
- 参考文献放在文档末尾、签名之前
- 参考文献前根据空白大小决定是否加 `\newpage`（如果空白小于1/2页则不加）
- 目录结构优化：合并重复章节（如"技术规格总结"和"总结"合并）

---

## v0.3 — 2026-04-30

### 修复
- 修复摘要环境名错误：`Abstract` → `abstract`（LaTeX 环境名不能大写）
- 修复 renewenvironment 语句语法错误（多余 `}` 问题）
- SKILL.md 三种模板（F002-1/2/3）和演示文档均已更正
- 文档已可正常编译，无 fatal error

---

## v0.2

---

## v0.1 — 2026-04-30

### 新增
- 六 Agent 流水线：规划 → 写作 → 内容优化（Doubao-seed-2.0-pro）→ 图表 → 格式优化 → 审查
- 四种内置模板：中文学术论文（ctexart）、IEEE英文期刊（IEEEtran）、项目申报书、科技查新报告
- 图表生成模板：LaTeX TikZ 流程图/pgfplots 折线图&柱状图/三线表/图片插入、Word 原生图表
- 多格式输出：LaTeX (pylatex)、Word (python-docx)
- 自动编译流程：XeLaTeX → BibTeX → XeLaTeX → XeLaTeX
- 完整 LaTeX 格式配置（参考模板）：页面设置、标题格式、参考文献 natbib、超链接 hidelinks、三线表 booktabs

### 变更
- 摘要格式统一使用 `\renewenvironment\Abstract}` 居中加粗，置于第一页底部
- 封面与摘要同页，目录紧跟摘要页后
- 参考文献格式改为 `\bibliographystyle{plainnat}` + `\bibliography{references}` 独立 .bib 文件
- 目录超出一页时按优先级策略压缩（合并小节→删 subsubsection→收紧行距→减 dotsep）
- 写作要求强化引用规范，禁止随意编造数据，所有论点必须论文支撑
- 编译流程默认使用 XeLaTeX，带 BibTeX 的完整四步编译
