# Arthas 参考数据

> 本文件是 CARD 命令库和路由数据的唯一事实源。决策规则、质量门禁、输出模板见 SKILL.md。

---

## 1. 统一路由表

> 根据用户输入的关键词匹配左列，定位右列 CARD，再到第 2 节读取命令模板。

### 1.1 场景路由（问题驱动优先匹配）

| 场景 | 关键词 | 首选 CARD | 可选 CARD |
|------|--------|-----------|-----------|
| 慢请求 | `慢请求` `RT高` `timeout` `耗时高` | `TRACE_COST` | `STACK_COST` `MONITOR_CYCLE` `THREAD_BASE` |
| 偶发异常 | `偶发异常` `报错不稳定` `参数污染` | `WATCH_BASE` `WATCH_ADVICE_EVENTS` | `WATCH_CONDITION_VERBOSE` `TT_RECORD` |
| 线程阻塞 | `线程阻塞` `deadlock` `WAITING` `BLOCKED` | `THREAD_BASE` | — |
| 类冲突 | `类冲突` `版本冲突` `NoSuchMethodError` | `CLASSLOADER_REQUIRED` | `VMTOOL_INSTANCES` |
| 配置/AOP | `配置值异常` `AOP代理` `classloader` | `WATCH_BASE` `OPTIONS_COMMON` | `CLASSLOADER_REQUIRED` |
| 栈溢出 | `StackOverflowError` `递归调用` `栈深异常` | `STACK_DEPTH_GUARD` | `STACK_COST` `TT_RECORD` |
| Spring配置 | `Spring配置值` `server.port` `配置来源` `PropertySource` | `SPRING_CONFIG_VALUE` `SPRING_CONFIG_SOURCE` | `OPTIONS_COMMON` `CLASSLOADER_REQUIRED` |
| 动态代理 | `Unknown Source` `动态代理` `$Proxy` `CGLIB` | `JAD_PROXY_METHOD` | `CLASSLOADER_REQUIRED` `MC_REDEFINE_FLOW` |
| 热更新 | `热更新` `临时修复` `redefine` | `CLASSLOADER_REQUIRED`(确认) | `MC_REDEFINE_FLOW`(⚠️高风险) |
| CPU热点 | `CPU高` `CPU飙高` `计算密集` `cpu热点` | `PROFILER_CPU` | `TRACE_COST` `MONITOR_CYCLE` |
| 内存分配热点 | `GC频繁` `Young GC多` `对象创建多` `内存抖动` `alloc` | `PROFILER_ALLOC` | `HEAPDUMP_BASE` |
| Native内存泄漏 | `native内存泄漏` `malloc泄漏` `堆外内存增长` `nativemem` | `PROFILER_NATIVEMEM` | `HEAPDUMP_BASE` |
| 响应慢CPU不高 | `响应慢CPU不高` `IO等待` `整体耗时` `wall-clock` | `PROFILER_WALL` | `THREAD_BASE` `TRACE_COST` |
| 锁竞争 | `锁竞争` `lock竞争` `BLOCKED多` `线程等锁` | `PROFILER_LOCK` | `THREAD_BASE` |

### 1.2 关键词 → CARD 索引

**方法级观测**
| 关键词 | CARD |
|--------|------|
| `watch 基础观测` | `WATCH_BASE` |
| `watch 条件过滤` | `WATCH_FILTER` |
| `watch 集合投影` `集合过滤` `首个匹配` `最后匹配` | `WATCH_COLLECTION_OGNL` |
| `表达式与条件分离` `size()>0 条件收敛` | `WATCH_EXPR_CONDITION_SPLIT` |
| `构造函数 watch` `重载方法定位` `sm 方法签名` | `WATCH_OVERLOAD_AND_CTOR` |
| `方法请求体` | `WATCH_METHOD_BODY_PARAMS` |
| `方法请求体字段过滤` | `WATCH_METHOD_BODY_FIELD_FILTER` |
| `方法请求头+请求体` | `WATCH_METHOD_HEADERS_AND_BODY` |
| `方法 header+url+body 过滤` | `WATCH_METHOD_HEADERS_BODY_FILTER` |
| `方法级 header 全量（单值）` `请求头全量（方法已知，单值）` | `WATCH_REQUEST_HEADERS_ALL_SINGLE` |
| `方法级 header 全量（多值）` `请求头全量（方法已知，多值）` | `WATCH_REQUEST_HEADERS_ALL_VALUES` |
| `方法级 header 指定项` | `WATCH_REQUEST_HEADERS_SELECTED` |
| `watch 事件点 b/s/e/f` | `WATCH_ADVICE_EVENTS` |
| `Advice 变量` | `WATCH_ADVICE_VARS` |
| `条件不生效 -v` | `WATCH_CONDITION_VERBOSE` |

