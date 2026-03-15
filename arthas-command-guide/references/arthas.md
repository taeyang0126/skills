# Arthas 用法（整合版）

## 0. 文档定位

- 本文件已合并原 `arthas.md` 与 `common-commands.md`。
- 目标：`场景识别 -> CARD 路由 -> 输出 1 条可执行单行命令`。
- 默认只给观测命令；高风险命令需先确认。
- 检索优先级：先看 `6. 检索索引` 定位 CARD，再按需读取 `4. CARD 命令库`。

---

## 1. 固定输出契约

### 1.1 默认输出

```text
<cmd>
```

### 1.2 可选扩展（仅用户明确要求时）

```text
命令：<cmd>
说明：<一句话>
收敛：<reset/stop/无>
```

### 1.3 硬规则

1. 命令必须单行、紧凑、可直接复制执行。
2. `watch/trace/tt` 默认带 `-n`。
3. `watch` 默认最终事件（`-f` 语义）；前置/异常再用 `-b/-e/-s`。
4. 用户给了类+方法，优先方法级 `watch`；否则可用 `DispatcherServlet doDispatch`。
5. 组合过滤统一一个 condition：`header && url && parameter`。
6. `getParameter()` 仅用于 query/form；JSON body 用 `params[i]`。
7. 用户要求原始 body 文本时，先说明默认不可稳定获取，再给替代方案。
8. 默认按 CARD 模板直接生成；CARD 能满足时，不得改写为自定义 OGNL。
9. 仅当 CARD 无法表达目标时，才允许 OGNL 兜底；未明确要求时禁止复杂 OGNL。

### 1.4 质量门禁（输出前必检）

1. 范围：优先类+方法；无法精准时必须有过滤条件。
2. 成本：限制 `-n`、控制 `-x`（默认 2，必要时 3）。
3. 条件：加空值保护，避免 NPE 误判。
4. 收敛：增强后给 `reset <ClassName>`，并区分 `quit/stop`。
5. 风险：高风险命令先确认。
6. 表达式：优先使用 CARD 的稳定表达式；禁止无必要的复杂 OGNL 拼接。

---

## 2. 场景路由（SCENE_ID）

### SCENE_ID: SLOW_REQUEST

- 关键词：`慢请求`、`RT高`、`timeout`、`耗时高`
- 首选 CARD：`TRACE_COST`
- 可选 CARD：`STACK_COST`、`MONITOR_CYCLE`、`THREAD_BASE`

### SCENE_ID: INTERMITTENT_EXCEPTION

- 关键词：`偶发异常`、`报错不稳定`、`参数污染`
- 首选 CARD：`WATCH_BASE`、`WATCH_ADVICE_EVENTS`
- 可选 CARD：`WATCH_CONDITION_VERBOSE`、`TT_RECORD`

### SCENE_ID: THREAD_BLOCK_OR_DEADLOCK

- 关键词：`线程阻塞`、`deadlock`、`WAITING`、`BLOCKED`
- 首选 CARD：`THREAD_BASE`

### SCENE_ID: CLASS_CONFLICT

- 关键词：`类冲突`、`版本冲突`、`NoSuchMethodError`
- 首选 CARD：`CLASSLOADER_REQUIRED`
- 可选 CARD：`VMTOOL_INSTANCES`

### SCENE_ID: CONFIG_OR_AOP_VALUE

- 关键词：`配置值异常`、`AOP代理`、`classloader`
- 首选 CARD：`WATCH_BASE`、`OPTIONS_COMMON`
- 可选 CARD：`CLASSLOADER_REQUIRED`

### SCENE_ID: STACK_OVERFLOW_RECURSION

- 关键词：`StackOverflowError`、`递归调用`、`栈深异常`、`调用链过深`
- 首选 CARD：`STACK_DEPTH_GUARD`
- 可选 CARD：`STACK_COST`、`TT_RECORD`

### SCENE_ID: SPRING_CONFIG_DIAG

