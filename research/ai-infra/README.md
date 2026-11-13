# CMU 11-768 × AI Infra 调研地图

用课程里的知识、思考方式和实验方法，找出实际 AI Infra 工作中值得改变的判断和动作。先读 [工作机会比较与首项验证](OPPORTUNITIES.md)，可以看到已有做法、建议改变的动作、何时采用、怎样验证，以及证据何时会改变优先级。

## 按工作问题查案例和方法

| 工作问题 | 本地案例 | 课程方法和扩展来源 |
|---|---|---|
| 先优化哪个机制，在哪些请求上测？ | [DeepInfer workload 选择](cases/07-workload-choice.md) · [HW3 推测接受率](cases/08-speculation-acceptance.md) | [L5 规划](notes/harness-tools-planning.md) · [L20 搜索与反馈](notes/search-and-next-experiment.md) · [推理 workload、回放与现有调优闭环](sources/workload-decisions.md) |
| 怎样找到第一处状态分歧或关键依赖？ | [vLLM 调度](cases/02-vllm-scheduler.md) · [DeepInfer 组件 profiling](cases/03-deepinfer-profiling.md) | [工具与可回放执行](notes/harness-tools-planning.md) · [vLLM 源码与指标](sources/vllm-followup.md) · [Nsight 与 CUDA Graph](coverage/search-2026-11-13.md) |
| 什么观察应改变候选排序或下一轮实验？ | [TMEM hoist 与 BF16 编译](cases/05-kernel-search.md) · [B300 GEMM](cases/01-b300-gemm.md) · [MiniMax M3 MSA decode](cases/04-minimax-msa-pdl.md) | [L20 候选筛选](notes/search-and-next-experiment.md) · [A2 评估与反例](notes/evaluation-and-operations.md) · [NVIDIA GEMM、CUTLASS、KernelEvolve](sources/kernel-decisions.md) |
| 哪些失败应先重连、修预检或重采？ | [DeepInfer 云运行和上游交付](cases/06-framework-delivery.md) · [CS336 分布式训练](cases/09-distributed-training.md) | [L3 上下文与恢复](notes/harness-context-skills.md) · [L15 编排与监控](notes/evaluation-and-operations.md) · [长任务恢复与分工](sources/harness-followup.md) |
| 如何验证兼容性、性能和交付范围？ | [vLLM 回归](cases/02-vllm-scheduler.md) · [DeepInfer 交付](cases/06-framework-delivery.md) · [CS336 数值与训练计时](cases/09-distributed-training.md) | [A1 工具和约束](coverage/harness-reading.md) · [A2/Project 对照设计](coverage/evaluation-operations.md) · [A3 训练消融](coverage/training-reading.md) · [证据复核方案](DECISIONS.md) |
| 何时用索引、工具、工作流、Agent、harness、示范或 SFT？ | [依据实际错误选择产物](OPPORTUNITIES.md#从改进动作选择产物) | [检索与证据](notes/training-research-and-retrieval.md) · [检索反例](sources/retrieval-followup.md) · [SFT 数据](notes/training-sft-data.md) · [RL 证据与系统](notes/training-rl-evidence-and-systems.md) · [人机协作与多 Agent](notes/people-coordination.md) |

## 资料入口与续做

- [课程来源与覆盖](sources/README.md) 从讲次、课件、录像、作业和 Project 进入。引用索引保留来源位置、版本和实际阅读状态。
- [按主题的扩展阅读](sources/exploration.md) 记录双向检索的入口和影响。最新扩展见 [L20 与评估边界](coverage/search-2026-11-13.md)、[算子决策](sources/kernel-decisions.md)、[工作负载](sources/workload-decisions.md) 及 [训练系统原始资料](sources/training-followup.md)。
- [进度与验收](PROGRESS.md) 记录已核对范围、仍可能改变排序的证据、录像与阅读缺口。案例区分历史实测、公开作者报告和待验证假设。

- [全部登记来源精读](deep-reading/README.md) 按原索引逐项更新，含已读正文范围、工程启发和待复核项。
