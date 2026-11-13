# RL 训练和系统优化要保留哪些证据

L9 用 Number Search 区分示范动作的似然与任务完成概率。同一任务允许多条正确路径。L11 把反馈设计、数据采样与更新稳定性分开。L12 再将采样、环境验证、训练和权重同步落到系统组件。A3 是按这些组件组织的作业。当前本地版本的训练核心仍含 `TODO(student)`，以下描述作业契约与所给实现。 [L9 p.2–7](https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf#page=2) · [L11 p.3–8](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf#page=3) · [A3 源码分工](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:82)

## 先核验奖励

- **任务目标、验证器与轨迹诊断分开。** 同一个任务的组内奖励中心化后，全部通过和全部失败都可能产生零 advantage。后者有至少三种解释。正确结果被验证器拒绝、模型确实不会、模型重复尝试同一种方法。先对照规格与实际断言，再用人工正确例或较强模型的示范检查可解性，最后检查动作多样性。修改采样或奖励后要保留独立评估，否则分数改善无法排除规则变化。 [L9 p.37–42](https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf#page=37) · [L11 p.37–47](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf#page=37)
- **一次成功给不出每步的因果贡献。** 奖励到最后才出现时，轨迹内的动作共享终局信号。学习 value 的 critic、GAE 或单步标注可以提供更细的判断。它们分别增加模型、估计误差和人工标签成本。用「命令数」「引用数」或「更短耗时」作奖励前，要写一个可以拿高分却未完成任务的反例。例如 GEMM 更短的计时可能来自少做了工作。核对输出正确性和测量口径，再判速度。 [L11 p.9–17、p.41–47](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf#page=9)
- **区分算法名称、优势规则与实际 loss。** A3 的 `train.algo` 和 `train.loss_fn` 分开。GRPO 变体使用组内均值中心化，省略 std 归一化和 reference KL。代码将归一尺度施加于 batch 的有效动作 token。它不等同于原论文使用固定生成预算作除数的完整公式。REINFORCE 的 G=1 组同时改变优势规则和 loss，只能比较配置。G=2 与 G=8 更接近组大小的对照。 [L9 p.38–42](https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf#page=38) · [Dr. GRPO v1 §3.1–3.2](https://arxiv.org/html/2503.20783v1#S3) · [A3 优势与 loss](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:177) · [A3 batch 归一](/Users/simida/csdiy/11-768-ai-agents/assignment-3/scottyrl/tinker_adapter.py:342) · [A3 消融](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:279)

## 明确计算阶段和重叠条件

训练侧有 prefill 式并行计算与梯度。采样侧是 decode 密集的逐 token 生成，还要等工具和环境。推理 prefix cache 复用历史 KV。训练 prefix sharing 合并前向和反向的重复前缀，需要隔离不同分支的 attention。权重同步会受 trainer 和 inference 的分片布局影响。看到「更快训练」时，应先查时间的分母是 actor update、采样阶段、整步，还是达到固定 held-out 成绩的总时间。讲义里 prefix sharing 的倍数分别来自 ZoRRo 与 AReaL-DTA 的特定实验。不能直接移作用户 workload 的收益。 [L12 p.3–17](https://www.cmu-agents.com/slides/lecture-12-rl-systems.pdf#page=3)

A3 的同步组由最新 snapshot 采样，异步组让采样与更新重叠。异步方案要求同一组绑定同一个策略和版本。learner 消费时判断陈旧度。有界队列提供背压。后台评估同样固定 snapshot 版本。worker 出错或取消后需让其他任务退出。Starter 的 fake 测试覆盖陈旧出队计数、零有效序列时不更新、后台评估与异常传播。它们证明设计有可离线检查的契约，当前没有运行这些测试，也没有验证 GPU 服务。 [A3 异步要求](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:249) · [A3 契约测试](/Users/simida/csdiy/11-768-ai-agents/assignment-3/tests/test_contracts.py:265)

AReaL 论文 v1 把行为策略与近端策略分开。前者用于校正采样分布，后者用于限制更新幅度。它还约束采样进度超过 learner 版本的距离。A3 是消费时丢弃陈旧组的较小方案，配合 token 级比率和 clipping。两者不能按同一异步算法汇报。AReaL 的系统对照含特定模型、节点数、训练步数与基线。课程 p2b/p3 的同步与异步比较也改变了 GPU 布局。应同时读同一质量目标下的墙钟、被丢弃组数、陈旧度与 held-out 表现。均值接近 1 的 importance ratio 仍可能有很大尾部。单 token 级校正也没有恢复由过去动作造成的全部历史分布差异。 [AReaL v1 §3.2、§4–5、§7.2](https://arxiv.org/html/2505.24298v1#S5) · [L11 p.19–35](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf#page=19) · [A3 部署拓扑](/Users/simida/csdiy/11-768-ai-agents/assignment-3/skyrl_tinker_modal/README.md:3) · [A3 时间与策略漂移诊断](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:142)

## 保存原始轨迹，并单独决定怎样喂给模型

Tinker 文档的 sequence extension 对比 token 序列。下一轮 prompt 以本轮 prompt 加 completion 为完整前缀时，可以拼接。上下文压缩、聊天模板改写旧 thinking 或停止 token 不一致会打断前缀。这时从新序列开始，保留模型当时实际看到的上下文。A3 的 loss mask 在 next-token 目标上对齐，长度为 n−1。仅 completion token 有采样 logprob 与 advantage。文档的概念例子以每个内容 token 标示 mask，不能直接复制成 A3 的数组下标。 [Tinker Tutorial 404](https://tinker-docs.thinkingmachines.ai/tutorials/advanced/sequence-extension/#how-promptcompletion-boundaries-shift) · [L12 p.19–24](https://www.cmu-agents.com/slides/lecture-12-rl-systems.pdf#page=19) · [A3 next-token 对齐](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:211) · [A3 改写历史的测试](/Users/simida/csdiy/11-768-ai-agents/assignment-3/tests/test_envs.py:139)

Miles 团队给出了更接近生产系统的例子。它把 rollout 中实际消费与采样的 token ID 保存下来，避免文本解码后重新编码改变 ID。比较「运行时 token 与训练输入是否精确一致」和「完整模板重新渲染后结构及文字是否一致」是两项不同检查。后者可以容忍中间文本段等价的编码差异，不能据此替换已记录的 token ID。团队 2026-08-18 的更新也说明，压缩上下文和 sub-agent 可以让一次任务留下多个可合并片段。该机制针对其实现和已测配置，尚未核验用户仓库中的推理运行时。 [Miles 团队的 TITO 说明](https://www.lmsys.org/blog/2026-05-13-no-token-left-behind/)

对应到用户的历史案例，可以先分开三份记录。第一份保存实际工具调用与结果，第二份保存给模型的压缩上下文，第三份保存实验状态和验收。用 vLLM 调度回放检查长上下文被压缩后是否保留首次分歧、版本和真实测试边界。用 DeepInfer 比较 CPU、H2D、GPU 事件时，要保留依赖与时间轴，才能判断是否存在可重叠开销。把这些输入喂给现有助手与结构化工作流即可开始对照。只有当独立评估能稳定辨认行为好坏，且收集新轨迹的成本和信号仍可控，才有条件比较训练的额外价值。 [vLLM 离线回放](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:15) · [DeepInfer 组件时间边界](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:15)
