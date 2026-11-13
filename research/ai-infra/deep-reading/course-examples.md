# 教学地址与本地端点

以下七项来自课件或作业的原样链接。回到原处读示例的动作、输入和限制。只有确切的来源才能支持真实服务的可达性或性能。两条课内本地演示链接所指的独立文件未在已核资料中取得，保持单列。

### `https://weather.example/weather`

- 原文：[L2 p.40 · REST 调用示例](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf#page=40)
- 范围：p.40 的请求路径、参数、凭据和说明。原 URL 是课件里的示例端点。
- 状态：示例已核
- 启发：将「选对操作和参数」与「成功取得业务结果」作为两次判断。用本地 fixture 测 profiler 提取接口的字段、shape 和错误响应，避免用 demo 成功推断真实服务可用。凭据由受控工具持有。

### `https://weather.example/openapi.json`

- 原文：[L2 p.43 · OpenAPI 投影为 MCP tools](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf#page=43)
- 范围：p.43 的 schema 加载、operation 到 tool 和 base URL 配置。原 URL 为示例 schema 地址。
- 状态：示例已核
- 启发：自动投影可以减少手写 schema，但工具面会随上游 spec 变化。冻结 spec，比较版本差异，测试是否只发现所需的只读操作，并用真实 fixture 核对返回值。可先对编译日志/运行状态接口做契约测试。

### `https://weather.example`

- 原文：[L2 pp.43–44 · MCP 与上游授权边界](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf#page=43)
- 范围：pp.43–44 的 base URL 和两端凭据。原 URL 为示例服务根。
- 状态：示例已核
- 启发：把访问工具与工具访问实际后端的权限分别测试。验证一个只读诊断流程只拿到状态和产物，不获得启动 GPU 或修改后端的能力。用受控拒绝样本核对错误能否反馈给调用者。

### `http://localhost:12445/examples/blank-name.html?mode=fixed`

- 原文：[L6 p.58 · Verifying a frontend repair](https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf#page=58)
- 范围：读课件截图标签、行为与外观验收、修补条件。localhost 是授课机器端点，独立演示文件未取得。
- 状态：原件受限
- 启发：把目标错误和原有行为同时固定下来。案例要求拒绝空输入且正常输入仍能提交。迁移到 vLLM，单测既应证明被跳过的请求可在应有轮次获得配额，也应保留已访问请求的回收与预算行为。

### `http://localhost:12445/images/importance-weights.png`

- 原文：[L11 p.29 · Why limit an update?](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf#page=29)
- 范围：读课件该页的标题、说明与图来源。localhost 是授课机器端点，独立原图未取得。
- 状态：原件受限
- 启发：RL 采样数据陈旧时，均值可能掩盖少量样本贡献过高。按策略 lag、任务和位置看重要性比率尾部和有效样本量，再检查 clipping、异常 token 或分母问题。图示只给方向，不能从中读实际数值。

### `http://localhost:8000/v1`

- 原文：[A3 README 固定提交，导出后本机评估示例](https://github.com/cmu-agents/assignment-3/blob/88d1b1fa2aea58938d6d251847ce5559c4ac6558/README.md#L137)
- 范围：本地 A3 README 128–145 行的导出、模型名、endpoint 和提交文件。原 URL 为运行者自己启动服务时的本机接口。
- 状态：示例已核
- 启发：先验证实际选中的 base/adapter 与导出指纹，再比较任务分数。API 返回成功不能证明请求走了指定 adapter。保留一个已知应有差异的输入，核对部署前后的输出和路由。

### `https://your-workspace--scottyrl-evaluation-serve.us-east.modal.run:`

- 原文：[A3 ASSIGNMENT 固定提交，Part 5 部署和评估](https://github.com/cmu-agents/assignment-3/blob/88d1b1fa2aea58938d6d251847ce5559c4ac6558/ASSIGNMENT.md#L336)
- 范围：本地 ASSIGNMENT 326–346 行。your-workspace 是占位，末尾冒号来自注释。
- 状态：示例已核
- 启发：记录部署输出的实际运行标识和评估对象。工具应区分地址占位、未部署、就绪但模型错、评估失分。用保存的配置和日志回放判断，避免为了确认地址而触发一次训练。
