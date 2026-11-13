# L2 · 工具协议、schema 和实际执行

本批读原规范与实现。用真实案例说明何时该检查协议、返回值、运行状态或实际业务结果。

### `https://developers.openai.com/api/docs/guides/function-calling`

- 原文：[OpenAI Docs · Function calling](https://developers.openai.com/api/docs/guides/function-calling)
- 范围：请求、接收调用、执行并返回结果，tool choice、parallel、strict 和 streaming 段落。
- 方法与限制：应用负责执行模型提出的工具请求，并用调用标识匹配结果。strict 为支持的 schema 子集约束参数，流式参数在完整结束前可能还不可用。schema 不能证明工具收到的是本次目标 workload。
- 工程用法：用「返回正确信息但绑定错 call」「同名工具读错实验」和「参数格式不对」分别建回放。记录格式合法、实际状态正确与判断结果。如果校验失败源于当前 provider 不支持字段，先修转换层。

### `https://platform.openai.com/docs/api-reference/chat/create`

- 原文：[OpenAI API Reference · Chat，登记 URL 跳转后的正文](https://developers.openai.com/api/reference/resources/chat)
- 范围：Chat create 的消息与工具参数、tool_choice、parallel_tool_calls、工具回复和返回类型。
- 方法与限制：请求参数选择是否调用或限制工具，结果由消息中的工具调用和应用反馈串起。模型支持随版本变化，不应假定所有推理模式都支持同样接口。
- 工程用法：记录实际请求模型和 API 模式，在 fake server 验证多工具并发和只读执行的映射。工具业务拒绝时保留结构化错误。对照 Responses 或其他接口时，分开判协议迁移与模型能力。

### `https://modelcontextprotocol.io/specification/draft/basic/transports`

- 原文：[MCP Draft · Transport overview](https://modelcontextprotocol.io/specification/draft/basic/transports)
- 范围：stdio 和 Streamable HTTP、每请求元数据、取消、自定义 transport 和旧版本兼容，记录访问日的 draft。
- 方法与限制：底层 transport 与协议消息分开。当前草案描述的版本和能力在每请求元数据中，HTTP 可镜像到 header。放弃响应与远端操作达到何种状态需要按协议与后端验证。
- 工程用法：检查版本握手、消息关联和取消语义。用已运行、未开始、客户端断开三个状态验证。若同一个 job 仍在远端执行，恢复观察原 ID。

### `https://modelcontextprotocol.io/specification/draft/server/tools`

- 原文：[MCP Draft · Tools](https://modelcontextprotocol.io/specification/draft/server/tools)
- 范围：Discovery、分页、input/output schema、结构化结果、handles 和 Error Handling。
- 方法与限制：协议错误与工具执行错误的通道不同。工具执行错误可给模型可行动信息。工具列表可随授权改变。某些返回句柄可比连接活得更久，需要按服务定义判断生命周期。
- 工程用法：把不存在的工具、缺字段、运行尚未完成、过期运行句柄和真实编译错误分开。用已知错误和正确观测验证模型能选读配置、查状态或结束。不要统一回一个空字符串。

### `https://json-schema.org/draft/2020-12/json-schema-core.html`

- 原文：[JSON Schema Draft 2020-12 Core](https://json-schema.org/draft/2020-12/json-schema-core)
- 范围：Schema dialect、Vocabulary、URI resolution、§8.2.3 的引用与动态引用。
- 方法与限制：schema 可由声明的 dialect 和 vocabulary 决定关键字语义。引用依据文档和当前基准 URI，递归扩展有动态规则。模型 API 往往只支持规范的一部分。
- 工程用法：固定实验结果 schema 及引用解析器版本。把旧 payload、未知字段、递归引用与不同实现支持范围当成独立测试。结构没变但实际读数分母变化时，应另修数据语义检查。

### `https://json-schema.org/draft/2020-12/json-schema-validation.html`

- 原文：[JSON Schema Draft 2020-12 Validation](https://json-schema.org/draft/2020-12/json-schema-validation)
- 范围：§6 的字段和依赖校验，§7 的 Format-Annotation 与 Format-Assertion。
- 方法与限制：required 检查字段存在，dependentRequired 表达存在性依赖。format 在默认词汇和支持断言的模式下意义不同。格式即使断言也主要是语法。
- 工程用法：先校验 run ID、时间、数值类型和必要字段，再查 ID 是否指向所需运行、单位能否比较及 shape 是否匹配。用「合法日期、错误实验日」和「合法路径、错版本」验证语义层。

### `https://spec.openapis.org/oas/latest.html`

- 原文：[OpenAPI v3.2.1](https://spec.openapis.org/oas/v3.2.1.html)
- 范围：Operation Object、operationId、参数/请求/响应、Server Object 和安全要求的引用边界。
- 方法与限制：API 描述可给操作稳定的 operationId，并表达 schema 与授权入口。生成工具依赖描述与实际服务一致。规范版本与转换器支持需分别核对。
- 工程用法：从 benchmark 服务公开只读查状态与读结果。保存版本和原 operationId，针对可接受与不可接受的请求建立 fixture。升级服务时对生成后工具列表和默认值做 diff。

### `https://gofastmcp.com/integrations/openapi`

- 原文：[FastMCP · OpenAPI integration](https://gofastmcp.com/docs/integrations/openapi)
- 范围：创建组件、RouteMap、排除规则、operationId 命名和标签。
- 方法与限制：默认会投影 API route，映射可以修改类型或排除操作。组件名经过截断与去重，上游新增路径也可能改变工具面。部署行为仍由实际配置决定。
- 工程用法：投影运行管理接口时先按操作清单限制能力，再验证发现出的工具是否与预期一致。发生版本更新时先比 schema、名字与写操作。用户需求若仅为读日志，可保留少数固定只读操作作为基线。

### `https://docs.openhands.dev/sdk/arch/tool-system`

- 原文：[OpenHands SDK · Tool System & MCP](https://docs.openhands.dev/sdk/arch/tool-system)
- 范围：ToolDefinition、Action、Observation、注册、分派、MCP bridge 与发现流程。
- 方法与限制：工具声明、执行器和返回观察分别承担不同职责。MCP 输入 schema 可生成校验模型，异步协议与 SDK 执行之间有桥接。示例没有测目标工具对 GPU 时间的影响。
- 工程用法：把 trace 提取动作设计成有类型、能审查和可恢复的调用。返回原文件位置、缺失统计和单位。先对比错误处理、业务结果与完整轨迹，再决定是否并行。

### `https://github.com/OpenHands/software-agent-sdk/blob/main/openhands-sdk/openhands/sdk/agent/agent.py`

- 原文：[OpenHands SDK · agent.py 主分支](https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/agent/agent.py)
- 范围：2026-11-13 主分支 1,609 行快照，重点核对工具参数处理与校验附近 1,310–1,413 行、ActionEvent/ObservationEvent 的关联。
- 方法与限制：解析和规范化工具调用后验证参数，再产生可记录的事件。校验错误可给简洁的字段提示，且避免把参数值写进提示。主分支实现会变化。
- 工程用法：比对模型发出的名称和规范化后实际名称，记录是否触发执行。回放参数别名、缺字段、已收到观察和远端运行状态不明四类情形，检查恢复逻辑是否把失败放在正确阶段。

### `https://github.com/OpenHands/software-agent-sdk/blob/main/openhands-sdk/openhands/sdk/tool/tool.py`

- 原文：[OpenHands SDK · tool.py 主分支](https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-sdk/openhands/sdk/tool/tool.py)
- 范围：2026-11-13 主分支 874 行快照，重点为开头的 response schema 校验/缓存和 ToolDefinition 约 319–410 行。
- 方法与限制：工具声明关联动作、观察和执行器。响应 schema 可来自模型类或字典，代码进行 Draft 2020-12 验证。schema 通过不证明观测涵盖了用户要求。
- 工程用法：对 profiler 结果独立校验来源、范围和计量。示例返回合法 JSON 且标记成功，但关键原始事件丢失，应给出可追踪的不足。用该反例检验下游是否会自动接受。

### `https://gorilla.cs.berkeley.edu/blogs/15_bfcl_v4_web_search.html`

- 原文：[Berkeley Function-Calling Leaderboard V4 · Web Search](https://gorilla.cs.berkeley.edu/blogs/15_bfcl_v4_web_search.html)
- 范围：search/fetch 的设计、注入 fetch 失败、题目有效期和典型错误。
- 方法与限制：搜索定位、读网页和回答分开。人为加入常见 HTTP 或连接失败，测重试和替代 URL。题目尽量兼顾新近与可稳定核验。该场景仍是固定设计。
- 工程用法：让查手册或 issue 的助手面对原文可读、备用版本、超时与未授权四种返回。分别判是否找到匹配版本、读到必要字段和形成正确的动作。只有链接或重复查询不能算完成取证。
