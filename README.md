# skills

用于维护常用 skill 的仓库，聚焦沉淀可复用、可维护的技能规范与参考资料。

## 仓库目的

- 统一管理常用 skill 的规则与知识库
- 持续迭代已有 skill，保证可读性与可执行性
- 为日常排障和协作提供稳定的技能资产

## 目录结构

- `arthas-command-guide/SKILL.md`：技能行为规范与执行流程
- `arthas-command-guide/references/arthas.md`：Arthas 场景索引与命令参考
- `AGENTS.md`：仓库协作与变更约定

## 快速检查

```bash
rg --files
sed -n '1,200p' arthas-command-guide/SKILL.md
rg -n "SCENE_ID|CARD|高风险" arthas-command-guide/references/arthas.md
git status --short
```

## 维护原则

- 行为规则写在 `SKILL.md`
- 可复用知识写在 `references/`
- 命令示例保持单行、可直接执行
- 涉及高风险操作必须明确确认
