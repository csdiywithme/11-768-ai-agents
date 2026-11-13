# L12 与 A3 · 推理调度、训练流水线与容量

本批读取机制、对照和限制，未复现实验。课程引文未固定 arXiv 版本时，记录本次实际阅读版本。先用已有案例中的状态、计时和日志做区分性检查，再考虑改调度、训练或工具。

### `https://www.usenix.org/conference/osdi22/presentation/yu`

- 原文：[Orca，OSDI 2022 正式版](https://www.usenix.org/system/files/osdi22-yu.pdf)
- 范围：§§3–4 的 iteration-level scheduling、selective batching 和内存，§6 的实验与 Fig. 9–10。
- 方法与结果：调度器每轮重新组成 batch，完成者可以离开，新请求可进入。非 attention 算子可以把不同请求 token 合并，attention 仍需使用各自历史。在作者 175B trace 上，匹配 190 ms 中位归一时延时，FasterTransformer 为 0.185 req/s，Orca 为 6.81 req/s。低负载下没有足够请求形成 batch，结果更依赖单次执行。这是 2022 年指定引擎与 trace 的比较。
- 工程用法：看到 TTFT 或有效吞吐变差时，用相同到达序列先比可进入每轮的请求数、最早出队时间、长度分布和剩余 KV。若 batch 中还有空位而队列中无可进入请求，先查负载供给。若有合格请求但未被访问，定位调度状态或配额。若调度状态一致而耗时差随单步设备执行改变，再移到 kernel 分析。对照须同时保留 TTFT、TPOT 和目标 SLO。

### `arxiv:2211.17192`

- 原文：[Fast Inference from Transformers via Speculative Decoding v2](https://arxiv.org/html/2211.17192v2)
- 范围：§§2.2–2.3、3.1–3.5、4.1–4.2，附录 A.1 的分布证明。
- 方法与结果：draft 从 q 采样，target 以 min(1,p/q) 接受，拒绝时从修正分布补样。固定 draft 长度 γ 时，独立同分布近似下，每轮期望输出为 1+α+…+α^γ。其时延模型还假设并行 verify 不增加 target 调用墙钟。draft 代价 c 依赖实现和硬件。论文实测实现是 T5-XXL 翻译、摘要，对 decoder-only 模型另做接受率观察。
- 工程用法：先把 11-763 与 DeepInfer 的 proposed、accepted draft、correction/bonus、emitted、draft/verify 墙钟和请求长度分开。若延长 draft 提高每步有效 token，却因验证成本或队列恶化增加每输出 token 时间，缩短 draft 或分并发选策略。应保留固定 workload 和无 profiler 的 AR/SD 对照。论文的输出分布等价不能替代现有代码的采样正确性测试。 [本地接受率案例](../cases/08-speculation-acceptance.md)

### `arxiv:1604.06174`

- 原文：[Training Deep Nets with Sublinear Memory Cost v2](https://arxiv.org/html/1604.06174v2)
- 范围：§§3、4.1–4.4 的图内存复用与重算规划，§5 ResNet/LSTM 的显存与运行时间。
- 方法与结果：先借依赖和生命周期复用缓冲区，再挑中间激活只在需要时重算。近似均匀的 n 层链，分 k 段时，段内峰值和段间保留量约为 O(n/k)+O(k)。在其 ResNet/LSTM 环境报告更低显存及额外运行时间。GPU 测量来自 Titan X 等历史设备和实现。
- 工程用法：CS336 若 OOM，先区分 checkpoint 改变的是活跃激活、allocator reserved、还是其他权重/优化器/通信占用。按最大活跃段的 bytes 与重算耗时选边界，固定 dtype、shape、microbatch、状态和正确性。若峰值仍在 checkpoint 无法覆盖的 optimizer 或 collective 阶段，就转向对应阶段。 [CS336 证据范围](../cases/09-distributed-training.md)

### `arxiv:2205.05198`

- 原文：[Reducing Activation Recomputation in Large Transformer Models](https://arxiv.org/html/2205.05198)
- 范围：正文 §§4.2、5、6.1–6.3、附录 C。读 Fig. 7 和 Tables 3–5 的对照口径。
- 方法与结果：sequence parallelism 将 TP 中重复保留的部分激活分散到序列维，selective recomputation 则优先丢弃占内存较大而重算较便宜的中间结果。两项结合在其混合精度 A100 80GB、固定序列 2048 的模型组中，相对全重算且无 SP 的对照，整步吞吐增加 29.0%–32.1%。约 5 倍显存下降针对论文的激活量基线。该实验没有数据并行。
- 工程用法：建立 SP 开关与 selective/full 重算的可运行格点，固定训练工作量和 dtype。一次运行同时报告 model FLOPs、实际算子量、激活峰值、collective 和整步耗时。若重算已被 fused attention 规避，先看同模型 trace 中还能省哪些活跃激活。论文的基线百分比不能转成对任意 FlashAttention 或 FSDP 配置的收益。

### `https://docs.nvidia.com/nemo-framework/user-guide/25.02/nemotoolkit/features/parallelisms.html`

- 原文：[NeMo Framework 25.02 · Parallelisms](https://docs.nvidia.com/nemo-framework/user-guide/25.02/nemotoolkit/features/parallelisms.html)
- 范围：指定归档版本的 TP、PP、SP、CP 与 distributed optimizer 段落，核对可配置范围和示例。
- 方法与限制：TP 切层内张量。SP 与 TP 配合，分散部分原本重复的序列激活。CP 在序列维分散模型的序列激活，attention 需要共享相关上下文。PP 在层间安排 microbatch，存在空泡。此页给拓扑和配置入口，未提供用户当前模型的性能排序。
- 工程用法：先记录 oom 前每卡的参数、optimizer、活跃激活和 KV/collective；再选一个能减少首要占用的轴。比较同设备、global batch、seq、dtype 和数据步骤的峰值、链路区间及整步。若只看到 per-rank 内存下降且通信使整步更慢，应按容量目标决定是否接受该代价。

### `https://docs.vllm.ai/en/latest/design/prefix_caching/`

- 原文：[vLLM V1 Automatic Prefix Caching](https://docs.vllm.ai/en/latest/design/prefix_caching/)
- 范围：Hash-based KV cache、缓存隔离、数据结构、allocate/append/free/evict 和结尾实例，2026-11-13 的 latest 页面。
- 方法与限制：缓存完整 block。key 绑定父 block hash、本 block token 和相关额外条件。例子中两个请求前 10 token 相同，但 block 大小为 4，只有 8 token 命中。运行中的 block 通过引用数保护，释放后的 block 仍可能缓存，直到复用或淘汰。配置和实现需与实际部署 commit 对齐。
- 工程用法：比较冷/热前缀时同时记录 tokenize 后实际匹配长度、可复用完整块、请求的 KV 使用、是否淘汰和 TTFT。相同可见文字可能有不同 token 前缀。若匹配 token 却命中为零，先追 block、key 和淘汰再优化 prefill。训练前缀复用还涉及分支 mask 和反向，这里只覆盖推理。

### `arxiv:2509.19128`

- 原文：[PipelineRL v2](https://arxiv.org/html/2509.19128v2)
- 范围：§§2.1–2.3、3–5、Limitations。核对 Algorithm 1–2、Fig. 2、5–6。
- 方法与结果：异步 actor 持续补请求，保留未完成序列。trainer 按 batch 拉数据并可在序列生成途中更新 actor。把 reward 随墙钟增长拆成样本吞吐与每样本学习效果。作者 128 GPU 的数学训练例中，相对可稳定运行的 G=32 conventional 对照，同奖励约快 2 倍；G=64 发生发散。最大 lag 较高时有效样本量仍可能接近 G=8。
- 工程用法：针对 A3 先画每步 active sequences、队列驻留、每 token/组版本与 learner wait，保留丢弃和质量指标。在同预算下比较改变 rollout 池、组大小或批刷新策略。若定长 workload 几乎无拖尾，或等待主要来自工具，则额外异步推理资源可能帮不上忙。A3 的固定组 snapshot 与本论文中途换权重需要不同验证。

### `arxiv:2508.03680`

- 原文：[Agent Lightning v1](https://arxiv.org/html/2508.03680v1)
- 范围：§§2.1–2.3、3.2–3.4、4.1–4.3 与 §5.2。查看 transition、trace、运行时和三类任务。
- 方法与限制：训练与 Agent 运行分开。从轨迹抽取每次 LLM 的 input、output、奖励和上下文，再分配信用，用现有单轮训练接口更新。SQL、检索问答和计算器任务有任务特定奖励。每次动作的归因仍受状态记录、奖励定义和模型调用边界约束。
- 工程用法：先给真实实验保存每次决策的目标、可见证据、动作和外部反馈，包括未执行原因。把「错误补测」「原调用已运行」「报告写好但测试失败」保留为不同事件。再测试两位评审在相同前缀下能否认可有效下一步。只有目标行为在输入和工具齐备后仍反复出错，才值得把这些数据转成示范或训练。单条最终成功轨迹不能单独说明其所有决策都正确。

### `arxiv:2605.15565`

- 原文：[AstraFlow](https://arxiv.org/html/2605.15565)
- 范围：§§3.1–3.3、4.2.1–4.2.3，核对数据流层、扩缩容和模拟跨区的指标。
- 方法与结果：rollout 和 trainer 通过共享数据流拉取、过滤和路由，并由独立的权重流同步。论文的一个扩缩容案例中，固定 11 GPU 和自动方案用时分别 23.9 h 和 24.4 h，自动方案 312 GPU-h，报告比固定池低约 13%。另一个跨区实验是以限功率和链路整形模拟的拓扑。
- 工程用法：想加入智能运维时，先以 queue depth、learner wait 的滑动阈值和滞回作强规则基线。检查同质量目标下的墙钟、GPU-h、恢复、样本版本和队列尾部。若等待来自评估器、同步或低质量组过滤，优先定位该阶段，避免把加 rollout GPU 当作通用答案。作者硬件与训练任务上的节省不能代入本地预计费用。

### `https://docs.sglang.io/docs/advanced_features/sglang_for_rl`

- 原文：[SGLang for RL Systems · Pause Generation 与版本归属](https://docs.sglang.io/docs/advanced_features/sglang_for_rl#pause-generation)
- 范围：2026-11-13 在线文档的暂停、继续、更新权重、每 token 权重归属、training–inference mismatch。
- 方法与限制：abort 会结束请求，retract 把正在运行的请求放回等待队列并允许重算 KV，in-place 依赖既有 KV，不能继续执行会破坏它的 flush。文档描述了跨版本的 token 区间记录。实际部署需核对所用引擎是否具备这些 API 和字段。
- 工程用法：恢复工具应先看同一 run 的状态、缓存能否保留、版本是否越界、可否重算，再决定继续、回收或重新提交。可离线回放 pause 前已完成、重入后重复、换权重期间并发和丢失 KV 等分支。正确性先检查原始 token、logprob 与版本，再对比恢复成本。

### `arxiv:2310.18547`

- 原文：[Punica: Multi-Tenant LoRA Serving v1](https://arxiv.org/html/2310.18547v1)
- 范围：§§3–5、7.1–7.3，重点是 SGMV、调度、加载、请求迁移与 7B/13B/70B 比较。
- 方法与结果：同 backbone 的多个 LoRA 请求共享主模型 batch，额外低秩矩阵按 adapter 分段聚合。调度记录现有 batch 和 KV 余量，需要时加载 adapter、迁移请求。作者有的对照是只运行 backbone 的 vLLM，有的模拟多个 adapter，分母各异。较长序列时 attention 也会稀释 adapter 算子所占比例。
- 工程用法：先确认业务负担是 backbone duplication、adapter 加载、算子碎片、KV 压力还是尾延迟。用固定 arrival、LoRA rank、热度和序列分布比较批内 adapter 多样性、加载时间和服务指标。若少数 adapter 已常驻且主要延迟在 KV 或 attention，先验证对应瓶颈，再试 LoRA 合并 kernel。推理多适配器不提供训练多租户收益。

### `https://www.trajectory.ai/field-notes/multi-lora-training-for-continual-learning`

- 原文：[Trajectory · Multi-LoRA Training for Continual Learning](https://www.trajectory.ai/field-notes/multi-lora-training-for-continual-learning)
- 范围：作者 2026-05-30 工程报告 §§2–4 的 warm service、训练与采样机制、八任务对照和结果分母。
- 方法与结果：常驻一个 backbone，rollout 混合多个 adapter，训练每次只装载一个租户的 adapter state。在单 H200 节点、Qwen3-4B-Instruct-2507 的工具型 GSM8K 同步 RL 中，八个并发任务的完工总时间较串行约快 2.81 倍，但第一项完工约慢 1.97 倍，步时约慢 2.62 倍。这是作者报告的指定任务，训练奖励相近。
- 工程用法：若目标是快速决定哪个假设值得做，应单独记录首份可判别结果时间。若目标是跑完整 sweep，再记录全部完成时间与 GPU-hours。保持模型、奖励、训练步和计算资源可比。若 rollout 或 CPU 环境吃满且优先任务恶化，减少共享并发、预留优先级或复用同一个热服务做顺序消融。该结果不支持现在就做持续训练。

## 归纳

Orca、PipelineRL 和 multi-LoRA 都能通过提高利用率改善某种总体指标，但分别牵涉请求级 SLO、单条轨迹 lag、最早实验完成时间。将三种总量分别对到目标分母后，才可决定优先优化 scheduler、资源分配或训练配置。当前更适合先在历史轨迹核验这些区分，再观察是否经常遇到足以改变下一步的状态差异。
