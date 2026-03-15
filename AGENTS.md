# Repository Guidelines

## 项目结构与模块组织
当前仓库仅包含一个技能包：

- `arthas-command-guide/SKILL.md`：技能主规范，定义路由规则、输出约束与执行流程。
- `arthas-command-guide/references/arthas.md`：参考资料索引，包含 `SCENE_ID`、`CARD` 与命令样例。

请将“行为规则”放在 `SKILL.md`，将“可复用知识库”放在 `references/`，避免两者混写。

## 构建、测试与开发命令
本仓库暂无编译型构建流程，主要做内容校验：

- `rg --files`：快速确认仓库文件结构。
- `sed -n '1,200p' arthas-command-guide/SKILL.md`：快速审阅技能规范。
- `rg -n "SCENE_ID|CARD|高风险" arthas-command-guide/references/arthas.md`：校验索引锚点与安全章节。
- `git status --short`：确认仅包含预期改动。

## 编码风格与命名规范
- 统一使用 Markdown，段落简洁、表述确定，避免歧义。
- 可直接执行的命令示例必须保持单行，不换行拆分。
- 复用既有命名：`SCENE_ID`、`CARD`、全大写路由标识。
- 优先增量修改，避免大范围重写，便于追踪行为变更。

## 测试指南
当前未配置自动化测试，采用“内容回归”方式验证：

- 至少覆盖 2 类场景：`问题驱动` 与 `命令驱动`。
- 检查输出约束是否保持：只输出 1 条最终命令、范围可控、风险收敛。
- 检查 `SKILL.md` 中引用的条目在 `references/arthas.md` 中均可定位。

## 提交与合并请求规范
仓库目前无历史提交，建议从现在起采用统一格式：

- 提交信息：`docs(skill): <变更说明>` 或 `feat(skill): <行为变更>`。
- 单次提交聚焦一个主题：路由、风险门禁或参考资料更新。
- PR 需包含：变更目的、修改文件、前后行为示例、风险说明（尤其高风险命令）。

## 安全与配置提示
默认不得输出会直接修改生产状态的指导。涉及高风险操作（如 `ognl`/`vmtool` 改值、`mc/redefine`）时，必须显式要求用户确认后再给出方案。