**Dispatcher 观测**
| 关键词 | CARD |
|--------|------|
| `请求头全量（方法未知）` | `WATCH_DISPATCHER_HEADERS_ALL` |
| `请求头过滤` | `WATCH_DISPATCHER_HEADERS_TRACE` |
| `url 过滤` | `WATCH_DISPATCHER_HEADERS_URI` |
| `url+参数 过滤`（推荐，避免编码问题） | `WATCH_DISPATCHER_HEADERS_URI_PARAM_SAFE` |
| `请求头+url` | `WATCH_DISPATCHER_HEADERS_TRACE_URI` |
| `请求头+url+参数` | `WATCH_DISPATCHER_HEADERS_TRACE_URI_PARAM` |
| `Dispatcher 前置 header 回退` | `WATCH_HEADERS_FALLBACK_DISPATCHER` |
| `Dispatcher 请求元信息` | `WATCH_DISPATCHER_REQ_META` |
| `Dispatcher 请求调试` | `WATCH_DISPATCHER_REQ_DEBUG` |

**性能与调用链**
| 关键词 | CARD |
|--------|------|
| `慢调用` `trace cost` | `TRACE_COST` |
| `trace -E` `多类多方法 trace` `线程名过滤 trace` | `TRACE_REGEX_THREAD_FILTER` |
| `stack cost` | `STACK_COST` |
| `StackOverflowError 前置捕获` `递归调用链定位` | `STACK_DEPTH_GUARD` |
| `monitor 周期统计` | `MONITOR_CYCLE` |
| `调用录制` `tt` | `TT_RECORD` |
| `线程阻塞` `thread` | `THREAD_BASE` |

**类加载与实例**
| 关键词 | CARD |
|--------|------|
| `classloader` `sc -d` | `CLASSLOADER_REQUIRED` |
| `getstatic 静态字段` `ognl -c` | `GETSTATIC_READ_FILTER` |
| `vmtool getInstances` | `VMTOOL_INSTANCES` |
| `动态代理 Unknown Source` `jad 代理方法` | `JAD_PROXY_METHOD` |
| `jad mc redefine` `热更新` | `MC_REDEFINE_FLOW` |

**运行配置与采样**
| 关键词 | CARD |
|--------|------|
| `logger level` | `LOGGER_LEVEL` |
| `options unsafe json-format job-timeout` | `OPTIONS_COMMON` |
| `vmoption 查询与设置` | `VMOPTION_READ_SET` |
| `spring 配置值` `server.port 运行时值` | `SPRING_CONFIG_VALUE` |
| `spring 配置来源` `findConfigurationProperty` | `SPRING_CONFIG_SOURCE` |
| `ognl 枚举参数` `ognl varargs` `ognl new 对象` | `OGNL_ENUM_VARARGS_NEWOBJ` |
| `sysprop` `sysenv` | `SYS_INFO` |
| `heapdump` | `HEAPDUMP_BASE` |
| `profiler 基础` `profiler start` `profiler stop` `profiler list` | `PROFILER_FLOW` |
| `CPU分析` `cpu热点` `-e cpu` `CPU采样` | `PROFILER_CPU` |
| `内存分配分析` `alloc` `-e alloc` `GC压力` `对象创建` | `PROFILER_ALLOC` |
| `native内存泄漏` `nativemem` `-e nativemem` `malloc泄漏` | `PROFILER_NATIVEMEM` |
| `wall-clock` `整体耗时` `-e wall` `IO等待` `响应慢CPU不高` | `PROFILER_WALL` |
| `锁竞争分析` `-e lock` `lock profiler` `线程等锁` | `PROFILER_LOCK` |
| `Java方法追踪` `-e ClassName.methodName` `方法调用采样` | `PROFILER_JAVA_METHOD` |
| `native函数追踪` `大对象分配追踪` `线程创建追踪` `类加载追踪` | `PROFILER_NATIVE_FUNC` |
| `jfr` | `JFR_FLOW` |
| `reset` `quit` `stop` | `RESET_STOP_QUIT` |