- 关键词：`Spring 配置值`、`server.port`、`配置来源`、`PropertySource`
- 首选 CARD：`SPRING_CONFIG_VALUE`、`SPRING_CONFIG_SOURCE`
- 可选 CARD：`OPTIONS_COMMON`、`CLASSLOADER_REQUIRED`

### SCENE_ID: DYNAMIC_PROXY_UNKNOWN_SOURCE

- 关键词：`Unknown Source`、`动态代理`、`$Proxy`、`CGLIB`
- 首选 CARD：`JAD_PROXY_METHOD`
- 可选 CARD：`CLASSLOADER_REQUIRED`、`MC_REDEFINE_FLOW`

### SCENE_ID: ONLINE_HOTFIX

- 关键词：`热更新`、`临时修复`、`redefine`
- 首选 CARD：`CLASSLOADER_REQUIRED`（仅确认）
- 高风险动作：`MC_REDEFINE_FLOW`（必须确认）

---

## 3. CARD 家族速查

- 方法级观测：`WATCH_BASE`、`WATCH_FILTER`、`WATCH_COLLECTION_OGNL`、`WATCH_EXPR_CONDITION_SPLIT`、`WATCH_OVERLOAD_AND_CTOR`、`WATCH_METHOD_*`、`WATCH_REQUEST_HEADERS_*`、`WATCH_ADVICE_*`
- Dispatcher 观测：`WATCH_DISPATCHER_*`、`WATCH_HEADERS_FALLBACK_DISPATCHER`
- 性能与调用链：`TRACE_COST`、`TRACE_REGEX_THREAD_FILTER`、`STACK_COST`、`STACK_DEPTH_GUARD`、`MONITOR_CYCLE`、`TT_RECORD`、`THREAD_BASE`
- 类加载与实例：`CLASSLOADER_REQUIRED`、`GETSTATIC_READ_FILTER`、`VMTOOL_INSTANCES`、`JAD_PROXY_METHOD`、`MC_REDEFINE_FLOW`
- 运行配置与采样：`LOGGER_LEVEL`、`OPTIONS_COMMON`、`VMOPTION_READ_SET`、`SPRING_CONFIG_VALUE`、`SPRING_CONFIG_SOURCE`、`OGNL_ENUM_VARARGS_NEWOBJ`、`SYS_INFO`、`HEAPDUMP_BASE`、`PROFILER_FLOW`、`JFR_FLOW`
- 执行与会话：`ASYNC_*`、`SAVE_RESULT_LOG`、`BATCH_SCRIPT_RUN`、`ATTACH_*`、`HTTP_API_*`、`RESET_STOP_QUIT`

---

## 4. CARD 命令库

### 4.1 方法级观测

#### CARD: WATCH_BASE
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' -n 5 -x 3
```

#### CARD: WATCH_FILTER
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' '<conditionExpression>' -n 5 -x 3
```

#### CARD: WATCH_ADVICE_EVENTS
```bash
watch <package.ClassName> <methodName> 'params' -b -n 3 -x 2
watch <package.ClassName> <methodName> '{params,returnObj}' -s -n 3 -x 2
watch <package.ClassName> <methodName> '{params,throwExp}' -e -n 3 -x 2
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' -f -n 3 -x 2
```

#### CARD: WATCH_ADVICE_VARS
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp,isBefore,isReturn,isThrow}' -f -n 3 -x 2
```

#### CARD: WATCH_CONDITION_VERBOSE
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' '<conditionExpression>' -v -n 5 -x 2
```

#### CARD: WATCH_COLLECTION_OGNL
```bash
watch <package.ClassName> <methodName> "params[0].{? #this.<field> != null}.{#this.<targetField>}" -n 5 -x 2
watch <package.ClassName> <methodName> "params[0].{^ #this.<field> != null}" -n 5 -x 2
watch <package.ClassName> <methodName> "params[0].{$ #this.<field> != null}" -n 5 -x 2
```

