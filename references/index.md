# doc-writer 写作知识库

> 积累领域术语、格式规范和优秀案例，提升每次写作的质量和效率。

## 目录

| 目录 | 说明 | 使用场景 |
|------|------|---------|
| [references/terminology/](references/terminology/) | 领域术语库（中英对照/正式表达/禁用词）| Writer 写作时参考，Optimizer 优化时参考 |
| [references/format_specs/](references/format_specs/) | 各类型文档格式规范 | Planner 规划时参考，Reviewer 审查时核对 |
| [references/examples/](references/examples/) | 优秀案例库（高质量文档样本）| Reviewer 对照参考，Writer 学习参考 |

## 使用指南

### Writer Agent
1. 读取 `references/terminology/{domain}.json`，了解该领域的正式表达
2. 写作时避免 `forbidden` 中的词汇
3. 合理使用过渡词（`transitions`）
4. 遇到优质表达，存入 `references/terminology/custom.json`

### Optimizer Agent
1. 读取对应领域的格式规范 `references/format_specs/*.md`
2. 优化时检查术语是否符合 `references/terminology/` 的规范
3. 将优化后的优质表达归档到术语库

### Reviewer Agent
1. 读取 `references/format_specs/{doc_type}.md` 作为审查标准
2. 对照优秀案例库（`references/examples/`）中的高分案例，指出差距
3. 发现高质量文档，标注后存入优秀案例库

### Planner Agent
1. 读取 `references/format_specs/index.md` 了解可用格式规范
2. 根据用户指定的文档类型，读取对应格式规范
3. 文档结构规划时参考拓扑依赖规则

## 贡献指南

- 发现术语库缺失的表达 → 更新 `references/terminology/custom.json`
- 发现格式规范有误 → 在对应 format_specs/*.md 中修正
- 积累高质量文档 → 存入 `references/examples/{type}/`，添加元数据头
- 每季度末整理一次知识库，清理低质量条目

## 版本历史

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0.0 | 2026-05-01 | 初始版本，包含学术论文和商业文档术语库、格式规范框架 |