**执行与会话**
| 关键词 | CARD |
|--------|------|
| `后台执行` | `ASYNC_RUN_BG` |
| `jobs` `fg` `bg` `kill` | `ASYNC_JOB_MANAGE` |
| `输出到日志文件` | `ASYNC_REDIRECT_LOG` |
| `save-result` | `SAVE_RESULT_LOG` |
| `批处理脚本` | `BATCH_SCRIPT_RUN` |
| `按进程名 attach` | `ATTACH_BY_SELECT` |
| `stat-url attach` | `ATTACH_BY_STAT_URL` |
| `HTTP API` | `HTTP_API_EXEC` |
| `HTTP API 会话` | `HTTP_API_SESSION_FLOW` |

---

## 2. CARD 命令库

> 槽位命名规范：
> - `<package.ClassName>` / `<methodName>` — 目标类和方法
> - `<conditionExpr>` — 条件表达式
> - `<headerName>` / `<headerValue>` — HTTP 请求头名/值
> - `<uriKeyword>` — URI 关键词片段
> - `<paramName>` / `<paramValue>` — 请求参数名/值
> - `<bodyIndex>` — 方法参数中 body 对象的索引位置
> - `<fieldAccessor>` / `<fieldValue>` — 对象字段访问器/期望值
> - `<classloaderHash>` — ClassLoader 哈希值（通过 `sc -d` 获取）

### 2.1 方法级观测

#### WATCH_BASE
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' -n 5 -x 3
```

#### WATCH_FILTER
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' '<conditionExpr>' -n 5 -x 3
```

#### WATCH_ADVICE_EVENTS
```bash
watch <package.ClassName> <methodName> 'params' -b -n 3 -x 2
watch <package.ClassName> <methodName> '{params,returnObj}' -s -n 3 -x 2
watch <package.ClassName> <methodName> '{params,throwExp}' -e -n 3 -x 2
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' -f -n 3 -x 2
```

