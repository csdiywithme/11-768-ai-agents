# 独立 workload 证据：CMU 11-763 HW3 的推测解码

这个案例来自另一门课程和独立运行批次。目标是在单卡离线生成中判断更小的 drafter 和 RMSNorm 融合是否值得保留。它与 DeepInfer 四卡测量的模型、请求和引擎均不同。由于本案例用于选择研究方向，也要从未来留出集中排除。

在 H100 80GB 上使用 Qwen3-8B target 和 Qwen3-0.6B draft，BF16、SDPA、batch=1、固定生成 50 个 token。同一轮内用相同的输入和 repeat seed 比较 AR、SD γ=2 和 γ=3。复算 63 条无 profiler 基线记录，三条 prompt 的合计 AR/SD 时间比为 0.848× 和 0.831×。最短样本是 1.043× 和 1.092×，接受率均为 100%。另两条提示仍减速。长度和内容同时变化，因此不能从三条数据单独归因给长度或接受率。原项目另有较大矩阵与 B200 复测，两者都有自己的基线。 [基准与负载分层](/Users/simida/csdiy/cs11-763/lminference-fall2025-code/homework3/profiling/RESULTS-06B.zh-CN.md:19) · [63 条基线及另存的 trace 记录](/Users/simida/csdiy/cs11-763/lminference-fall2025-code/homework3/results/profile-06b-h100-20260924/profile/records.jsonl) · [独立复测范围](/Users/simida/csdiy/cs11-763/lminference-fall2025-code/homework3/REPRODUCE.md:11)

后来用同卡 A/B 比较 RMSNorm 融合。84 次正式测量完成。42 对原版与候选中，0 对通过全部同工作量条件。21 对 SD 中只有 11 对生成 token 一致。summary 为 `invalid`、`failure=null`，wrapper 仍记 `failed`。按协议应淘汰该候选的性能结论，并决定是否另做数值定位。根据 wrapper 结果重跑同一配置，或比较已改变输出的 wall time，都无法回答原问题。 [验收与解释](/Users/simida/csdiy/cs11-763/lminference-fall2025-code/homework3/profiling/RESULTS-06B.zh-CN.md:59) · [原始 A/B 汇总](/Users/simida/csdiy/cs11-763/lminference-fall2025-code/homework3/results/profile-rmsnorm-ab-06b-h100-20260924/ab/summary.json)

如果下一项工作的目标是选择推测配置，先用公开或已获许可的请求在同样上下文与并发内分层，再比较 draft、verify、有效 token 和无 profiler 的时延。若目标是融合候选，先用当前协议中的小输入和 teacher-forced 检查作为初筛，再做完整输出和配对计时。两条路径都可先用保存记录测试决策逻辑。此案例支持记录不同内容的 acceptance 和工作量门槛。它没有提供线上吞吐、队列或跨引擎收益。