#### CARD: WATCH_EXPR_CONDITION_SPLIT
```bash
watch <package.ClassName> <methodName> "{params,returnObj,throwExp}" 'params[0].{? #this.<key>=="<value>"}.size()>0' -n 5 -x 2
watch <package.ClassName> <methodName> '{params,throwExp}' 'throwExp != null' -n 5 -x 3
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' 'params[0]=="<stringValue>"&&params[1]==<longValue>L' -n 5 -x 2
```

#### CARD: WATCH_OVERLOAD_AND_CTOR
```bash
sm <package.ClassName>
watch <package.ClassName> <init> '{params,throwExp}' -n 5 -x 2
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' 'params.length==<argCount>&&params[1] instanceof <java.Type>&&params[0]==<value>' -n 5 -x 3
```

#### CARD: WATCH_METHOD_BODY_PARAMS
```bash
watch <package.ClassName> <methodName> 'params[<bodyIndex>]' -n 5 -x 3
```

#### CARD: WATCH_METHOD_BODY_FIELD_FILTER
```bash
watch <package.ClassName> <methodName> 'params[<bodyIndex>]' 'params[<bodyIndex>]!=null&&params[<bodyIndex>].<fieldAccessor>!=null&&"<fieldValue>".equals(params[<bodyIndex>].<fieldAccessor>.toString())' -n 5 -x 3
```

#### CARD: WATCH_METHOD_HEADERS_AND_BODY
```bash
watch <package.ClassName> <methodName> '{params[<bodyIndex>],(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap())}' -n 5 -x 3
```

#### CARD: WATCH_METHOD_HEADERS_BODY_FILTER
```bash
watch <package.ClassName> <methodName> '{params[<bodyIndex>],(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap())}' '(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req!=null&&"<headerValue>".equals(#req.getHeader("<headerName>"))&&#req.getRequestURI().contains("<uriKeyword>")&&params[<bodyIndex>]!=null&&params[<bodyIndex>].<fieldAccessor>!=null&&"<fieldValue>".equals(params[<bodyIndex>].<fieldAccessor>.toString()))' -n 5 -x 3
```

#### CARD: WATCH_REQUEST_HEADERS_ALL_SINGLE
```bash
watch <package.ClassName> <methodName> '{(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap())}' -n 5 -x 2
```

#### CARD: WATCH_REQUEST_HEADERS_ALL_VALUES
```bash
watch <package.ClassName> <methodName> '{(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders())}' -n 5 -x 3
```

#### CARD: WATCH_REQUEST_HEADERS_SELECTED
```bash
watch <package.ClassName> <methodName> '{(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),"<HeaderA>="+(#req==null?null:#req.getHeader("<HeaderA>"))+", <HeaderB>="+(#req==null?null:#req.getHeader("<HeaderB>")))}' -n 5 -x 1
```

### 4.2 Dispatcher 观测

#### CARD: WATCH_HEADERS_FALLBACK_DISPATCHER
- 说明：使用 `-b` 在前置阶段读取请求头，避免后置对象状态变化影响观测结果。
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders(),params[0].getRequestURI()}' 'params[0].getRequestURI().contains("<uriKeyword>")' -b -n 5 -x 3
```

#### CARD: WATCH_DISPATCHER_HEADERS_ALL
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' -n 5 -x 2
```

#### CARD: WATCH_DISPATCHER_HEADERS_TRACE
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' '"<headerValue>".equals(params[0].getHeader("<headerName>"))' -n 5 -x 2
```

#### CARD: WATCH_DISPATCHER_HEADERS_URI
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' 'params[0].getRequestURI().contains("<uriKeyword>")' -n 5 -x 2
```