#### WATCH_ADVICE_VARS
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp,isBefore,isReturn,isThrow}' -f -n 3 -x 2
```

#### WATCH_CONDITION_VERBOSE
```bash
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' '<conditionExpr>' -v -n 5 -x 2
```

#### WATCH_COLLECTION_OGNL
```bash
watch <package.ClassName> <methodName> "params[0].{? #this.<fieldAccessor> != null}.{#this.<fieldAccessor>}" -n 5 -x 2
watch <package.ClassName> <methodName> "params[0].{^ #this.<fieldAccessor> != null}" -n 5 -x 2
watch <package.ClassName> <methodName> "params[0].{$ #this.<fieldAccessor> != null}" -n 5 -x 2
```

#### WATCH_EXPR_CONDITION_SPLIT
```bash
watch <package.ClassName> <methodName> "{params,returnObj,throwExp}" 'params[0].{? #this.<fieldAccessor>=="<fieldValue>"}.size()>0' -n 5 -x 2
watch <package.ClassName> <methodName> '{params,throwExp}' 'throwExp != null' -n 5 -x 3
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' 'params[0]=="<fieldValue>"&&params[1]==<fieldValue>L' -n 5 -x 2
```

#### WATCH_OVERLOAD_AND_CTOR
```bash
sm <package.ClassName>
watch <package.ClassName> <init> '{params,throwExp}' -n 5 -x 2
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' 'params.length==<argCount>&&params[1] instanceof <java.Type>&&params[0]==<fieldValue>' -n 5 -x 3
```

#### WATCH_METHOD_BODY_PARAMS
```bash
watch <package.ClassName> <methodName> 'params[<bodyIndex>]' -n 5 -x 3
```

#### WATCH_METHOD_BODY_FIELD_FILTER
```bash
watch <package.ClassName> <methodName> 'params[<bodyIndex>]' 'params[<bodyIndex>]!=null&&params[<bodyIndex>].<fieldAccessor>!=null&&"<fieldValue>".equals(params[<bodyIndex>].<fieldAccessor>.toString())' -n 5 -x 3
```

#### WATCH_METHOD_HEADERS_AND_BODY
```bash
watch <package.ClassName> <methodName> '{params[<bodyIndex>],(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap())}' -n 5 -x 3
```

#### WATCH_METHOD_HEADERS_BODY_FILTER
```bash
watch <package.ClassName> <methodName> '{params[<bodyIndex>],(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap())}' '(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req!=null&&"<headerValue>".equals(#req.getHeader("<headerName>"))&&#req.getRequestURI().contains("<uriKeyword>")&&params[<bodyIndex>]!=null&&params[<bodyIndex>].<fieldAccessor>!=null&&"<fieldValue>".equals(params[<bodyIndex>].<fieldAccessor>.toString()))' -n 5 -x 3
```

#### WATCH_REQUEST_HEADERS_ALL_SINGLE
```bash
watch <package.ClassName> <methodName> '{(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap())}' -n 5 -x 2
```

#### WATCH_REQUEST_HEADERS_ALL_VALUES
```bash
watch <package.ClassName> <methodName> '{(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),#req==null?null:new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders())}' -n 5 -x 3
```

#### WATCH_REQUEST_HEADERS_SELECTED
```bash
watch <package.ClassName> <methodName> '{(#ra=@org.springframework.web.context.request.RequestContextHolder@getRequestAttributes(),#req=#ra==null?null:#ra.getRequest(),"<headerName>="+(#req==null?null:#req.getHeader("<headerName>")))}' -n 5 -x 1
```

### 2.2 Dispatcher 观测

#### WATCH_HEADERS_FALLBACK_DISPATCHER
说明：`-b` 前置阶段读取请求头，避免后置对象状态变化。
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders(),params[0].getRequestURI()}' 'params[0].getRequestURI().contains("<uriKeyword>")' -b -n 5 -x 3
```

#### WATCH_DISPATCHER_HEADERS_ALL
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' -n 5 -x 2
```

#### WATCH_DISPATCHER_HEADERS_TRACE
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' '"<headerValue>".equals(params[0].getHeader("<headerName>"))' -n 5 -x 2
```

