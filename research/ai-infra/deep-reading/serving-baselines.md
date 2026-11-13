# 工作负载与现成流程 · NVIDIA Dynamo、AIPerf、llm-d

从原有扩展索引继续读实现与文档。这里区分优化流程基线、服务运行的负载基线和能判别机制的原始观测。文档案例的硬件、版本与目标范围不继承到本地项目。

### `https://docs.nvidia.com/dynamo/agent-skills/optimization`

- 原文：[NVIDIA Dynamo · Performance Optimization Skills](https://docs.nvidia.com/dynamo/agent-skills/optimization)
- 范围：v1.6.1 文档的八个技能阶段、前置和配合使用的部署与评估入口。
- 方法与限制：先形成 workload 合同和确认基线，再选和审查一个候选，按同一 benchmark 和 SLO 对照。此页列流程和技能，并无 DeepInfer 或 CUDA kernel 层的效果验证。
- 工程用法：把这个现成流程作为服务调优基线。将同一实际请求、指标和 GPU 预算交给该流程与有更自由证据访问的方案。检查后者能否在遇到真实运行差异时选到更有效的测试。

### `https://github.com/ai-dynamo/dynamo/blob/v1.6.1/agent-docs/guides/optimization/optimize-loop.md`

- 原文：[Dynamo v1.6.1 · optimize-loop.md](https://github.com/ai-dynamo/dynamo/blob/v1.6.1/agent-docs/guides/optimization/optimize-loop.md)
- 范围：角色、baseline 进入路径、workload/series 约束、候选循环、停止和可复现报告。
- 方法与限制：基线来自用户或已确认的 recipe，固定工作负载及判分方式。部署、跑基准、分析、候选生成和挑战职责明确。影响 baseline 或 measurement series 的更改需要可追溯。文档描述闭环能表达什么，不给用户工作中的采用收益。
- 工程用法：在已有材料中抽一段「fused+prefetch 的 p50 与 p95 方向不同」的状态，交给现成对照逻辑。检查是否要求重复基线并在用户明确的 SLO 上排序。若只有源代码和依赖的解析能力缺失，优先接入只读观测。

### `https://raw.githubusercontent.com/ai-dynamo/dynamo/v1.6.1/.agents/skills/consult-perf-knowledge/SKILL.md`

- 原文：[Dynamo v1.6.1 · consult-perf-knowledge](https://raw.githubusercontent.com/ai-dynamo/dynamo/v1.6.1/.agents/skills/consult-perf-knowledge/SKILL.md)
- 范围：输入工作负载指纹、现有 baseline、版本核验、优先筛候选大类和避免噪声区间调参的规则。
- 方法与限制：先看部署实际使用的引擎与 workload，再读对应知识。跨版本、硬件、拓扑或任务的推断要留假设。邻近候选都落在噪声内时回到重新观察或其他机制。该 skill 仍依赖正确的证据和工具。
- 工程用法：对同一个 GPU 计时反转，比较按噪声规则暂停搜索与继续探索。若 A/A 已显示测量分辨率不足，优先改变采集条件。若只是某个 shape 触及调度边界，先按 shape 隔离再生成候选。

### `https://docs.nvidia.com/dynamo/dev/agents/agent-simulation`

- 原文：[Dynamo dev · Simulate an Agent Run](https://docs.nvidia.com/dynamo/dev/agents/agent-simulation)
- 范围：捕获请求、请求图、live/offline replay、What This Captures 和模拟边界。
- 方法与限制：保留到达、长度、会话依赖及 prefix 相关信息，用于 serving workload replay。它不让原 agent 重新决定下一步，也不重新执行原工具。默认合成内容难以保留与真实语义相关的接受率或内容反馈。
- 工程用法：如果问题是缓存与队列，优先用相同请求图隔离路由改动。若问题是 drafter 接受率、输出内容驱动的下一步或工具时延，补保真输入与真实交互对照。比较两个 replay 在 token 计数、prefix 命中、draft acceptance 与尾延迟各自保留多少。

### `https://github.com/ai-dynamo/aiperf/blob/main/docs/benchmark-modes/trace-replay.md`

- 原文：[AIPerf · Trace Replay](https://github.com/ai-dynamo/aiperf/blob/main/docs/benchmark-modes/trace-replay.md)
- 范围：trace format、时序/调度、会话和预先格式化 messages 的使用说明。
- 方法与限制：结构化到达 trace 可用于重现负载节奏。预格式化 messages 可保留真实输入文本，但生成长度和模型实际行为仍由目标服务决定。此处为 main 文档。
- 工程用法：同一 trace 中固定历史 message、目标输出限制和 tokenizer，比较 baseline 与候选。若只有 prefix hash 和长度，先记录为合成负载，再另用可用的公开消息校验接受率。出现时间差时分辨 replay 发起时间、路由排队和实际生成。

### `https://llm-d.ai/docs/operations/validation/benchmark`

- 原文：[llm-d · Performance assessment](https://llm-d.ai/docs/operations/validation/benchmark)
- 范围：Available workload profiles、Customizing the workload、harness 指标、Workspace and results layout 及资源清理。
- 方法与限制：guide profile 重现文档的负载阶梯，sanity 只检查基本功能。不同 harness 会保留自己的原始结果。统一报告的 TTFT、ITL、TPOT、E2E 和吞吐仍需要核对测量方式与配置覆盖。
- 工程用法：比较 prefill/decode 拆分前后时，先保存已渲染清单、实际并发与请求分布。sanity 通过后再测延迟曲线与有效吞吐。若在总体吞吐提高时 ITL/SLO 退化，按目标决定改变池配比还是停止方案。

### `https://llm-d.ai/docs/operations/traffic/flow-control`

- 原文：[llm-d · Queue and Prioritize Requests](https://llm-d.ai/docs/operations/traffic/flow-control)
- 范围：flow control 的概念与适用场景、功能验证、Benchmarking、队列观测和资源。
- 方法与限制：流量控制与基准负载生成器分别配置。文档示例可验证队列和负载状态，但多租户公平性需要可区分的请求组成与目标指标。
- 工程用法：比较两个路由配置时，先固定各请求到达和业务类别。在同压强下分别查看排队、拒绝、每租户 SLO 与目标吞吐。若只记录总体 QPS，补出队与分类读数，再判断是队列策略、单副本饱和还是 KV 压力。

### `https://llm-d.ai/docs/dev/architecture/disaggregation/pd-disaggregation`

- 原文：[llm-d dev · Disaggregated Serving](https://llm-d.ai/docs/dev/architecture/disaggregation/pd-disaggregation)
- 范围：机制、When to use P/D and how to tune it、路由、KV 传输和限制。
- 方法与限制：独立预填和解码池可减少长输入干扰，并按各自资源需求扩缩容。要支付 KV 传输、额外排队和部署资源成本。长输入与中大型模型更有可能受益，适用性取决于输入输出比例、资源配比和互联。
- 工程用法：以代表性请求、固定模型和同总卡数比较聚合和拆分。先核验路由真的走两段，传输成功，并区分 TTFT、TPOT、队列与失败。若短输入负载没有稳定改善，不从长输入结果外推。

### `https://llm-d.ai/docs/well-lit-paths/foundations/pd-disaggregation`

- 原文：[llm-d v1.0 · Deploy Prefill/Decode Disaggregation](https://llm-d.ai/docs/well-lit-paths/foundations/pd-disaggregation)
- 范围：部署原理、默认 NVIDIA 拓扑、Verify、Observe、DisaggregatedSet 运行。
- 方法与限制：默认演示为 1P TP1 与 1D TP4，共五卡。分别由 router 选 P 与 D，通过 connector 传 KV。部署资源就绪只证明能够运行，不能给出最佳池比。
- 工程用法：记录真实路由、模型端 prefill/decode 请求、跨实例缓存命中和传输状态。可按路径完成性先做便宜验证。若没有真正走 P/D，修 connector 或选择器，再做配对性能实验。

### `https://llm-d.ai/docs/architecture/kv-management`

- 原文：[llm-d · KV Cache Management](https://llm-d.ai/docs/architecture/kv-management)
- 范围：prefix-aware routing、索引、offload、peer sharing 各层定位和关系。
- 方法与限制：路由可能用估计或真实索引找 prefix，offload 增加容量，peer sharing 可借索引选择远端 KV 来源。复用收益受命中、传输带宽和本地重算成本限制。
- 工程用法：用相同 token 前缀与到达序列，分别记路由命中、KV 实际取回、缓存失效和 TTFT。若路由选择正确而目标块未取回，先定位传输或容量。若取回成本超过重算，重新选择复用条件。

### `https://llm-d.ai/docs/dev/operations/observability/router-metrics`

- 原文：[llm-d dev · Router Metrics](https://llm-d.ai/docs/dev/operations/observability/router-metrics)
- 范围：各 endpoint 队列和 KV 利用率、聚合与分布、KV index/events、NIXL 失败。
- 方法与限制：池均值可看整体压力，每 endpoint 和离散程度用于查不均。路由和传输计数提供相关证据，不能自动给出请求关键路径。
- 工程用法：若平均队列正常而 p99 坏，先按副本、长度和是否命中分层，再追路由和传输错误。用同请求时间线验证。若证据显示只有单个 endpoint 饱和，优先比较路由策略；若全池都饱和，再查容量或 kernel。
