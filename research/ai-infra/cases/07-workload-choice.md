# DeepInfer：选哪条 decode 优化线，下一组 workload 怎么定

这个案例接在已有的 [DeepInfer block-table 组件 profiling](03-deepinfer-profiling.md) 之后。组件数据回答构表、传输、展开各自花多久。本例转到真实权重，比较同一条 decode 路径上可以采取的下一步。需要区分三个决策：先让哪条部署路径跑起来，选择哪个优化候选，再用哪些请求测它能否改善目标服务。

## 还原已有决策与观测

| 当时的动作 | 已观察到的结果 | 对下一步的影响 |
|---|---|---|
| 原 GIN/GDAKI device communicator 在先前的 B300 探针中初始化失败。转用社区 PR #246 的可选 LocalEngram 路径，在一个 NVLink 节点加载完整模型。 | 本地 4×B300 短文本 smoke 成功。PR 作者 wukehu 的 4×B200、EP4/DP4/CP1、128K、64 decode requests、nextn=3 对照给出约 2.01× prefill 和 1.27× decode tokens/s/GPU。PR 估计 V4.1 每 rank 的 Engram shard 约 47.21 GiB。 | 单节点 profiling 有可行基线。若目标是多节点或 HBM 紧张部署，原 GIN 的复现和拓扑支持仍是前置。不同路径的性能和可运行性分别判断。 [原 GDAKI 探针日志](/Users/simida/my/projects/deepinfer_optimize/profiling/results/engram-current-sdk-b300/gdaki-full.log:750) · [社区 PR #246 · Scope、Controlled validation](https://github.com/deepseek-ai/DeepInfer/pull/246) · [本地 smoke 结果](/Users/simida/my/projects/deepinfer_optimize/profiling/results/local-engram-246-model/model-smoke-result.json:1) |
| 在同一热引擎做 8K×1、128K×1、8K×8（每 rank 请求数）的 clean/trace 对照。 | 真实权重 DeepSeek-V4.1-Flash-1017，PR 提交 `0a9059c`，模型 revision `1bae0e1`，EP4/DP4/CP1。请求由同一可重复提示构造，温度 0、nextn=3、固定长度验证，每请求 256 个 token。8K×1 的 rank 0 在 32 个测量 step 中发出 128 token。该请求全输出记录为 accepted draft 192 / drafted 195；读回各组记录中其它请求的同一分母也相同。 | 这是同模型下检查 trace、输出及结构性成本的窄基准。上下文长度和并发变了，但内容高度可预测，几乎没有 acceptance 对照。顺序固定，后测形状也继承 allocator、JIT 和 graph 历史。 [工作负载定义](/Users/simida/my/projects/deepinfer_optimize/profiling/local_engram_246_model_profile.py:38) · [采样与计数代码](/Users/simida/my/projects/deepinfer_optimize/profiling/local_engram_246_model_profile.py:166) · [固定长度 guard](/Users/simida/my/projects/deepinfer_optimize/profiling/local_engram_246_model_profile.py:315) · [8K×1 的 clean 记录](/Users/simida/my/projects/deepinfer_optimize/profiling/results/local-engram-246-model/followup-sweep/profile-8192-1/artifacts/clean/rank-0.json:1) · [sweep 完成记录](/Users/simida/my/projects/deepinfer_optimize/profiling/results/local-engram-246-model/followup-sweep/result.json:1) |
| 针对表读取试两种干预。用 fused gather 合并读取路径，用分层 prefetch 争取把工作移到消费前。组合成 base、fused、prefetch、两者组合，另做 base_repeat。 | 两种 8K 请求数都通过本轮逐 rank 的输出一致性检查。按每步四 rank 最大值再取中位数，fused 相对 base 的 step-start interval 下降约 214.23 µs（1 请求/rank）和 190.51 µs（8 请求/rank）。prefetch 单独下降约 61.64 和 17.31 µs，叠到 fused 后，中位数反而分别增加约 28.32 和 30.23 µs。8 请求/rank 的尾部读数方向不同，组合的 p95 比 fused 再低 28.82 µs，但同一轮 base_repeat p95 自身就下降 150.74 µs。 | 在这个范围内，下一次验证优先保留 fused 单改动。prefetch 可能有收益，也可能移走原有并行或引入争用。先做重排复测和依赖观测。当前结果为单次运行内描述性比较，p95 只来自 31 个步间隔。 [四种实现](/Users/simida/my/projects/deepinfer_optimize/profiling/local_engram_246_variants.py:22) · [clean 对照与分母](/Users/simida/my/projects/deepinfer_optimize/profiling/results/local-engram-246-model/engram-ablation/clean-analysis.json:1545) · [1 请求/rank 的组合对照](/Users/simida/my/projects/deepinfer_optimize/profiling/results/local-engram-246-model/engram-ablation/clean-analysis.json:504) · [8 请求/rank 的组合与重复基线](/Users/simida/my/projects/deepinfer_optimize/profiling/results/local-engram-246-model/engram-ablation/clean-analysis.json:1146) |