#### WATCH_DISPATCHER_HEADERS_URI
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' 'params[0].getRequestURI().contains("<uriKeyword>")' -n 5 -x 2
```

#### WATCH_DISPATCHER_HEADERS_URI_PARAM_SAFE
说明：`#变量` 语法避免 `&&p` 编码问题，适用于 URI + 参数组合过滤。
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{#req=params[0],new org.springframework.http.server.ServletServerHttpRequest(#req).getHeaders().toSingleValueMap()}' '#req=params[0],#req.getRequestURI().contains("<uriKeyword>")&&#req.getParameter("<paramName>")!=null&&#req.getParameter("<paramName>").contains("<paramValue>")' -n 5 -x 2
```

#### WATCH_DISPATCHER_HEADERS_TRACE_URI
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' '"<headerValue>".equals(params[0].getHeader("<headerName>"))&&params[0].getRequestURI().contains("<uriKeyword>")' -n 5 -x 2
```

#### WATCH_DISPATCHER_HEADERS_TRACE_URI_PARAM
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch 'new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()' '"<headerValue>".equals(params[0].getHeader("<headerName>"))&&params[0].getRequestURI().contains("<uriKeyword>")&&"<paramValue>".equals(params[0].getParameter("<paramName>"))' -n 5 -x 2
```

#### WATCH_DISPATCHER_REQ_META
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{params[0].getMethod(),params[0].getRequestURI(),new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()}' '"<headerValue>".equals(params[0].getHeader("<headerName>"))' -n 5 -x 2
```

#### WATCH_DISPATCHER_REQ_DEBUG
```bash
watch org.springframework.web.servlet.DispatcherServlet doDispatch '{params[0].getMethod(),params[0].getRequestURI(),params[0].getQueryString(),params[0].getParameterMap(),new org.springframework.http.server.ServletServerHttpRequest(params[0]).getHeaders().toSingleValueMap()}' 'params[0].getRequestURI().contains("<uriKeyword>")' -n 5 -x 2
```

### 2.3 性能与调用链

#### TRACE_COST
```bash
trace <package.ClassName> <methodName> '#cost > 500' -n 5
trace --skipJDKMethods false <package.ClassName> <methodName> '#cost > 500' -n 5
```

#### TRACE_REGEX_THREAD_FILTER
```bash
trace -E '<classRegexA|classRegexB>' '<methodRegexA|methodRegexB>' '@Thread@currentThread().getName().contains("<threadNameKeyword>")&&#cost>500' -n 5
```

#### STACK_COST
```bash
stack <package.ClassName> <methodName> '#cost > 200' -n 5
```

#### STACK_DEPTH_GUARD
```bash
stack <package.ClassName> <methodName> '@java.lang.Thread@currentThread().getStackTrace().length >= <depthThreshold>' -b -n 1
```

#### MONITOR_CYCLE
```bash
monitor -c 5 <package.ClassName> <methodName>
```

#### THREAD_BASE
```bash
thread
thread -b
thread --all --state TIMED_WAITING
```

#### TT_RECORD
```bash
tt -t <package.ClassName> <methodName> -n 100
tt -l
tt -i <index> -w 'target'
tt -i <index> -w 'params'
tt -d <index>
```

### 2.4 类加载与实例

#### CLASSLOADER_REQUIRED
```bash
sc -d <package.ClassName>
ognl -c <classloaderHash> '<ognlExpression>'
classloader -t
classloader -c <classloaderHash> -r <resourceName>
classloader --load <classloaderClass>
```

#### GETSTATIC_READ_FILTER
```bash
getstatic <package.ClassName> <staticFieldName>
getstatic <package.ClassName> <staticFieldName> '<ognlExpression>'
sc -d <package.ClassName>
ognl -c <classloaderHash> '@<package.ClassName>@<staticFieldNameOrStaticMethod>'
```

#### VMTOOL_INSTANCES
```bash
vmtool --action getInstances --className <package.ClassName> --limit 10 -x 2
vmtool --action getInstances --className <package.ClassName> --express 'instances[0]' -x 2
vmtool --action forceGc
```

#### JAD_PROXY_METHOD
```bash
jad com.sun.proxy.$ProxyXX <methodName>
jad --source-only <proxyClassName> <methodName>
```

#### MC_REDEFINE_FLOW
⚠️ 高风险：执行前必须先走第 3 节确认模板。
```bash
jad --source-only <package.ClassName> > /tmp/<ClassName>.java
mc -c <classloaderHash> /tmp/<ClassName>.java -d /tmp
redefine /tmp/<package/path/ClassName>.class
```

### 2.5 运行配置与采样

#### LOGGER_LEVEL
```bash
logger
logger -n <loggerName>
logger --name <loggerName> --level debug
```

#### OPTIONS_COMMON
```bash
options
options unsafe true
options json-format true
options dump true
options job-timeout 1d
```

#### VMOPTION_READ_SET
```bash
vmoption
vmoption <optionName>
vmoption <optionName> <fieldValue>
```

#### SPRING_CONFIG_VALUE
```bash
vmtool --action getInstances --className org.springframework.context.ConfigurableApplicationContext --express 'instances[0].getEnvironment().getProperty("<propertyKey>")' --limit 1 -x 2
```

#### SPRING_CONFIG_SOURCE
```bash
watch org.springframework.boot.context.properties.source.ConfigurationPropertySourcesPropertySource findConfigurationProperty '{params[0],returnObj==null?null:returnObj.getValue(),returnObj==null?null:returnObj.getOrigin()}' 'params[0]!=null&&params[0].toString().contains("<propertyKey>")' -n 5 -x 3
```

#### OGNL_ENUM_VARARGS_NEWOBJ
```bash
ognl -c <classloaderHash> '@<serviceClass>@call(@<enumClass>@<enumValue>)'
ognl -c <classloaderHash> '@com.alibaba.fastjson.JSON@toJSONString(<obj>,new com.alibaba.fastjson.serializer.SerializerFeature[]{@com.alibaba.fastjson.serializer.SerializerFeature@WriteMapNullValue})'
ognl -c <classloaderHash> '#addr=new java.net.InetSocketAddress("127.0.0.1",<port>),@<managerClass>@getInstance().queryClient(#addr)'
```

#### SYS_INFO
```bash
sysprop
sysprop <propertyKey>
sysenv
sysenv <envKey>
```

#### HEAPDUMP_BASE
```bash
heapdump /tmp/heap.hprof
```

#### PROFILER_CPU
说明：CPU 耗时分析，定位计算密集型热点代码。方块宽度正比于 CPU 时间消耗。
```bash
# 手动 start/stop
profiler start --event cpu
profiler stop --format html --file /tmp/cpu.html
# 指定时长自动停止
profiler start --event cpu --duration <seconds> --file /tmp/cpu.html --format html
```

#### PROFILER_ALLOC
说明：堆内存分配分析，定位大量创建临时对象的代码，减少 GC 压力。方块宽度正比于分配的堆内存大小。
```bash
# 手动 start/stop
profiler start --event alloc
profiler stop --format html --file /tmp/alloc.html
# 指定时长自动停止
profiler start --event alloc --duration <seconds> --file /tmp/alloc.html --format html
```

#### PROFILER_NATIVEMEM
说明：Native 内存泄漏分析，记录 malloc/realloc/calloc/free 调用并匹配地址，聚焦未释放的分配。需配合 jfrconv 处理。
```bash
# Arthas 命令：启动采集
profiler start --event nativemem --file /tmp/app.jfr
profiler stop
# Shell 命令（非 Arthas）：处理 jfr 文件生成泄漏报告
jfrconv --total --nativemem --leak /tmp/app.jfr /tmp/app-leak.html
```

#### PROFILER_WALL
说明：Wall-clock 时间分析，采样所有线程（含休眠/IO等待），适用于 CPU 不高但响应慢的场景。方块宽度正比于真实物理时间。
```bash
# 手动 start/stop
profiler start --event wall --threads --interval 50ms
profiler stop --format html --file /tmp/wall.html
# 指定时长自动停止
profiler start --event wall --threads --interval 50ms --duration <seconds> --file /tmp/wall.html --format html
```

#### PROFILER_LOCK
说明：锁竞争分析，采样被 synchronized 阻塞的线程，定位锁竞争热点。方块宽度正比于等待锁的总时间。
```bash
# 手动 start/stop
profiler start --event lock
profiler stop --format html --file /tmp/lock.html
# 指定时长自动停止
profiler start --event lock --duration <seconds> --file /tmp/lock.html --format html
```

#### PROFILER_JAVA_METHOD
说明：Java 方法级追踪，instrument 指定 Java 方法并记录所有调用的调用栈。
```bash
profiler start --event <package.ClassName>.<methodName>
profiler start --event <package.ClassName>.<methodName> --duration <seconds>
profiler stop --format html --file /tmp/method.html
```

#### PROFILER_NATIVE_FUNC
说明：Native 函数追踪，用于追踪 JVM 内部特定操作（大对象分配、线程创建、类加载等）。
```bash
profiler start --event G1CollectedHeap::humongous_obj_allocate
profiler start --event JVM_StartThread
profiler start --event Java_java_lang_ClassLoader_defineClass1
profiler stop --format html --file /tmp/native.html
```

#### PROFILER_FLOW
说明：Profiler 基础操作（启停、状态查看、已采集样本数、支持事件列表）。
```bash
profiler start
profiler status
profiler getSamples
profiler stop --format html --file /tmp/profile.html
profiler list
# 容器场景：启动 HTTP 服务在线查看火焰图
apt-get update && apt-get install -y python3
python3 -m http.server 8080 -d /tmp
```

#### JFR_FLOW
```bash
jfr start
jfr check
jfr dump filename=/tmp/recording.jfr
jfr stop
```

#### RESET_STOP_QUIT
```bash
reset <package.ClassName>
quit
stop
```

### 2.6 执行与会话

#### ASYNC_RUN_BG
```bash
trace <package.ClassName> <methodName> '#cost > 200' -n 100 &
watch <package.ClassName> <methodName> '{params,returnObj,throwExp}' -n 50 -x 2 &
```

#### ASYNC_JOB_MANAGE
```bash
jobs
fg <jobId>
bg <jobId>
kill <jobId>
```

#### ASYNC_REDIRECT_LOG
```bash
trace <package.ClassName> <methodName> '#cost > 200' -n 200 >> <outputFile> &
```

#### SAVE_RESULT_LOG
```bash
options save-result true
options save-result false
```

#### BATCH_SCRIPT_RUN
```bash
./as.sh -f <scriptFile.as> <pid> > <outputFile>
./as.sh -c '<cmd1>; <cmd2>' <pid> > <outputFile>
```

#### ATTACH_BY_SELECT
```bash
./as.sh --select <processName>
```

#### ATTACH_BY_STAT_URL
```bash
./as.sh --stat-url http://<host>:<port>/actuator/stat
```

#### HTTP_API_EXEC
```bash
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"exec","execTimeout":30000,"command":"<arthasCommand>"}'
```

#### HTTP_API_SESSION_FLOW
```bash
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"init_session"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"async_exec","sessionId":"<sessionId>","command":"watch <package.ClassName> <methodName> \"{params,returnObj,throwExp}\" -n 3"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"pull_results","sessionId":"<sessionId>","consumerId":"<consumerId>"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"interrupt_job","sessionId":"<sessionId>"}'
curl -Ss -XPOST http://127.0.0.1:8563/api -d '{"action":"close_session","sessionId":"<sessionId>"}'
```

---

## 3. 风险控制

### 3.1 高风险确认模板（必须原样输出）

```text
⚠️ 危险操作检测！

操作类型：运行态改值/热更新
影响范围：<类名/方法名/实例数量>
风险评估：可能导致行为变化、请求异常、实例状态不一致

请确认是否继续？[需要明确的"是"或"确认"]
```

### 3.2 风险触发映射

| 触发条件 | 动作 |
|----------|------|
| `ognl` 改值（静态字段/实例字段赋值） | 必须先输出确认模板 |
| `vmtool` 改值（`--express` 含赋值语义） | 必须先输出确认模板 |
| `mc/redefine` 热更新 | 必须先输出确认模板 |
| `options unsafe true` | 必须先输出确认模板 |
| `vmtool --action forceGc` | 必须先输出确认模板 |
| `vmoption <optionName> <value>` | 必须先输出确认模板 |
| `tt -i <index> -p` 重放调用 | 必须先输出确认模板 |
| `ognl` 调用写方法/赋值表达式 | 必须先输出确认模板 |
| `getstatic` 使用赋值表达式 | 必须先输出确认模板 |

### 3.3 高开销轻提示（无需确认，追加 1 行）

| 触发条件 | 提示 |
|----------|------|
| `heapdump` | 可能产生明显 IO 与磁盘占用 |
| `profiler start` / `jfr start` | 采样会带来额外开销 |
| 大范围 `trace/watch`（未限定类方法或条件过宽） | 可能放大 CPU 开销 |

```text
提示：该命令属于高开销观测，可能带来额外 CPU/IO 开销，请先在最小范围执行并控制次数。
```