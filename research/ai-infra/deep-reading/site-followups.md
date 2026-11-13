# 从登记来源扩展的站点阅读

以下页面从已登记来源的同站导航或正文找到。它们有独立阅读记录，未并入本轮冻结的 755 个原来源标识。核查日期 2026-11-13。

## llm-d · 用什么 trace 测什么问题

[From agent traces to replay](https://llm-d.ai/blog/from-agent-traces-to-replay) 来自 llm-d v1.0 发布页的同站导航。读了数据保留字段表、六种重放器与数据特定行为、四个开发阶段的对照。Mooncake 只有请求时序、长度和 hash，适合查 cache 与队列。WEKA 有会话、分支和上下文形状，内容以 hash 代替。Exgentic 保留输入、输出、工具定义和请求参数，但来源为 benchmark 任务，部分组合样本不均衡。对 Exgentic，Inference-Perf 可把 live 输出接入下游，AIPerf 两种时序模式都重放已记录请求，工具交互也没有真正执行。两者对因果依赖的保留不同。

**会改变 workload 选择。** 缓存与调度候选先用同 hash 和到达序列作可控试验。测 speculative acceptance 时需要实际输入和 target/drafter 的生成条件。测 Agent 行为需继续检查真实工具结果是否依赖新输出。按这三类任务分别写重放协议。若某数据不含决定性字段，换 trace 或补原始证据，避免根据整条会话的长度推测内容与行为。

## llm-d · 已有工具怎样缩小开发范围

[Code Assistant Skills for Accelerated llm-d Development, Evaluation and Configuration](https://llm-d.ai/blog/skills-for-accelerated-llm-d-development-evaluation-and-configuration) 来自同一发布导航。读单任务技能、autoconfig 流程、基准实验与作者结论。作者把部署、回收、配置建议、负载、清缓存和 A/B 比较做成已有组件。三个多月约 170 个两路或三路实验，超过 350 次运行。报告给使用规模与实际流程，缺与相同工程师和任务的人工时间对照。

**会改变产物比较。** 在 llm-d 场景，先把已有 smoke、配置建议和 A/B 流程作为可复用基线。把新方法的输入设为原先仍需人工判断的环节，例如解释 workload 漂移、为何换下一项实验或怎样发现宣称的收益来自测量条件。若已有组件能同样快地定位，继续用该组件，把时间留给跨框架任务。

## llm-d · 调度证据该记录到哪一层

[Seeing Through the Stack](https://llm-d.ai/blog/end-to-end-and-fine-grained-tracing-in-llm-d) 来自同站导航。读设计目标、组件级观测缺口、端到端传播与细粒度决策记录。连通一个请求的 spans 和解释为何选这个 worker 是两个数据要求。路由缓存命中、P/D 决策、后端执行和响应分别需要原始边界。指标适合总体 SLO，单请求 trace 支持判别组件归因，内部 span 太多会增加成本。

**会改变补测位置。** 若端到端延迟升高，先关联 router、scheduler 和 engine 的请求 ID，再抽取可疑区间的选择依据。对高吞吐先做低率抽样，另保留异常请求对照。没有共同 ID 时先补归属。仅有时间接近无法证明两个组件事件属于同一请求。

## llm-d · 拓扑需要逐层验证

[Topology as an End-to-End Contract](https://llm-d.ai/blog/topology-as-an-end-to-end-contract-p-d-disaggregated-inference-in-llm-d-on-gke) 来自同站导航。读网络与设备拓扑、RDMA 可达性、rail 对齐与测试平台差异。某些 GKE 配置虽然申请到了网卡，但缺失 pod 中相应路由和邻居项。作者区分临时兼容做法与平台侧修复。A4X Max 在当时测试中没有同节点 RDMA hairpin，不能套用虚拟机路径。约 2–3 µs 的额外 hop 延迟来自其设备和测试。

**会改变预检。** 把实际 GPU/NIC 位置、pod 里可见网络、预期 P/D 配对和小数据面探测放在同一运行记录。某副本传输慢时先区分未连通、跨 rail、拥塞和远端计算，再决定改放置、网络或 cache 设计。换集群后重验拓扑，避免照搬论文节点标签。

## NVIDIA · Agent 的调用状态能否改善推理服务

[Full-Stack Optimizations for Agentic Inference with NVIDIA Dynamo](https://developer.nvidia.com/blog/full-stack-optimizations-for-agentic-inference-with-nvidia-dynamo) 是从 Dynamo 主题继续在 NVIDIA 站内找到的工程文章。读三层架构、agent hints、路由、分层 KV 和生命周期。文章区分 harness、router 和 engine。harness 可以知道子任务结束、工具等待和下一轮时机，router 可以结合 KV overlap 与 decode load，engine 持有 KV。作者的 Claude Code cache 命中观测来自托管路径，NAT 自定义路由的 TTFT 对照是另一设置。预取与生命周期部分有正在构建的能力，部署时要回查目标版本。

**会改变优化次序。** 先比较同一会话图下的默认路由、固定 session affinity 与能感知排队的路由。分别采集首轮和后续轮的 cache 命中、排队、跨卡读取和 GPU 计算。再做短分支结束、长工具等待和多用户交错的对照。若提示提高命中率却挤占了活跃请求或引入预取 I/O，按各任务层的尾延迟与有效吞吐决定保留条件。
