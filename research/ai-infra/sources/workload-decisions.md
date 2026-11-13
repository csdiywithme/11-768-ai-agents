# 按工作负载选择推理优化

核验日期 2026-11-13。问题来自 [DeepInfer 本地历史运行](../cases/07-workload-choice.md)。真实权重能复现运行路径，固定提示的接受率却几乎恒定。沿原论文到作者代码，再沿 NVIDIA 和 llm-d 官方文档到回放与优化工作流，核查哪些材料能帮助选下一项实验。

## 动态验证的收益取决于在哪个负载区间测

- **相同 drafter 的验证策略对照。** DeepInfer 作者在 Open-PerfectBlend 上用 EP64/DP64 benchmark 数据估算 throughput。固定验证 5 个 draft token 与动态长度相比，16 sequences/rank 的估算变化为 −1.5%，60 为 +5.4%，96 为 +11.0%。在 60 sequences/rank，平均 verified length 从 5.00 降至 3.58，表中 accepted length 从 3.30 降至 3.21。这个对照适合决定先查并发还是先查接受率。它无法给出 B300 四卡、EP4/DP4、nextn=3 的收益。 [DeepInfer 部署问答 · Speculative decoding](https://github.com/deepseek-ai/DeepInfer/blob/main/doc/inference_deployment_qa.md#speculative-decoding)
- **线上对照改了整套推测路径。** DSpark 论文 §5.4 比较 DeepSeek-V4 preview 生产引擎中的 DSpark-5 和历史 MTP-1。V4-Flash 在匹配的系统吞吐下，每用户生成速度提高 60%–85%。论文也报告了更严格 SLA 的大倍率，原因包括基线已接近可运行边界。它提供系统吞吐和用户生成速度联合评估的实例。该对照同时换了 drafter 和调度路径，不能归出单项 scheduler 收益。§5.4 的限制也指出，缩短目标验证只能节省 verify 工作，固定 draft block 的生成成本仍在。 [DSpark v1 §5](https://arxiv.org/html/2607.05147v1#S5)
- **用相同决策输入比较接受行为。** TraceSpec v1 §3.2.4 用保存的相同输入比较 drafters。Qwen3-8B、T=0 的 DSpark 每轮 accepted tokens（含 correction 或 bonus）在小于 2K 到 16–32K 的区间间，网页调研从 5.242 降至 3.442，软件工程从 3.073 升至 4.321。论文同节的在线总体趋势和任务配对结果也有差异。可迁移的实验设计是先按任务类型和阶段配对，再在各层比较上下文或验证策略。该指标包含额外 token，不能直接和本地 accepted draft / drafted 比。 [TraceSpec v1 §3.2](https://arxiv.org/html/2610.41177v1#S3.SS2)
- **从论文跟到可复现代码。** DSpark 原文链接的 DeepSpec README 给出按匹配 target、draft checkpoint 跑任务的 acceptance 评估，包含数学、代码及对话。它可用于借鉴如何构造 draft 任务集。README 的这组 checkpoint 对应论文离线 Table 1，不能复用其结果来证明 DeepInfer serving 的 latency、queue 或 topology 收益。 [DeepSpec · Evaluation](https://github.com/deepseek-ai/DeepSpec#evaluation)

## 回放工具保留了哪些负载条件

- **从 Dynamo 的回放文档继续跟到 AIPerf。** Dynamo dev 默认捕捉到达时间、序列长度、prefix sharing 和 session 依赖，无原始 prompt/response，默认 live replay 生成合成请求。它适合比较路由、队列与 cache 行为。AIPerf 的 Mooncake loader 另支持直接传预格式化的 `messages`。对受内容影响的 draft 接受率，优先用可访问的原文、可回放 token 或公开固定提示。只有长度与 hash 时，接受率仍是一项需要实测的假设。对 closed-loop 的 agent 行为，也需另看生成内容是否进入下一轮、依赖是否与固定回放一致。 [Dynamo dev · What This Captures](https://docs.nvidia.com/dynamo/dev/agents/agent-simulation#what-this-captures) · [AIPerf · Pre-formatted Messages](https://github.com/ai-dynamo/aiperf/blob/main/docs/benchmark-modes/trace-replay.md#using-pre-formatted-messages)
- **llm-d 的性能评估说明了现成的可重复工作。** 官方文档用版本化 workload profile 区分 guide 的负载阶梯和 sanity test，性能 harness 输出 TTFT、ITL、TPOT、端到端时延、吞吐，server metrics 需要相应采集。它可做服务对照的流程基线。接受率分层与 draft、verify、Engram readiness 的关联仍需要目标引擎观测。 [llm-d · Performance assessment](https://llm-d.ai/docs/operations/validation/benchmark#available-workload-profiles)

## 已有优化工作流应进入基线比较

从 NVIDIA Dynamo 的 Performance Optimization Skills 页面跟到 v1.6.1 的 Optimize Loop 和知识查询规则。已有实现包括冻结目标工作负载与基线、依证据选择和审查候选、按假设更换测量系列、按噪声和预算停止。知识查询的规则还要求在可行的大类没有筛过时先比较大类，在近邻调参落入噪声时回到探索。它主要面向 Dynamo 配置与 AIPerf，对 DeepInfer 的 Engram 源码和 trace 仍需核实接入成本。该仓库明确提醒，合成内容测得的 speculative acceptance 应回到生产遥测确认。 [Dynamo · Performance Optimization Skills](https://docs.nvidia.com/dynamo/agent-skills/optimization) · [Dynamo v1.6.1 · Optimize Loop](https://github.com/ai-dynamo/dynamo/blob/v1.6.1/agent-docs/guides/optimization/optimize-loop.md) · [Dynamo v1.6.1 · Consult Performance Knowledge](https://raw.githubusercontent.com/ai-dynamo/dynamo/v1.6.1/.agents/skills/consult-perf-knowledge/SKILL.md)

在本例里，先把当前 workload 条件、候选和下一次判别性观测交给上述闭环的等价手工流程。再比较三种增量。直接把每步 draft、verify、acceptance、Engram 依赖抽取成工具。把代表性请求、乱序复测和输出校验接入可重放的工作流。让能读取源码和 trace 的助手在证据改变时提出下一项候选。对照同一预算下有无改变优化排序、减少无效运行或降低人工成本。该比较仍是设计，当前没有执行结果。
