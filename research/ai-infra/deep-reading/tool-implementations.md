# L2 · 模型工具格式与接口实现

### `https://huggingface.co/Qwen/Qwen2.5-7B-Instruct/blob/main/tokenizer_config.json`

- 原文：[Qwen2.5-7B-Instruct tokenizer_config](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct/blob/main/tokenizer_config.json)
- 范围：2026-11-13 主分支的 chat_template、工具列表、调用和结果角色分支。
- 工程用法：该模板把逻辑调用渲染为标签内的 JSON，工具回复的角色边界由模板决定。用空参数、多个结果和混合文本对拍渲染后的 token ID。若转码后参数仍可读但 token 边界不同，先查模板或后端，避免归到模型质量。

### `https://huggingface.co/Qwen/Qwen3.8-27B/blob/main/tokenizer_config.json`

- 原文：[Qwen3.8-27B tokenizer_config](https://huggingface.co/Qwen/Qwen3.8-27B/blob/main/tokenizer_config.json)
- 范围：2026-11-13 主分支的 token 表和 response 解析配置，核对 function 和 parameter 分支。
- 工程用法：参数解析含具名标签和类型约束，不能直接拿 Qwen2.5 的 JSON 解析器判错。迁移测试覆盖零参数、多参数、括号、特殊字符及响应再编码。比较规范化动作及原始 token 两份结果。

### `https://huggingface.co/mistralai/Mistral-Small-4-119B-2603`

- 原文：[Mistral-Small-4-119B-2603 模型卡](https://huggingface.co/mistralai/Mistral-Small-4-119B-2603)
- 范围：模型与推理说明、工具调用和图片示例，2026-11-13 页面。
- 工程用法：目标推理栈还要确认模板、解析器、工具和实际资源。用固定请求核查能力前先记 revision 与推理配置，再分开测文本、图像及工具返回。示例成功无法推出大负载时延。

### `https://github.com/mistralai/cookbook/blob/main/concept-deep-dive/tokenization/tool_calling.md`

- 原文：[Mistral Cookbook · Tool Calling Tokenization](https://github.com/mistralai/cookbook/blob/main/concept-deep-dive/tokenization/tool_calling.md)
- 范围：控制 token、工具定义、调用、结果及多个 tokenizer 版本的例子。
- 工程用法：检查工具调用和结果是否保留匹配的 call ID。格式差异含控制 token、顺序和空白。做 tokenize/decode 往返，比较原始 ID 与实际部署。如果只是文本相似，继续检查训练 mask 和首个输出 token。

### `https://huggingface.co/deepseek-ai/DeepSeek-V3.2/blob/main/encoding/encoding_dsv32.py`

- 原文：[DeepSeek-V3.2 · encoding_dsv32.py](https://huggingface.co/deepseek-ai/DeepSeek-V3.2/blob/main/encoding/encoding_dsv32.py)
- 范围：2026-11-13 主分支的消息编码与工具调用解析，重点为 DSML、参数类型与停止边界。
- 工程用法：统一业务结构时保留原格式和解析状态。用字符串、非字符串、多个调用和结束标记做 fixture。遇到空调用先分清模型未调用、解析失败和后端过滤。

### `https://gorilla.cs.berkeley.edu/leaderboard.html`

- 原文：[Berkeley Function Calling Leaderboard V4](https://gorilla.cs.berkeley.edu/leaderboard.html)
- 范围：公开指标、模型对照和交互 demo 的两种输出说明。方法见课表 V4 文章。
- 工程用法：区分原输出、可解析的标准化动作及执行结果。榜单会变化，若模型在榜上更好，仍用当前工具、版本和错误日志回放验证。先测目标任务分层。

### `https://openrouter.ai/`

- 原文：[OpenRouter 官方首页](https://openrouter.ai/)
- 范围：入口的 routing、provider failover、控制和用量面板介绍。未取得 L2 原演示的统计快照。
- 工程用法：有 provider fallback 时保存实际 provider、模型标识、重试、响应状态和时延。固定 provider 做归因对照，再独立评估路由的延迟和可用性。

### `https://fastapi.tiangolo.com/tutorial/first-steps/`

- 原文：[FastAPI · First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/)
- 范围：path operation、自动 API 文档与 OpenAPI JSON 的相关段落。
- 工程用法：保存部署实际生成的 API 描述并与本地契约对照。测新增可选字段、空数组、缺响应类型。生成文档只能说明框架知道声明，部署还需核对实际操作。

### `https://fastapi.tiangolo.com/tutorial/security/`

- 原文：[FastAPI · Security](https://fastapi.tiangolo.com/tutorial/security/)
- 范围：OpenAPI 安全类型、fastapi.security 与交互文档的关系。
- 工程用法：分别核对 GPU 接口文档声明和实际执行的授权。用测试身份验证只读角色可读产物，创建任务路径有独立限制。schema 变更、调用被拒与远端故障分别返回。

### `https://blog.modelcontextprotocol.io/posts/2026-07-28/`

- 原文：[MCP · The 2026-07-28 Specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- 范围：stateless core、MRTR、授权演进和 SDK 迁移说明。
- 工程用法：跨断开继续时显式保存业务状态，避免用传输 session 充当运行 ID。测重连、重新发现工具与业务动作的关系。升级时分开验证协议错误和权限错误。

### `https://ts.sdk.modelcontextprotocol.io/v2/`

- 原文：[MCP TypeScript SDK v2](https://ts.sdk.modelcontextprotocol.io/v2/)
- 范围：v2 SDK 入口、客户端与服务端指南入口、transport 选择及版本说明。
- 工程用法：保存 SDK 与协议版本，再验收发现、输入输出和恢复。对 TS 与 Python 各做最小读取测试，检查能否取得同一业务状态。该入口未提供实际调优结果。

### `https://gofastmcp.com/servers/auth/token-verification`

- 原文：[FastMCP · Token Verification](https://gofastmcp.com/docs/servers/auth/token-verification)
- 范围：JWTVerifier、签名、issuer/audience、JWKS 刷新与示例边界。
- 工程用法：区分过期、audience 不匹配、授权不足与网络失败。用测试凭据确认轮换时的行为。读产物和创建作业需要不同权限，短期故障和错误权限也不应触发同一重试。

### `https://gofastmcp.com/clients/auth/bearer`

- 原文：[FastMCP · Bearer Token Authentication](https://gofastmcp.com/docs/clients/auth/bearer)
- 范围：客户端和 transport 的 auth 参数、BearerAuth、预置 token 的使用边界。
- 工程用法：将凭据留在客户端配置，回放与日志保留调用身份、作用域和授权结果。若客户端已创建却无预期工具，先核授权和目标服务，再查模型的工具选择。

### `https://huggingface.co/docs/transformers/chat_templating`

- 原文：[Transformers Chat templates，L2 登记路径](https://huggingface.co/docs/transformers/chat_templating)
- 范围：角色和 control token、Using apply_chat_template、continue_final_message、Model training。与 L8 登记的英语路径对应，分别核对可达正文。
- 方法与限制：相同 base model 的两个微调版本可使用不同 role token。add_generation_prompt 开始下一条 assistant 消息，continue_final_message 去掉结尾以续写末条，两者不可同时用。训练预处理通常关闭开始新消息的提示。具体 token 由模板决定。
- 工程用法：本地 tool trace 保留序列化后的 token、角色、模板 hash 和 action mask。用「末条是 user」「assistant 预填」「有工具结果」对照。不匹配时先定位第一个 token 分歧。 [L8 训练格式](sft-data.md)
