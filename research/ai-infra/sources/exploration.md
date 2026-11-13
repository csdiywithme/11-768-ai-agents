# 扩展阅读记录

核验日期为 2026-11-12。课内引文、外部工程资料和本地历史记录分开归属。网页 `latest`、`dev` 或 `main` 只表示访问时看到的文档。用到历史运行时以当地工具版本、仓库提交和原始日志为准。

## NVIDIA · 从课程引文和 B300 案例双向展开

| 入口与检查范围 | 实际读到的材料 | 对决策的作用 |
|---|---|---|
| L03 课表与 slides p.36 引文 → Megatron Core 文档的 Parallelism Strategies、Context Parallel Overview | [Context Parallel Overview](https://docs.nvidia.com/megatron-core/developer-guide/latest/user-guide/features/context_parallel.html) 的 TP2CP2 图与 KV 交换段 | CP 切分训练序列激活。它减少每卡保留的 KV，增加注意力所需的 KV 交换。此处 CP 的权衡不能直接套到 DeepInfer decode 测量。 |
| A3 数据流水线引文 → NeMo RL 的 Design Docs | [Sequence Packing and Dynamic Batching](https://docs.nvidia.com/nemo/rl/latest/design-docs/sequence-packing-and-dynamic-batching.html) 的 Problem、Packing、CP、PP 与兼容性 | A3 将单条多轮轨迹按 token 前缀拼接。NeMo RL 打包多条独立样本来减少 padding，还要保持各自的 attention 与 loss 边界。这是两个不同层次。动态 shape 与 Megatron PP 的固定通信 buffer 存在兼容限制。 |
| B300 案例反向查 `Nsight Compute replay clock cache workload duration` → Profiling Guide、Compute Triage、Technical Blog | [Profiling Guide §§2.2.3、2.6](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html) · [Compute Triage Guide](https://docs.nvidia.com/nsight-compute/ComputeTriage/) · [CUDA Toolbox 示例 §§2–6](https://developer.nvidia.com/blog/the-modern-cuda-toolbox-in-practice-a-step-by-step-optimization-walkthrough/) | Replay 方式、缓存清理和锁频会改变测量。示例用 Nsight Systems 找到 CUDA kernel，优化后再看到分配和 H2D。它展示如何重查关键路径。文中图像处理收益不作为 GEMM 或 serving 的数字。 |
| B300 案例反向查 `Blackwell block-scaled GEMM tile cluster schedule` → CUTLASS 文档 | [Blackwell SM100 GEMMs](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/blackwell_functionality.html) 的支持形状、dispatch 和 block-scaled 例子 | tile、cluster、dtype/layout 会改变指令和 schedule。该页以 SM100 为范围。本地 B300 的 SM103a、具体 TMEM/SMEM 和实测驻留仍按本地资源与 profiler 记录判断。 |
| DeepInfer 失败反向查 `NVSHMEM GDAKI IBGDA GPUNetIO prerequisites` → NVIDIA 安装与通信文档 | [Using NVSHMEM · GDAKI prerequisites](https://docs.nvidia.com/nvshmem/api/latest/using.html#using-gpudirect-async-kernel-initiated-communication) · [NVSHMEM Performance · Remote Transport Selection](https://docs.nvidia.com/nvshmem/release-notes-install-guide/best-practice-guide/performance.html) | 普通 verbs、CPU proxy 和 GPU-initiated 路径需要分别记录。文档列出的组件版本和网络条件可用于准备矩阵，不能反推出本地 event-channel EINVAL 的根因。 |

**对案例判断的改变。** B300 的同轮计时中，固定 BN128 的两种 grid 可检验并发贡献。没有 BN256/296 对照，因此无法完整估计 tile 与 grid 的交互项。实际驻留与依赖指标用于进一步解释机制。DeepInfer 原有组件时长只有在与下一轮 GPU consumer 的依赖边连接后，才能判断该改动是否降低 decode 延迟。优先把采集方式、比较口径和证据层级纳入检查器。[B300 案例](../cases/01-b300-gemm.md) · [DeepInfer 案例](../cases/03-deepinfer-profiling.md)

**覆盖边界。** 检查了上述目录和定位段落。没有遍历 NVIDIA 站点全量文章。剩下最有区分力的证据是同配置整机 trace、实际驻留与目标传输路径的成功日志。继续阅读相近的入门文章很难补上这些本地证据。

## llm-d · 从 DistServe 到部署和运行时证据

从 L03 引用的 DistServe 进入。用 `llm-d P/D tuning goodput ISL OSL`、`llm-d benchmark spec prefix cache` 和 `llm-d DisaggregatedSet release` 查项目官网、Well-Lit Paths、架构、操作指标、benchmark 仓库和发布记录。llm-d 是扩展阅读来源。

| 阅读层 | 一手材料和定位 | 对决策的作用 |
|---|---|---|
| 论文机制 | [DistServe v3 §§2.3、3、6](https://arxiv.org/html/2401.09670v3) | 将 prefill 干扰和 decode 交互性放入同一 SLO 约束下比较。固定 TTFT/TPOT 门槛再比较 goodput。论文的模型、硬件、流量与实现约束保留在引用中。 |
| 架构与调优 | [P/D 架构 · When to use P/D and how to tune it](https://llm-d.ai/docs/dev/architecture/disaggregation/pd-disaggregation) · [P/D 部署手册 · Overview、Verify、Observe](https://llm-d.ai/docs/well-lit-paths/foundations/pd-disaggregation) | 长输入与明显 prefill 干扰支持尝试分离。分离还要承担 KV 传输和两池配比。官网 NVIDIA vLLM 示例是 gpt-oss-120b、H200、1P TP1 + 1D TP4。它是验证拓扑示例。文档另给 5k/250、8 个 TP1 P 和 2 个 TP4 D 的配置例子，两者角色与总卡数不同。 |
| 路由与观测 | [KV Cache Management](https://llm-d.ai/docs/architecture/kv-management) · [Router Metrics · KV-Cache Index & Events](https://llm-d.ai/docs/dev/operations/observability/router-metrics) | 把估计或事件驱动的 prefix 命中、索引开销、P 与 D 排队和实际 KV transfer 分开观察。路由选中了 P/D 只是第一层。vLLM 指标确认 prefill 有请求且 decode external prefix hit 增加，才支持走过目标路径的判断。小量 smoke 请求不提供稳态证据。 |
| Benchmark 工具 | [llm-d-benchmark README · specs、harnesses、Benchmark Report](https://github.com/llm-d/llm-d-benchmark) · [Guide 索引](https://github.com/llm-d/llm-d/blob/main/guides/README.md) | 保存实际渲染的 manifests、部署预检、流量和结果。不同 load generator 的字段和口径先确认。比较同一 SLO 与失败定义下的结果。该文档是 main，功能需与实测 commit 对齐。 |
| 发布边界 | [v1.0.0 release](https://github.com/llm-d/llm-d/releases/tag/v1.0.0) · [发布说明](https://llm-d.ai/blog/llm-d-v1.0-state-of-the-art-inference-ready-for-day-2) | v1.0.0 记录 DisaggregatedSet 的角色联动和 revision 约束。发布声明本身没有验证目标部署的升级可用性。稳态吞吐、传输 fallback 和滚动期间可用性是三个实验。 |

**可迁移的实验。** 聚合与分离两组固定模型、精度、卡数、网络条件和负载生成器。先验证实际路由和 KV 传输。按输入输出长度、到达率、prefix 复用与并发分层，给出 TTFT、ITL/TPOT、达到 SLO 的吞吐、错误及排除样本。CPU/GPU 时间线要能对应同一请求与 step。把 NIXL/P-D 与 DeepInfer 的 GIN/GDAKI 作为不同机制分别记账。

**反例与未决。** 文档明确短输入或缺少高性能互联可能不适合 P/D。相同 tokens/s 也可能对应不同的尾延迟、失败率或资源成本。检查了架构、配置入口、指标与 v1.0.0 release。未复跑它们的 benchmark，未扫描全部 guide、issue 或 commit。下一步用本地案例验证是否保留了必要的比较条件。

## 评估方法 · 从 L15 的引用回查原案例

[Docent 作者案例](https://transluce.org/docent/blog/terminal-bench) 的步骤 1–3 与脚注。对同一任务配对成功和失败的轨迹，用差异提出假设，再查结构化 run metadata。剔除任何一侧超时的任务后结果反转。作者注明该筛选可能留下更容易的任务。L15 p.30 的分母是课件展示的 scored-run 口径。作者网页报告的是自己的复跑统计。保留各自分母与筛选条件，避免将两组数值混写成一次测量。[编排与监控讲义 pp.25–31](https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf#page=25)

vLLM 案例反向检查 [上游 issue #49097 的 R3–R5 与原始输出](https://github.com/vllm-project/vllm/issues/49097) 和 [当前生产指标](https://docs.vllm.ai/en/latest/usage/metrics/)。前者支持特定 scheduler 路径的首次差异。后者支持设计独立的服务观测，不证明历史版本或当前服务有同样的延迟损失。[vLLM 案例](../cases/02-vllm-scheduler.md)

## 算子评估 · 从 B300 反向查基准是否代表 workload

入口是 `GPU kernel benchmark correctness performance wall-clock addressable fraction` 和 `KernelEvolve long running job harness`。打开原始代码库、作者案例和同行实验。

- [KernelBench 仓库 · Overall Benchmark Metric](https://github.com/ScalingIntelligence/KernelBench) 用 `fast_p` 统计同时通过正确性且超过速度阈值的任务比例。比较时保留 PyTorch baseline、任务集合和失败样本的分母。这个比例不能给出目标 workload 的端到端收益。
- [KernelEvolve 论文](https://arxiv.org/abs/2512.23236) 与 [Meta 作者说明 · Searching for Optimal Kernels](https://engineering.fb.com/2026/04/02/developer-tools/kernelevolve-how-metas-ranking-engineer-agent-optimizes-ai-infrastructure/) 描述编译、正确性、计时、profiling、长运行恢复和搜索上下文。它说明当执行和验证足够成熟时，harness 可管理大量候选。作者报告的生产收益依赖具体 ranking workload 和芯片。
- [Kernel headroom v1 §§3、6、10](https://arxiv.org/html/2609.21058v1) 用独立验证检查正确性与速度，并测目标模型中可替换部分。不同 workload 的可优化比例差异很大。作者只测有限设备、形状和模型。应先核对自有 workload 的替换范围，再决定是否自动搜索。对照 A2 的 mutant 思路，可用数值容差过松或输出覆盖不足的负例检验本地 verifier。

这组来源改变了验收顺序。先对拍结果和范围，再对齐测量口径，再问目标 workload 中的关键路径能减少多少。现有案例没有同口径的多轮自动 kernel 搜索成本记录。

## 证据检索与压缩 · 反向查会丢掉哪些判断条件

从 A1 的工作记忆和 vLLM 的首次状态分歧出发。查询 `agent context evidence preservation qualifiers`、`coding agent context retrieval intermediate evaluation`。重点找能区分检索、记忆和判断的实验。

- [KEEP-Bench v1 §§3.2–3.3、4.1 与 Appendix A](https://arxiv.org/html/2610.26827v1) 分别构造并行证据、条件证据和少数证据。作者检查压缩文本能否保留每项必要事实，避免只凭最终答案判断是否丢证据。可迁移的测试是把 PDL 条件、测量层级或失败变体放进长记录，再单独检查摘要是否保留该条件。它是文本压缩评估，不是 Infra 工作流的实测。
- [ContextBench v1 §2.3、Table 2 与 §3.5](https://arxiv.org/html/2602.05892v1) 用经过核查的参考代码位置评估检索，另计修复结果。该表同一模型下不同 scaffold 的检索指标和最终修复率排名不同。作者也承认同一问题可能有多种有效补丁。给本地案例标参考证据时，允许其它足以支持结论的证据组合。只要求命中固定行会错罚有效路线。
- [Execution Instability 摘要](https://arxiv.org/abs/2608.06503) 发现候选。全文 HTML 当前访问失败，暂无正文核查。后续优先检验压缩边界前后是否改变工具参数或状态。 

## 人工投入 · 从 L19 讲师主题扩展

L19 当前课表未列讲义。以下为相邻研究，不作为该讲内容。查看讲师研究入口，查软件 Agent 的人工评估和任务用时实验。

- [Valerie Chen 等 · PULSE 论文主页摘要](https://proceedings.mlr.press/v306/chen26da.html) 将人类反馈与预测的满意度组合。已读 PMLR 摘要。PDF 抓取返回内容类型错误，细节留待核对。对三个案例，先用少量真实审阅记录测人需要纠正什么。是否值得拟合满意度，取决于样本和成本。
- [METR 2026-02-24 研究设计更新](https://metr.org/blog/2026-02-24-uplift-update/) 报告 2025 年初特定开发者样本的结果和随后实验的选择偏差。部分参与者会在 agent 执行期间做其它事，导致很难报告单个任务用时。用时比较应分别记录主动工作、等待、审阅返工和最终墙钟。历史研究无法给出你的预期节省比例。

## 工作流恢复 · 从 L15 展开 LangGraph 文档

从课件 pp.16–22 的 checkpoint 和 retry 开始，查看 LangGraph 文档的 Capabilities、Persistence 与 Fault tolerance。原 `durable-execution` 链接目前跳转到 Persistence，版本依实际安装确定。

- [Persistence · Checkpointers 和 Stores](https://docs.langchain.com/oss/python/langgraph/persistence) 区分按线程保存的运行状态与跨线程数据。内存 checkpointer 在进程退出后会丢失。L14 p.17 也将会话状态和工作区持久化分开。迁移到 Infra，应各自记录调研进度、仓库 commit、产物路径、实际作业 ID 和结果。
- [Fault tolerance · Retry boundaries and idempotency](https://docs.langchain.com/oss/python/langgraph/fault-tolerance#retry-boundaries-and-idempotency) 说明重试会重新调用整个节点。外部动作可能已成功，但状态还没写入。工作流必须先查已保存的执行标识与实际作业状态，再决定是否重试。该页面的节点 timeout 和 error handler 对框架版本有约束。

只有真实案例反复出现恢复与重复执行问题时，才把这层纳入首个产物。可用离线故障样本验证“结果已写入但最后一步超时”和“任务从未启动”会走不同分支。这个测试无需创建新 GPU 作业。