PR 作者的 B200 结果、本地项目的 B300 真实权重运行和实验性 fused/prefetch 各有独立证据。PR #246 属于社区作者。上述路径和原始结果已读取核对，本轮未改源码或启动实验。

## 哪条新观测会改变下一步

1. **先确定目标是单节点可运行、交互延迟还是吞吐。** 单节点 HBM 仍足够、目标是改善当前交互路径时，保留同一 LocalEngram 后端。目标变成跨节点或 HBM 预算不足时，优先验证目标通信路径和容量。把四卡本地 decode 直接扩大成线上部署会同时改变成本、并发和瓶颈。
2. **Engram 关键路径受限与推测效率受限需要不同试验。** 先复用现有数据和已获许可的代表性请求；若只有这批可重复合成输入，就把它留作回归用例。再挑与实际工作对应的高/低接受请求，在固定上下文和并发下分组。比较 stock 与 fused 的每个输出 token 成本。如果两个分组的 fused 收益都能在换序复测中重现，且依赖证据显示 Engram 消费点提前，才扩大范围。若主要差距随接受数和排队改变，转去比较固定与 confidence-scheduled verification，先保持 fetch 路径不变。
3. **将干预和读数预先配对。** 首批先比较两个能决定排序的切片，比如当前交互段与一个低接受段。固定 model revision、精度、硬件、EP/DP、上下文、输出长度、并发与 sampling。保持相同初始状态或交错运行，单独记录 warm/cold。每步保留 proposed、实际验证、accepted draft 和最终 emitted tokens，分离 draft、verify、queue 与 Engram readiness。先检查 greedy 输出或相应采样正确性，再比较无 profiler 的 token 时延。若部署问题涉及混合负载，再用同一到达序列比较 TTFT、TPOT/ITL、有效吞吐和尾部回归。trace 用于归因。判据取决于目标 workload 与基线漂移，由实验前确定。
4. **判断可复用产物。** 如果主要困难是得到每请求的 draft、verify 和 Engram 依赖证据，优先考虑 trace 提取工具。若每次都要重做请求分层、配对、正确性和清洁计时，固定 benchmark 工作流有直接可验证的作用。只有中间结果常改变下一项观测、且规则无法稳定表达时，才值得比较条件式编排与开放式 Agent。评估时考查是否切换到能区分竞争解释的实验，及一次调查中的无效运行数与人审成本。

以上是基于历史结果的下一步选择。当前证据支持把 fused 保留为复测候选，尚无代表性请求、独立重复和线上验证来确认该排序。课程 L5 的反馈重规划、A2 的压力测试和 Project 的对照设计，都可以用来检验这一次改选。 [规划与验证专题](../notes/harness-tools-planning.md) · [评估专题](../notes/evaluation-and-operations.md) · [DSpark 原文 §3、§5](https://arxiv.org/html/2607.05147v1#S5)

外部对照显示，作者在不同并发处估算的动态验证收益方向有变化，且同长度的不同任务可以有相反的接受率趋势。NVIDIA Dynamo 已有负载约定、候选选择和反证的实现，可作工作流基线。具体条件和适用边界见 [按工作负载选择推理优化](../sources/workload-decisions.md)。