#### CARD: WATCH_DISPATCHER_HEADERS_TRACE_URI
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' '"<headerValue>".equals(params[0].getHeader("<headerName>"))&&params[0].getRequestURI().contains("<uriKeyword>")' -n 5 -x 2
```

#### CARD: WATCH_DISPATCHER_HEADERS_TRACE_URI_PARAM
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' '"<headerValue>".equals(params[0].getHeader("<headerName>"))&&params[0].getRequestURI().contains("<uriKeyword>")&&"<paramValue>".equals(params[0].getParameter("<paramName>"))' -n 5 -x 2
```

#### CARD: WATCH_DISPATCHER_REQ_META
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{params[0].getMethod(),params[0].getRequestURI(),new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()}' '"<headerValue>".equals(params[0].getHeader("<headerName>"))' -n 5 -x 2
```

#### CARD: WATCH_DISPATCHER_REQ_DEBUG
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{params[0].getMethod(),params[0].getRequestURI(),params[0].getQueryString(),params[0].getParameterMap(),new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()}' 'params[0].getRequestURI().contains("<uriKeyword>")' -n 5 -x 2
```

### 4.3 性能与调用链

#### CARD: TRACE_COST
```bash
trace <package.ClassName> <methodName> '#cost > 500' -n 5
trace --skipJDKMethods false <package.ClassName> <methodName> '#cost > 500' -n 5
```

#### CARD: TRACE_REGEX_THREAD_FILTER
```bash
trace -E '<classRegexA|classRegexB>' '<methodRegexA|methodRegexB>' '@Thread@currentThread().getName().contains("<threadNameKeyword>")&&#cost>500' -n 5
```

#### CARD: STACK_COST
```bash
stack <package.ClassName> <methodName> '#cost > 200' -n 5
```

#### CARD: STACK_DEPTH_GUARD
```bash
stack <package.ClassName> <methodName> '@java.lang.Thread@currentThread().getStackTrace().length >= <depthThreshold>' -b -n 1
```

#### CARD: MONITOR_CYCLE
```bash
monitor -c 5 <package.ClassName> <methodName>
```

#### CARD: THREAD_BASE
```bash
thread
thread -b
thread --all --state TIMED_WAITING
```

#### CARD: TT_RECORD
```bash
tt -t <package.ClassName> <methodName> -n 100
tt -l
tt -i <index> -w 'target'
tt -i <index> -w 'params'
tt -d <index>
```

### 4.4 类加载与实例

#### CARD: CLASSLOADER_REQUIRED
```bash
sc -d <package.ClassName>
ognl -c <classloaderHash> '<ognlExpression>'
classloader -t
classloader -c <classloaderHash> -r <resourceName>
classloader --load <classloaderClass>
```

#### CARD: GETSTATIC_READ_FILTER
```bash
getstatic <package.ClassName> <staticFieldName>
getstatic <package.ClassName> <staticFieldName> '<ognlExpression>'
sc -d <package.ClassName>
ognl -c <classloaderHash> '@<package.ClassName>@<staticFieldNameOrStaticMethod>'
```

#### CARD: VMTOOL_INSTANCES
```bash
vmtool --action getInstances --className <package.ClassName> --limit 10 -x 2
vmtool --action getInstances --className <package.ClassName> --express 'instances[0]' -x 2
vmtool --action forceGc
```

#### CARD: JAD_PROXY_METHOD
```bash
jad com.sun.proxy.$ProxyXX <methodName>
jad --source-only <proxyClassName> <methodName>
```

#### CARD: MC_REDEFINE_FLOW
- 说明：高风险热更新流程，执行前必须先走 `5. 高风险操作确认模板`。
```bash
jad --source-only <package.ClassName> > /tmp/<ClassName>.java
mc -c <classloaderHash> /tmp/<ClassName>.java -d /tmp
redefine /tmp/<package/path/ClassName>.class
```

### 4.5 运行配置与采样

#### CARD: LOGGER_LEVEL
```bash
logger
logger -n <loggerName>
logger --name <loggerName> --level debug
```

#### CARD: OPTIONS_COMMON
```bash
options
options unsafe true
options json-format true
options dump true
options job-timeout 1d
```

