# SFT 数据先解决什么问题

**与 Infra 的关系**。把可核验的操作和判断留在数据中，可以同时用于评估、提示示范和后续训练。是否要把行为写进权重，取决于失败能否在固定工具、来源和验收规则下重复。当前三个历史案例适合检验标注能否成立，暂无法用它们判断训练收益。

## 记录什么

- 原始轨迹按系统指令、用户目标、模型动作、工具观察和终止状态保存。将版本、来源、前置条件和验证结果附在轨迹上。保留原始轨迹，再为目标模型与 harness 渲染副本。ADP 将异构数据转换成结构化动作和观察，再转成目标 harness 可用的表示。论文 §3.2–3.3 说明转换过程。转换减少重复写适配器的工作，仍须核验目标工具是否能执行转换后的动作。[L8 讲义 p.3–5、p.31–34](https://www.cmu-agents.com/slides/lecture-08-sft.pdf#page=31) · [ADP v1 §3.2–3.3](https://arxiv.org/html/2510.24702v1#S3)
- 教学示例默认只对模型应生成的 assistant token 计 loss。工具输出和用户输入是条件上下文。区分模型生成的停止 token 与运行时插入的角色标记。推理段是否作为上下文、是否计 loss，是两个选择。先核对渲染后的 token、停止位置与 mask。再按动作类型核对实际计 loss 的 token 数。按样本平衡的数据，经过渲染后可能被长日志分析或长说明主导。[L8 讲义 p.11–19](https://www.cmu-agents.com/slides/lecture-08-sft.pdf#page=11) · [L8 录像 10:35–19:06](https://www.youtube.com/watch?v=O3HSU0AoILc&t=635s)
- 将成功结果与可模仿动作分开判断。测试通过的修复轨迹可能含无效的编辑—测试循环或多余 debug 输出。记录有信息增益的失败及其纠正过程。区分「失败观察之后的好动作」和「前面造成失败的动作」。同一问题的完整正确路径、异常后的纠正路径和错误输出样本都可保存。前两者可用作经核验示范，错误输出更适合检验评估器或作偏好数据，不能直接作为正例提高其概率。[L8 讲义 p.21–29](https://www.cmu-agents.com/slides/lecture-08-sft.pdf#page=21) · [L9 讲义 p.3–7](https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf#page=3)

## 阅读原文后保留的限制

OpenThoughts-Agent 在其 SFT 流程内改变任务来源、teacher 和筛选。强 teacher 的直接成绩未准确预测其学生成绩。固定任务描述上追加轨迹的结果在论文设置内出现平台期。其 §4 所谓「新任务」扩展使用原 Tezos 问题的合成改写和不同的重采样权重，未引入新的底层问题。可以用它支持「扩展描述比在同一描述下继续采样更有价值」的假设，不能把改写视为独立问题，也不能据此规定 Infra 示范必须达到某个回合数。回合数与操作成本相关，但短路径可能是最准确的示范。[OpenThoughts-Agent v1 §3.2–3.6、§4](https://arxiv.org/html/2606.24855v1#S3) · [L8 讲义 p.23–28](https://www.cmu-agents.com/slides/lecture-08-sft.pdf#page=23)

老师示范中的每步历史来自录制轨迹，部署中的历史来自模型此前的动作。低训练 loss 无法证明模型能在新观察下恢复。DAgger 的原始结果针对顺序预测的分布变化。迁移到 Infra 时，可以先回放已有的反常观察，让评审人员写下一步建议，检查标签是否一致。这个步骤尚未构成 DAgger 训练。最终评估要让模型在目标工具接口下做任务。一个任务的多个轨迹和改写应放在同一分组，跨版本或跨仓库泛化的结论需要相应留出。[L8 讲义 p.39、p.42–47](https://www.cmu-agents.com/slides/lecture-08-sft.pdf#page=42) · [DAgger 原文 §1–2](https://proceedings.mlr.press/v15/ross11a/ross11a.pdf#page=1) · [L8 录像 49:35–54:23](https://www.youtube.com/watch?v=O3HSU0AoILc&t=2975s)

## 三个可验证的迁移假设

| 案例 | 可标注的判断 | 验证怎样区分方案 |
|---|---|---|
| B300 FP4 GEMM | 识别同 GPU allocation 中用 event 得到的时间。识别 NCU replay 用于解释的计数与当前测量条件。 | 先给模型相同的测量数据。比较普通提示、带字段检查的提示和经核验的示范。若字段检查已消除误比，脚本能解决该错误。若口径齐全仍误读，进一步核验示范是否可稳定标注。[NCU 报告](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/experiments/m4_lowprec_20261022/NCU_ANALYSIS.md:3) |
| vLLM 调度 | 给定固定版本和两条回放，标出首次状态分歧。列出能在真实运行中检验它的最小断言。 | 同一个断言可以先核查离线状态差异。训练与最终验证按 bug 或版本分组。以一个教学回放改写多份数据，仍只有一个底层任务。[离线回放](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:15) |
| DeepInfer block table | 区分组件时间与未被其他计算覆盖的关键路径时间。给出验证两个解释所需的 CPU/GPU 依赖事件。 | 对同一输入分别加入真实依赖证据或只提供组件表。比较模型能否在证据不足时保留条件并提出具体观测。把端到端加速误写成事实的结果可作为评估反例。[组件结果](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:15) |

如果评审者无法就「哪条结论越过证据」达成一致，先修评估规则。若错误能由字段约束或只读预检稳定消除，先衡量其成本。只有数据中存在可核验且重复出现的判断行为，并且在新任务上可以独立评估，SFT 才有可检验的目标。当前笔记提供待验证假设，未生成训练集或测量模型表现。
