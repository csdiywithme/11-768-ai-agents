# A3 · 从 API 和运行文档查实验契约

这些材料主要回答实验能否按预期运行。先用本地 fake client 和历史日志检查状态，再考虑云端验证。未提交作业或运行远程服务。

### `https://docs.python.org/3/library/asyncio-queue.html`

- 原文：[Python 3.12.16 · Queues](https://docs.python.org/3.12/library/asyncio-queue.html)
- 范围：Queue、PriorityQueue、LifoQueue、例子，重点核对 Queue 的 maxsize、put、get、join 和 task_done。以 A3 指定的 Python 3.12 为准。
- 方法与限制：有界 FIFO 的 put 在满时等待。一次 get 会腾出槽位，但 join 依据 task_done 计数，语义不同。Queue 方法本身没有超时参数。Python 3.12 页面没有更新版本的 Queue.shutdown。入队的快照版本和数据验收仍由应用记录。
- 工程用法：给 A3 异步流程一个容量为 1 的可控队列。验证 producer 阻塞、consumer 慢或异常时状态如何推进，检查发布新权重后旧样本在哪一步丢弃。先从事件找 learner wait 是等队列、过滤、同步还是更新。若瓶颈与队列容量无关，单加缓冲不会提高吞吐。 [A3 原任务](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:249)

### `https://docs.python.org/3/library/asyncio-task.html`

- 原文：[Python 3.12.16 · Coroutines and Tasks](https://docs.python.org/3.12/library/asyncio-task.html#task-cancellation)
- 范围：Task Cancellation、Task Groups、Shielding From Cancellation、Timeouts。读文档的完成、异常传播与等待语义。
- 方法与限制：task 收到取消时抛出 CancelledError，应在 finally 处理清理并通常重新传播。TaskGroup 中首个非取消异常会取消同组其他任务并等待结束。shield 可让受保护任务在等待者取消后继续。远程计算是否取消仍取决于远端 API。
- 工程用法：回放「本地 observer 超时、远端仍运行」及「任一 worker 报错、其余在等队列」。检查原异常、原 call ID、停止动作和中间产物。若远端状态仍显示运行，恢复观察原调用。在本地队列中测清理是否完成，再验证远端取消如何确认。 [DeepInfer 运行记录](../cases/06-framework-delivery.md)

### `https://tinker-docs.thinkingmachines.ai/tinker/`

- 原文：[Tinker SDK 入口和契约索引](https://tinker-docs.thinkingmachines.ai/tinker/)
- 范围：首页的 clients、sampler、Datum、训练调用、并发请求及模型权重说明入口。实际调用见下面独立快照。
- 方法与限制：API 把调用方的数据与算法逻辑同托管的模型执行分开。训练 client 和 sampling client 各有生命周期。相同 API 形状仍需校验后端在 template、权重与调度上的行为。
- 工程用法：选后端前先固定 client 到 trainer/sampler 的契约。用少量已知 token 对拍输入、输出、采样版本、mask 与 checkpoint 的可加载性。若同时换后端和算法后结果改变，用固定轨迹先区分 API 兼容和算法差异。

### `https://tinker-docs.thinkingmachines.ai/tinker/quickstart/`

- 原文：[Tinker Get Started](https://tinker-docs.thinkingmachines.ai/tinker/quickstart/)
- 范围：SFT、RL 示例中 Datum、权重 mask、forward_backward 与 optim_step、save_weights_for_sampler 和重新建立 sampler。
- 方法与限制：示例将输入和目标移位，使用与目标同下标的权重 mask。梯度和更新调用分别返回 future。权重交给采样器是单独步骤。训练示例为教程任务，未验证 A3 的数据、梯度归一或具体 fork。
- 工程用法：在本地构造三五个 token 的最小样本，确认首个 completion target 被训练、提示词被 mask、采样 logprob 与原 ID 同位、失败的优化步骤不发布新版本。若首步 loss 与 CPU 参考不同，先查掩码和分母，再调超参。 [A3 对齐契约](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:211)

### `https://docs.skyrl.ai/docs/tinker/overview`

- 原文：[SkyRL Tinker API · Overview](https://docs.skyrl.ai/docs/tinker/overview)
- 范围：What is Tinker、Why Tinker on SkyRL、Supported Features。核对 2026-11-13 页面，A3 本地 fork 按独立 commit 记录。
- 方法与限制：文档列出 FSDP 和 Megatron 后端、多租户 LoRA 支持范围。目前表中 Megatron + vLLM 支持多租户 LoRA training/sampling，FSDP 多租户和多租户 full parameter 不在其所列支持范围。API 相同也要核对配置、负载和交付状态。
- 工程用法：需要比较分片或多租户时，先在同模型、后端和接口各确认兼容项。区分「文档当前支持」「本地 fork 有代码」「选定配置跑过」。用短探针对拍 checkpoint、输出和租户隔离，再分别测首结果和 sweep 用时。

### `https://docs.fireworks.ai/fine-tuning/training-api/introduction`

- 原文：[Fireworks Training API · Overview](https://docs.fireworks.ai/fine-tuning/training-api/introduction)
- 范围：服务和 Cookbook/SDK 的分工，责任表、运行组件、checkpoint 与迁移对照建议。
- 方法与限制：托管层承担 GPU 执行和相应基础设施。使用 recipe 时许多训练步骤由 recipe 管。用原始 SDK 时调用方编排梯度、优化、rollout、评估和恢复。文档未给出与 A3、SkyRL 在指定 workload 上的配对对照。
- 工程用法：若实际工作要选托管还是自托管，比较同一训练目标下首次合格样本时间、首次可复用 checkpoint、可恢复状态、成本和资源可见性。不要只按「SDK 支持 GRPO」推断相同训练口径。当前调研可先验证数据和评估契约。

### `https://docs.astral.sh/uv/`

- 原文：[uv 官方入口](https://docs.astral.sh/uv/) · [Locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/)
- 范围：项目、Python 版本、统一锁文件，以及自动锁定和同步、检查锁文件、安装环境的说明。
- 方法与限制：uv 把解析写入 lockfile，把依赖同步到项目环境。普通 uv run 可触发锁定与同步。--locked 检查锁文件与项目元数据是否一致，--frozen 跳过锁文件是否需要更新的检查。锁定 Python 包不能替代 CUDA、镜像、驱动、架构和远端构建记录。
- 工程用法：同一实验必须比较相同依赖时，先记录 lockfile 指纹、Python 版本、已装版本和镜像。若无法复现，先查实际生效环境，再看 kernel 或模型修改。将错误 checkout、缺模块和锁文件变化分别记账。

### `https://github.com/radixark/miles/issues/3284`

- 原文：[Miles #3284 · Multi-LoRA / Tinker API roadmap](https://github.com/radixark/miles/issues/3284)
- 范围：路线图正文、后续交付与覆盖说明。仅核对 issue 中的维护者报告。
- 方法与限制：路线图区分 per-slot 模型和优化器状态、服务接口、共享 base 与推理后端。后续记录含冻结 shared router bias 和跨 slot 隔离的 GPU 验收。维护者注明仍有分布式 EP、后端和 CUDA Graph 覆盖限制。
- 工程用法：框架可选功能交付前，应分别核对哪些变化已合入、支持的拓扑、数值/隔离测试和默认路径。给同样的 adapter 独立训练与混合服务各跑一个最小对照，检查另一租户或共享 base 是否受到不应有的改变。现有报告只能用于选择测试点。

### `https://github.com/vllm-project/vllm/issues/4449`

- 原文：[vLLM #4449](https://github.com/vllm-project/vllm/issues/4449) · [当前 CUDA Graphs 设计](https://docs.vllm.ai/en/latest/design/cuda_graphs/)
- 范围：原 issue 的历史参数讨论及当前设计的 runtime mode、嵌套 wrapper 与 warm-up。
- 方法与限制：issue 用历史版本解释 enforce_eager 和超长输入时的回退。当前设计区分 FULL、PIECEWISE 与 NONE，依 batch descriptor 和 attention backend 决定实际捕获、复用或 eager。参数与默认值需以目标 commit 为准。
- 工程用法：若 eager 后显存或时延变了，先验证实际 graph mode、批形、capture 次数与 replay。分开比较冷启动、稳态、图保留内存和正确性。若只有 config 改了但实际一开始就在 eager，先找配置是否生效。A3 的旧选项不能直接映射到最新 runtime mode。