#### CARD: VMOPTION_READ_SET
```bash
vmoption
vmoption <optionName>
vmoption <optionName> <value>
```

#### CARD: SPRING_CONFIG_VALUE
```bash
vmtool --action getInstances --className org.springframework.context.ConfigurableApplicationContext --express 'instances[0].getEnvironment().getProperty("server.port")' --limit 1 -x 2
```

#### CARD: SPRING_CONFIG_SOURCE
```bash
watch org.springframework.boot.context.properties.source.ConfigurationPropertySourcesPropertySource findConfigurationProperty '{params[0],returnObj==null?null:returnObj.getValue(),returnObj==null?null:returnObj.getOrigin()}' 'params[0]!=null&&params[0].toString().contains("server.port")' -n 5 -x 3
```

#### CARD: OGNL_ENUM_VARARGS_NEWOBJ
```bash
ognl -c <classloaderHash> '@<serviceClass>@call(@<enumClass>@<enumValue>)'
ognl -c <classloaderHash> '@com.alibaba.fastjson.JSON@toJSONString(<obj>,new com.alibaba.fastjson.serializer.SerializerFeature[]{@com.alibaba.fastjson.serializer.SerializerFeature@WriteMapNullValue})'
ognl -c <classloaderHash> '#addr=new java.net.InetSocketAddress("127.0.0.1",<port>),@<managerClass>@getInstance().queryClient(#addr)'
```

#### CARD: SYS_INFO
```bash
sysprop
sysprop <propertyKey>
sysenv
sysenv <envKey>
```

#### CARD: HEAPDUMP_BASE
```bash
heapdump /tmp/heap.hprof
```

#### CARD: PROFILER_FLOW
```bash
profiler start
profiler status
profiler stop --format html --file /tmp/profile.html
profiler list
```

#### CARD: JFR_FLOW
```bash
jfr start
jfr check
jfr dump filename=/tmp/recording.jfr
jfr stop
```

#### CARD: RESET_STOP_QUIT
```bash
reset <package.ClassName>
quit
stop
```

### 4.6 执行与会话

#### CARD: ASYNC_RUN_BG
```bash
trace <package.ClassName> <methodName> '#cost > 200' -n 100 &
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' -n 50 -x 2 &
```

#### CARD: ASYNC_JOB_MANAGE
```bash
jobs
fg <jobId>
bg <jobId>
kill <jobId>
```

#### CARD: ASYNC_REDIRECT_LOG
```bash
trace <package.ClassName> <methodName> '#cost > 200' -n 200 >> <outputFile> &
```

#### CARD: SAVE_RESULT_LOG
```bash
options save-result true
options save-result false
```

#### CARD: BATCH_SCRIPT_RUN
```bash
./as.sh -f <scriptFile.as> <pid> > <outputFile>
./as.sh -c '<cmd1>; <cmd2>' <pid> > <outputFile>
```

#### CARD: ATTACH_BY_SELECT
```bash
./as.sh --select <processName>
```

#### CARD: ATTACH_BY_STAT_URL
```bash
./as.sh --stat-url http://<host>:<port>/actuator/stat
```

#### CARD: HTTP_API_EXEC
```bash
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"exec","execTimeout":30000,"command":"<arthasCommand>"}'
```

#### CARD: HTTP_API_SESSION_FLOW
```bash
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"init_session"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"async_exec","sessionId":"<sessionId>","command":"watch <package.ClassName> <methodName> \"{params,returnObj,throwExp}\" -n 3"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"pull_results","sessionId":"<sessionId>","consumerId":"<consumerId>"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"interrupt_job","sessionId":"<sessionId>"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"close_session","sessionId":"<sessionId>"}'
```

---

## 5. 高风险操作确认模板（必须原样输出）

```text
⚠️ 危险操作检测！

操作类型：运行态改值/热更新
影响范围：<类名/方法名/实例数量>
风险评估：可能导致行为变化、请求异常、实例状态不一致

请确认是否继续？[需要明确的“是”或“确认”]
```

