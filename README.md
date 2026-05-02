# doc-writer

> 多智能体协同学术/商业文档写作技能，支持单轨/多轨并行/扩轨三种模式，可多支团队同时走不同方向并评选最优解。支持 LaTeX 和 Word 双格式输出。

**v3.3.1** | CherryStudio / Claude Code

## 特性

- **六 Agent 流水线**：规划 → 写作 → 内容优化（Doubao-seed-2.0-pro）→ 图表 → 格式优化 → 审查
- **四种内置模板**：中文学术论文（ctexart）、IEEE英文期刊（IEEEtran）、项目申报书、科技查新报告
- **图表生成**：LaTeX TikZ/pgfplots 流程图/折线图/柱状图、Word 原生图表
- **多格式输出**：LaTeX (pylatex)、Word (python-docx)
- **自动编译**：XeLaTeX → BibTeX → XeLaTeX → XeLaTeX，一键生成 PDF
- **引用规范**：所有论点必须有论文支撑，禁止随意编造数据

## 文件结构

```
doc-writer/
├── SKILL.md        # 主 Skill 文件（六 Agent 流水线 + 模板配置）
├── CHANGELOG.md    # 版本更新记录
└── README.md       # 本文件
```

## 安装

**CherryStudio**：将 `doc-writer/` 目录放入 Skills 目录

**Claude Code**：
```bash
claude skill install https://github.com/Air-0000/doc-writer-skill
```

---

## 相关仓库

- **doc-writer**（本仓库）：https://github.com/Air-0000/doc-writer-skill
- **code-dev-team**：https://github.com/Air-0000/code-dev-team-skill
- **chat-history-sync**：https://github.com/Air-0000/chat-history-sync-skill
- **skill-publisher**：https://github.com/Air-0000/skill-publisher
