# A1 与 A3 · 工具协议、运行环境和实验记录

### `https://agentskills.io/home`

- 原文：[Agent Skills Overview](https://agentskills.io/home)
- 范围：目录结构、发现、激活、按需加载与兼容性。2026-11-13。
- 方法与限制：先加载 name/description，任务匹配再加载 SKILL.md，其它引用和脚本按需读。可减少上下文开销，是否正确匹配、脚本权限和版本要由运行时验证。
- 工程用法：用同一批 mixed workload 比较全量操作手册、按需 skill 与独立脚本。分别测匹配成功、误触发、token、额外工具动作及最终结果。优先抽取可重复、容易漏条件的过程。 [标准细则](memory-02.md)

### `https://developers.openai.com/api/reference/python/resources/chat/subresources/completions/methods/create`

- 原文：[OpenAI Chat Completions Python reference](https://developers.openai.com/api/reference/python/resources/chat/subresources/completions/methods/create)
- 范围：messages、assistant tool_calls、tool result ID、tool_choice、parallel_tool_calls、finish_reason 与 completion 长度。2026-11-13。
- 方法与限制：工具选择由模型输出和请求条件控制，真正调用由客户端完成。消息需要保留 ID 和角色，模型支持的选项各异。单个 finish_reason 无法证明外部动作执行或任务完成。
- 工程用法：A1 客户端用同一状态串对照单调用、并行调用、错误参数、传输失败和重入。按 call ID 验证回执归属，再用外部状态判断是否完成。工具选择的结构正确性与调用结果分开评估。

### `https://docs.vllm.ai/en/v0.7.2/getting_started/examples/openai_chat_completion_client_with_tools.html`

- 原文：[vLLM v0.7.2 · OpenAI Chat Completion Client With Tools](https://docs.vllm.ai/en/v0.7.2/getting_started/examples/openai_chat_completion_client_with_tools.html)
- 范围：指定版本启动说明、示例 tools/arguments、dispatcher、tool_call_id 和第二次调用。示例未执行。
- 方法与限制：客户端解析参数、选本地函数，随后附回 tool result。服务端的工具解析器与 chat template 影响模型返回。示例的模型和参数范围属于归档版。
- 工程用法：为使用 OpenAI-compatible 的实际服务写最小协议探针，记录响应类型、工具名、参数、finish_reason 和第 2 轮是否读到回执。协议探针通过后再比任务能力。换 vLLM 版本或 parser 时重跑。

### `https://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation`

- 原文：[作业登记的 FEN 入口](https://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation) · [Chess Programming Wiki 的字段说明](https://chessprogramming.org/Forsyth-Edwards_Notation)
- 范围：A1 §3.3 的六字段输入，核对字段、例子和表示边界。此处按课程接口应用。
- 状态：示例已核
- 方法与限制：棋子位置、轮到谁、王车易位、吃过路兵目标、半步计数和全步数一起表达当前局面。某些重复局面等判断还依赖历史。简化棋盘不足以恢复全部合法动作。
- 工程用法：把 GPU 运行恢复视作类似状态序列化。当前日志行只能解释表面。还要有 commit、编译产物、request/job ID、已完成尝试和正在生效的约束。回放时用两个表面相同、状态不同的输入测试是否会采取错误动作。

### `https://en.wikipedia.org/wiki/Universal_Chess_Interface`

- 原文：[作业登记的 UCI 入口](https://en.wikipedia.org/wiki/Universal_Chess_Interface) · [UCI 2006 作者规范转录](https://backscattering.de/chess/uci/)
- 范围：协议用途、命令和 move format，回到 A1 的 JSON 字段，未运行引擎。
- 状态：示例已核
- 方法与限制：引擎协议用文本交换位置、动作和输出。A1 HTTP 只使用 UCI 行棋字符串，不要求客户端实现整套进程协议。升变 move 带后缀。
- 工程用法：把“表示合法”“当前状态动作合法”“runner 接受”做成不同检查。GPU 调优的配置同样需要先做 schema 检查，再查设备、输入尺寸与预算，最后看实际执行回执。

### `https://github.com/cmu-agents/chess-app.git`

- 原文：[课程 chess-app 原仓库](https://github.com/cmu-agents/chess-app) · [A1 子模块固定 commit](https://github.com/cmu-agents/chess-app/tree/4aaabca1194ef5f175ccae4fecf902bc71cc303b)
- 范围：本地 A1 gitlink 与子模块 HEAD 同为 4aaabca；README、server.py L35–54、game.py L60–112。未运行服务。
- 方法与限制：state 读当前棋局，move 改局并触发确定性对手，非法输入返回错误。A1 simulate 一步与 live move 可有不同副作用与步数，需要使用实际作业端点合同。
- 工程用法：用明确 dry run 和 commit 接口来比较候选，不重复请求有副作用的动作。动作回执缺失时先查当前状态。复现评估固定子模块和环境，不从界面或依赖的最新分支猜契约。

### `https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/chatgpt?tabs=python-key%2Cdotnet-secure%2Cjavascript-secure&pivots=programming-language-python`

- 原文：[Microsoft Foundry · Work with chat completion models](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/chatgpt)
- 范围：登记的 query 变体在网页工具不可达，读同站同路径正文的 Python client、消息、部署标识和 token budget。
- 方法与限制：云部署名、端点和认证与直连服务不同。多轮需要客户端传历史或使用对应服务状态。不同模型可用的 token 字段也有区别。
- 工程用法：同一个 A1 工具调用任务先跑 provider smoke，记录部署 ID、真实模型、模板、可用参数和错误状态。对比 cost、延迟、invalid actions 要固定这些条件，不能只按模型名称聚合。

### `https://docs.nvidia.com/nemo/rl/latest/design-docs/sequence-packing-and-dynamic-batching.html`

- 原文：[NVIDIA NeMo-RL · Sequence Packing and Dynamic Batching](https://docs.nvidia.com/nemo/rl/latest/design-docs/sequence-packing-and-dynamic-batching.html)
- 范围：Problem、packing、dynamic batching、configuration 与 metrics。2026-11-13。
- 方法与限制：packing 拼接不同长度样本并保留边界，dynamic batching 在 chunk 内按长度与 token 预算组 microbatch。所读版本两者互斥，packing 还依赖后端与 attention 实现。文档里的 padding 比例不是任意训练负载的常数。
- 工程用法：从实际 rollout 算序列长度分布、每步 padding token、最慢 rank 和峰值。固定训练样本与 loss 分母比较三种批处理，先核对 mask、梯度或 loss 等价。再量真实 tokens/s 与同质量训练时间。遇到低长度差时降低其优先度。

### `https://modal.com/`

- 原文：[Modal](https://modal.com/) · [GPU acceleration](https://modal.com/docs/guide/gpu) · [Volumes](https://modal.com/docs/guide/volumes)
- 范围：官网产品导航与同站 GPU 资源、选择设备、卷挂载和可见性文档。2026-11-13。
- 方法与限制：运行在声明资源的 worker 上，卷有跨容器可见和提交规则。请求资源与实际获取型号、冷启动和文件可见时点需要运行确认。
- 工程用法：记录远端 call ID、实际 GPU 型号、镜像、输出卷路径、产物校验。若日志结束却本地没文件，先查 commit/可见性或任务最终状态。复用已有运行可少付重试成本。

### `https://modal.com/pricing`

- 原文：[Modal · Pricing](https://modal.com/pricing)
- 范围：2026-11-13 的 GPU/CPU/内存资源表、按秒口径、计划边界。
- 方法与限制：GPU 标价按 GPU 秒，CPU 与内存有独立费率。页面的 B300 USD 0.001972/秒约为 USD 7.10/GPU-hour。它是公开列表价，不等于用户账单和实际可用性。
- 工程用法：将候选实验按实际卡数、可计费运行和并行等待估 GPU-hours，另记冷启动、空闲、失败以及人工时间。候选缩短墙钟但增加 GPU-hours 时，按任务截止期和预算取舍。

### `https://wandb.ai`

- 原文：[Weights & Biases 官网](https://wandb.ai) · [官方 SDK 仓库](https://github.com/wandb/wandb)
- 范围：官网实验和追踪入口、SDK README 的 init/config/log，交叉读取 A3 `scottyrl/metrics.py` 的 run_id 与 step 写入。
- 方法与限制：A3 可以将 config、标量和 run ID 发送到运行日志。训练本地的 step 与外部 dashboard 恢复/聚合状态有不同边界。动态曲线只证明这些值被写入。
- 工程用法：保留原始本地日志与 checkpoint，按 run ID 对上模型版本、真实完成的 step 和采样分母。对比续跑时检查重复 step、时间坐标、异常退出和完整验证。面板只能作为索引，关键结论回到回执和原始运行。
