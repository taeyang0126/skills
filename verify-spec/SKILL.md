---
name: verify-spec
description: |
  验证 feature 实现是否与 Kiro spec 工件一致。对照 .kiro/specs/{feature}/ 下的 requirements.md、design.md、tasks.md 检查代码实现的完整性、正确性与一致性，输出结构化验证报告。
  触发场景：用户要求"验证 spec"、"检查实现是否匹配需求/设计"、"verify spec"、"验收检查"、"spec 对齐检查"时使用。
---

验证某个 feature 的代码实现是否与 `.kiro/specs/{feature}/` 下的 spec 工件一致。

## 输入

可选指定 feature 名称（即 `.kiro/specs/` 下的目录名）。
未指定时扫描 `.kiro/specs/` 列出所有含 `tasks.md` 的 feature，标注完成进度（如 "3/11 tasks"），让用户选择。
**不要猜测或自动选择，始终让用户确认。**

## 流程

### 1. 加载工件

读取 `.kiro/specs/{feature}/` 下：
- `requirements.md` — 需求（`### Requirement N: 标题` + `#### Acceptance Criteria`）
- `design.md` — 设计（`## 组件与接口`、`## 数据模型`、`## 错误处理`、`## 架构`、`## 正确性属性`）
- `tasks.md` — 任务清单（`- [ ]` / `- [x]` / `- [-]` / `- [~]`）

记录哪些工件存在、哪些缺失。

### 2. 验证三个维度

#### Completeness（完整性）

**任务完成度**：解析 `tasks.md` 复选框，统计叶子任务完成数/总数。未完成任务 → CRITICAL。

**需求覆盖度**：从 `requirements.md` 提取每项 Requirement 的关键类名、方法名、配置项，用 grepSearch 在代码库中搜索。未找到实现证据 → CRITICAL。

#### Correctness（正确性）

**验收标准映射**：对每项 Requirement 的每条 Acceptance Criteria，在代码中搜索实现证据并评估是否符合意图。偏差 → WARNING，附 `File.java:行号`。

**属性测试覆盖**：如果 `design.md` 含 `正确性属性` / `Correctness Properties` 章节，提取 `### Property N: 标题`，搜索对应 `*PropertyTest.java` 中是否有 `Property N:` 注释。缺失 → WARNING。

#### Coherence（一致性）

**设计遵循度**：对照 `design.md` 中声明的类/接口/包路径/配置/错误处理策略，检查实现是否匹配。矛盾 → WARNING。无 `design.md` 则跳过并注明。

**代码模式一致性**：检查新增代码是否符合项目模式（命名、目录、风格）。显著偏差 → SUGGESTION。

### 3. 输出报告

```
## Verification Report: {feature}

### Summary
| Dimension    | Status                              |
|--------------|-------------------------------------|
| Completeness | X/Y tasks, M/N reqs found          |
| Correctness  | M/N ACs covered, P/Q props tested  |
| Coherence    | Followed / N issues                |
```

按优先级列出问题：
1. **CRITICAL** — 未完成任务、缺失需求实现。每条附可执行建议。
2. **WARNING** — 需求/设计偏差、验收标准/属性测试缺失。每条附文件行号。
3. **SUGGESTION** — 模式偏差、小改进。每条附具体建议。

结论：
- 有 CRITICAL："发现 X 个 critical issue，请修复后再确认完成。"
- 仅 WARNING："无 critical issue。可考虑处理 Y 个 warning。实现基本完整。"
- 全通过："所有检查通过。实现与 spec 一致。"

## 启发式

- Completeness：聚焦客观清单（复选框、需求列表）
- Correctness：关键词搜索 + 合理推断，不要求绝对确定
- Coherence：关注明显不一致，不做风格挑刺
- 误报控制：不确定时 SUGGESTION > WARNING > CRITICAL
- 每个问题必须有可执行建议，附 `File.java:行号`

## 平滑降级

- 仅 `tasks.md`：只验证任务完成度
- `tasks.md` + `requirements.md`：验证完整性 + 正确性
- 三个文件齐全：验证全部三个维度
- 始终说明跳过了哪些检查及原因