### 5.1 风险触发映射

- `ognl` 改值（静态字段/实例字段赋值） -> 必须先输出确认模板
- `vmtool` 改值（`--express` 中包含赋值语义） -> 必须先输出确认模板
- `mc/redefine` 热更新 -> 必须先输出确认模板
- `options unsafe true` -> 必须先输出确认模板
- `vmtool --action forceGc` -> 必须先输出确认模板
- `vmoption <optionName> <value>` -> 必须先输出确认模板
- `tt -i <index> -p` 重放调用 -> 必须先输出确认模板
- `ognl` 调用写方法/赋值表达式 -> 必须先输出确认模板
- `getstatic` 使用赋值表达式（如 `#field=...`） -> 必须先输出确认模板

### 5.2 高开销轻提示（无需确认）

- `heapdump` -> 默认追加一行轻提示（可能产生明显 IO 与磁盘占用）
- `profiler start`/`jfr start` -> 默认追加一行轻提示（采样会带来额外开销）
- 大范围 `trace/watch`（未限定类方法或过滤条件过宽） -> 默认追加一行轻提示（可能放大 CPU 开销）

```text
提示：该命令属于高开销观测，可能带来额外 CPU/IO 开销，请先在最小范围执行并控制次数。
```

---

## 6. 检索索引（关键入口，优先使用）

