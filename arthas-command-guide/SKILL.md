---
name: arthas-command-guide
description: 按线上故障场景生成或改写 Arthas 命令：支持从故障描述生成命令，也支持在用户提供的现有 Arthas 命令基础上最小改写。仅用于 Arthas 相关诊断，不用于非 Arthas 问题。
version: 1.1.0
---

# Arthas Command Guide

## 1. 核心契约

- 默认输出 **1 条**单行紧凑命令，可直接复制执行。
- 默认优先观测命令；不主动输出改值/热更新。
- 默认按 CARD 模板生成；CARD 能满足时，不得改写为自定义 OGNL。
- 常规场景禁用复杂 OGNL（集合投影/多变量拼接/多段三元表达式）。
- 除非用户明确要求，否则不输出分段解释。

## 2. 执行流程

1. 识别模式：`问题驱动` / `命令驱动`。
2. 提取关键槽位：`<package.ClassName>`、`<methodName>`、`<conditionExpr>`、`<headerName>`、`<headerValue>`、`<uriKeyword>`、`<paramName>`、`<paramValue>`、`<fieldAccessor>`、`<fieldValue>`、`<bodyIndex>`。
3. 查 `references/arthas.md` 的 `路由表` 定位 CARD，再读取对应命令段。
4. 问题驱动按场景关键词路由；命令驱动遵循最小改动。
5. 通过质量门禁和风险检查后输出命令。

## 3. 命令生成优先级（强制）

1. **CARD 原样**：直接使用 CARD 模板，仅替换槽位值。
2. **CARD 同家族最小改动**：仅调整 `类/方法/条件/-n/-x/事件点`，不切家族。
3. **OGNL 兜底**：仅在 CARD 无法表达目标时允许。
4. **复杂 OGNL 需显式要求**：未被用户明确要求时禁止。

## 4. 决策规则

1. **风险分级**：
   - 观测类 → 直接输出。
   - 高开销类（大范围 trace/watch、heapdump、profiler/jfr）→ 追加 1 行轻提示，不二次确认。
   - 高风险类 → 先输出确认模板，等用户确认后再给命令（详见 `references/arthas.md` 风险触发映射）。
2. **槽位缺失先澄清**：缺类/方法/过滤目标时，不猜测，先问用户。
3. **命令驱动最小改动**：保留原家族与事件点，仅改用户指定项。
4. **家族切换需授权**：原家族无法满足时，先说明并征得允许。
5. **表达式升级控制**：CARD 满足时禁止升级为自定义 OGNL。

## 5. 路由规则

- 用户给了 `<ClassName> <methodName>` → 优先方法级 CARD；方法级不可用时可回退 `WATCH_DISPATCHER_*`。
- 约束条件按 `header/url/query/body` 动态拼接，统一空值保护。
- `watch` 问题驱动默认 `-f`；命令驱动保留原事件点。
- `getParameter()` 仅用于 query/form；JSON body 用 `params[<bodyIndex>]`。
- 用户要求原始 body 文本时，先说明默认不可稳定获取，再给替代方案。
- 复杂条件表达式必须使用 `#变量` 语法，避免 `&&p` 被错误解析为 `¶` 的字符编码问题。
  - 示例：`'#req=params[0],#req.getRequestURI().contains("/api")&&#req.getParameter("id")!=null'`

## 6. 质量门禁（输出前必检）

| 检查项 | 规则 |
|--------|------|
| 范围 | 优先类+方法；无法精准时必须有过滤条件 |
| 成本 | `watch/trace/tt` 必带 `-n`；`-x` 默认 `2`，按需提升到 `3` |
| 条件 | 必须空值保护；不生效时优先 `WATCH_CONDITION_VERBOSE`（`-v`） |
| 表达式 | 优先 CARD 稳定表达式；禁止无必要的复杂 OGNL |
| 收敛 | 增强类操作可回滚；默认不额外输出第二条命令 |
| 风险 | 命中高风险触发项必须先确认 |

## 7. 输出模板

默认：
```text
<cmd>
```

扩展（仅用户明确要求解释时）：
```text
命令：<cmd>
说明：<一句话>
收敛：<reset/stop/无>
```

## 8. 端到端示例

### 示例 1：问题驱动 — 方法返回 null

用户："UserService.getUser 方法返回 null，帮我看下入参和返回值"
- 模式：问题驱动
- 槽位：`com.example.UserService` / `getUser`
- 路由：`watch 基础观测` → `WATCH_BASE`
- 输出：
```text
watch com.example.UserService getUser '{params,returnObj,throwExp}' -n 5 -x 3
```

### 示例 2：问题驱动 — 接口慢

用户："/api/order/list 接口 RT 很高，帮我排查下"
- 模式：问题驱动
- 场景：`SLOW_REQUEST` → `TRACE_COST`
- 槽位缺失：用户未给类名方法名 → 用 Dispatcher 回退
- 输出：
```text
trace org.springframework.web.servlet.DispatcherServlet doDispatch '#cost > 500' -n 5
```

### 示例 3：命令驱动 — 加过滤条件

用户："把这条命令加个 userId=123 的过滤：watch com.example.OrderService query '{params,returnObj}' -n 5 -x 2"
- 模式：命令驱动
- 最小改动：保留原家族 WATCH，仅加 condition
- 输出：
```text
watch com.example.OrderService query '{params,returnObj}' 'params[0]!=null&&"123".equals(params[0].toString())' -n 5 -x 2
```

### 示例 4：高风险 — 热更新

用户："帮我 redefine 一下 PayService"
- 模式：问题驱动
- 场景：`ONLINE_HOTFIX` → 高风险，先输出确认模板
- 输出：
```text
⚠️ 危险操作检测！

操作类型：运行态改值/热更新
影响范围：PayService
风险评估：可能导致行为变化、请求异常、实例状态不一致

请确认是否继续？[需要明确的"是"或"确认"]
```

## 参考文件

- 唯一事实源：`references/arthas.md`（路由表 / CARD 命令库 / 风险模板）
