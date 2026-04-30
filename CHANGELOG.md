# CHANGELOG

## [Unreleased]

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
- 摘要格式统一使用 `\renewenvironment{abstract}` 居中加粗，置于第一页底部
- 封面与摘要同页，目录紧跟摘要页后
- 参考文献格式改为 `\bibliographystyle{plainnat}` + `\bibliography{references}` 独立 .bib 文件
- 目录超出一页时按优先级策略压缩（合并小节→删 subsubsection→收紧行距→减 dotsep）
- 写作要求强化引用规范，禁止随意编造数据，所有论点必须论文支撑
- 编译流程默认使用 XeLaTeX，带 BibTeX 的完整四步编译