- `命令生成优先级` -> `CARD 原样 > CARD 同家族最小改动 > OGNL 兜底`
- `复杂 OGNL` -> `仅在 CARD 无法表达或用户明确要求时使用`
- `请求头 全量（方法已知，单值）` -> `CARD: WATCH_REQUEST_HEADERS_ALL_SINGLE`
- `请求头 全量（方法已知，多值）` -> `CARD: WATCH_REQUEST_HEADERS_ALL_VALUES`
- `请求头 全量（方法未知）` -> `CARD: WATCH_DISPATCHER_HEADERS_ALL`
- `请求头过滤` -> `CARD: WATCH_DISPATCHER_HEADERS_TRACE`
- `url 过滤` -> `CARD: WATCH_DISPATCHER_HEADERS_URI`
- `请求头+url` -> `CARD: WATCH_DISPATCHER_HEADERS_TRACE_URI`
- `请求头+url+参数` -> `CARD: WATCH_DISPATCHER_HEADERS_TRACE_URI_PARAM`
- `watch 基础观测` -> `CARD: WATCH_BASE`
- `watch 条件过滤` -> `CARD: WATCH_FILTER`
- `watch 集合投影`、`watch 集合过滤`、`watch 首个匹配`、`watch 最后匹配` -> `CARD: WATCH_COLLECTION_OGNL`
- `表达式与条件分离`、`size()>0 条件收敛` -> `CARD: WATCH_EXPR_CONDITION_SPLIT`
- `构造函数 watch`、`重载方法定位`、`sm 方法签名` -> `CARD: WATCH_OVERLOAD_AND_CTOR`
- `方法请求体` -> `CARD: WATCH_METHOD_BODY_PARAMS`
- `方法请求体字段过滤` -> `CARD: WATCH_METHOD_BODY_FIELD_FILTER`
- `方法请求头+请求体` -> `CARD: WATCH_METHOD_HEADERS_AND_BODY`
- `方法 header+url+body 过滤` -> `CARD: WATCH_METHOD_HEADERS_BODY_FILTER`
- `方法级 header 全量（单值）` -> `CARD: WATCH_REQUEST_HEADERS_ALL_SINGLE`
- `方法级 header 全量（多值）` -> `CARD: WATCH_REQUEST_HEADERS_ALL_VALUES`
- `方法级 header 指定项` -> `CARD: WATCH_REQUEST_HEADERS_SELECTED`
- `Dispatcher 前置 header 回退` -> `CARD: WATCH_HEADERS_FALLBACK_DISPATCHER`
- `Dispatcher 请求元信息` -> `CARD: WATCH_DISPATCHER_REQ_META`
- `Dispatcher 请求调试` -> `CARD: WATCH_DISPATCHER_REQ_DEBUG`
- `watch 事件点 b/s/e/f` -> `CARD: WATCH_ADVICE_EVENTS`
- `Advice 变量` -> `CARD: WATCH_ADVICE_VARS`
- `条件不生效 -v` -> `CARD: WATCH_CONDITION_VERBOSE`
- `stack cost` -> `CARD: STACK_COST`
- `trace -E`、`多类多方法 trace`、`线程名过滤 trace` -> `CARD: TRACE_REGEX_THREAD_FILTER`
- `StackOverflowError 前置捕获`、`递归调用链定位` -> `CARD: STACK_DEPTH_GUARD`
- `monitor 周期统计` -> `CARD: MONITOR_CYCLE`
- `vmtool getInstances` -> `CARD: VMTOOL_INSTANCES`
- `动态代理 Unknown Source`、`jad 代理方法反编译` -> `CARD: JAD_PROXY_METHOD`
- `getstatic 静态字段`、`classloader hash`、`ognl -c` -> `CARD: GETSTATIC_READ_FILTER`
- `jad mc redefine` -> `CARD: MC_REDEFINE_FLOW`
- `logger level` -> `CARD: LOGGER_LEVEL`
- `options unsafe json-format job-timeout` -> `CARD: OPTIONS_COMMON`
- `vmoption 查询与设置` -> `CARD: VMOPTION_READ_SET`
- `spring 配置值`、`server.port 运行时值` -> `CARD: SPRING_CONFIG_VALUE`
- `findConfigurationProperty 配置来源`、`spring 配置来源` -> `CARD: SPRING_CONFIG_SOURCE`
- `ognl 枚举参数`、`ognl varargs`、`SerializerFeature 数组`、`ognl new 对象` -> `CARD: OGNL_ENUM_VARARGS_NEWOBJ`
- `sysprop sysenv` -> `CARD: SYS_INFO`
- `heapdump` -> `CARD: HEAPDUMP_BASE`
- `profiler` -> `CARD: PROFILER_FLOW`
- `jfr` -> `CARD: JFR_FLOW`
- `reset quit stop` -> `CARD: RESET_STOP_QUIT`
- `后台执行` -> `CARD: ASYNC_RUN_BG`
- `jobs fg bg kill` -> `CARD: ASYNC_JOB_MANAGE`
- `输出到日志文件` -> `CARD: ASYNC_REDIRECT_LOG`
- `save-result` -> `CARD: SAVE_RESULT_LOG`
- `批处理脚本` -> `CARD: BATCH_SCRIPT_RUN`
- `按进程名 attach` -> `CARD: ATTACH_BY_SELECT`
- `stat-url attach` -> `CARD: ATTACH_BY_STAT_URL`
- `HTTP API` -> `CARD: HTTP_API_EXEC`
- `HTTP API 会话` -> `CARD: HTTP_API_SESSION_FLOW`
- `慢调用` -> `CARD: TRACE_COST`
- `线程阻塞` -> `CARD: THREAD_BASE`
- `调用录制` -> `CARD: TT_RECORD`
- `classloader` -> `CARD: CLASSLOADER_REQUIRED`
- `高风险确认` -> `5. 高风险操作确认模板`
- `高开销提示` -> `5.2 高开销轻提示（无需确认）`
- `热更新` -> `SCENE_ID: ONLINE_HOTFIX`
- `StackOverflowError` -> `SCENE_ID: STACK_OVERFLOW_RECURSION`
- `Spring 配置排查` -> `SCENE_ID: SPRING_CONFIG_DIAG`
- `Dynamic Proxy Unknown Source` -> `SCENE_ID: DYNAMIC_PROXY_UNKNOWN_SOURCE`
