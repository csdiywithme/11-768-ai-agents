# L15–L17 · 持久化、执行恢复与评估

本批沿 L15 文档和实证报告检查固定工作流能做到什么。当前实现文档以 2026-11-13 查阅为准。性能效果须另做配对对照。

### `https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph`

- 原文：[Thinking in the Graph API，登记 URL 重定向后的正文](https://docs.langchain.com/oss/python/langgraph/thinking-in-graph-api)
- 范围：状态字段、保持原始数据、节点设计、Command 与错误类型和条件分支。
- 方法与限制：先把状态字段和输出定义好。节点收到结构化事实，可更新字段并选择下一步。读网络临时失败、模型可补救错误、需要用户决定和代码缺陷按不同路径处理。示例来自邮件工作流。
- 工程用法：把 call ID、远端阶段、镜像哈希和计时口径保存在状态中，prompt 从它们生成。对同一日志比较自由调查和带分支的固定图。若阶段和下一动作已能稳定枚举，先测固定图是否降低误重提。若模型必须从未见过的输出识别故障，给该节点工具和受控证据。

### `https://docs.langchain.com/oss/python/langgraph/interrupts`

- 原文：[LangGraph · Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)
- 范围：Pause、Resuming interrupts、serializable payload、重启节点和副作用约束。
- 方法与限制：interrupt 和 resume 依赖同一 thread ID 和可持久化 checkpoint。resume 从节点开头重新执行，前置副作用可能重复。持久化只恢复工作流内部状态，远端提交是否发生仍需核验。
- 工程用法：需要等待用户确认的动作在分开的节点执行，且用稳定运行键连接到服务端状态。回放一次批准前崩溃、批准后网络断开和成功回执丢失。观察是否重复提交或错误取消了仍在运行的任务。

### `https://docs.langchain.com/oss/python/langgraph/test`

- 原文：[LangGraph · Test](https://docs.langchain.com/oss/python/langgraph/test)
- 范围：测试完整 graph、单节点和部分执行，查看 checkpointer 行为。
- 方法与限制：直接调用节点适合测字段和分支。单节点路径会绕过编译时 checkpointer。测试部分执行可以从已知状态进入后续节点。需要独立覆盖重启和恢复。
- 工程用法：单测阶段分类器能否区分编译错误与数值失败。再对同一调用在节点成功后、状态写回前中断，复测恢复。若单测全通过而重启仍重复动作，优先修状态与幂等边界。

### `https://docs.langchain.com/oss/python/langgraph/overview`

- 原文：[LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview)
- 范围：核心能力、durable execution、人审、memory、debugging 和相关入口。
- 方法与限制：图的持久化和检查点可支持多步可恢复执行。框架提供编排能力，规则和工具必须用工作场景的输入与回放验证。
- 工程用法：先用一个会跨进程恢复的任务比较框架和简单状态记录。量化恢复成功率、重复动作和维护成本。如果固定状态和一个远端调用已覆盖主要失败，现有脚本或队列可能足够。

### `https://docs.langchain.com/oss/python/langgraph/persistence`

- 原文：[LangGraph · Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
- 范围：Checkpointers 和 Stores 的边界、thread_id、内存状态重启丢失、保留策略。
- 方法与限制：checkpoint 保存运行中间状态，跨会话知识进入 Store。内存 saver 只在进程内有效。运行时间和轨迹增长也会增大存储及读取开销。
- 工程用法：分别保存远端事实、可恢复流程状态和结论。记录被引用原始文件的路径和指纹。杀掉观察器后重连同一运行，检查查到的版本、阶段和输出，测不同记录方式的回放成本。

### `https://docs.langchain.com/oss/python/langgraph/fault-tolerance`

- 原文：[LangGraph · Fault tolerance](https://docs.langchain.com/oss/python/langgraph/fault-tolerance)
- 范围：Retry boundaries and idempotency、missing acknowledgement、Run timeout / Idle timeout、Recovery strategies。
- 方法与限制：节点重试可能重新执行副作用。稳定幂等键只在下游按键去重时有效。等待回执超时后需要视状态为未知，查证后决定补偿或重试。节点没有进度与总墙钟超限是两种情况。
- 工程用法：先给读日志与创建云调用不同的重试策略。向回放注入「已创建但 ack 丢失」和「从未创建」，比较是否恢复原调用。若服务无法对照幂等键，先把不可判定的重提交给人工或使用可追溯提交路径。

### `https://docs.langchain.com/oss/python/langchain/test`

- 原文：[LangChain · Test](https://docs.langchain.com/oss/python/langchain/test)
- 范围：原页 Unit testing、Integration testing、Evals。此页为测试入口，未给出目标 Infra 测量。
- 方法与限制：mock 测代码契约，真实模型与工具用于集成，轨迹评估可用确定规则或模型评审。三种问题有不同分母。
- 工程用法：先用历史日志和固定工具返回测状态，再验证真实工具如何携带路径与版本。性能和代码正确性都须按其自身验收。若评审说计划可行而工具连证据都读不到，先修集成。

### `https://docs.langchain.com/langsmith/evaluation`

- 原文：[LangSmith Evaluation](https://docs.langchain.com/langsmith/evaluation)
- 范围：Offline/Online workflow、数据集、评估器类型、重复执行与实验比较。
- 方法与限制：离线可用人工、代码规则、模型和 pairwise。线上大多缺少参考输出，需要用可观测的错误类型和采样策略。文档给流程，未证明某种判分在目标任务有效。
- 工程用法：正确性、重复提交、输入口径用确定规则，归因与候选价值由懂场景的评审独立判断。先盲评同一条决策前缀。如果两位评审对目标行为常不一致，先修规格与标签定义，再考虑模型判分或训练。

### `https://modal.com/docs/guide/sandboxes`

- 原文：[Modal · Sandboxes](https://modal.com/docs/guide/sandboxes)
- 范围：创建、运行命令、输出、终止、环境和沙盒生命周期。
- 方法与限制：sandbox、在内执行的进程和控制端各自有标识和完成条件。进程执行可设置超时，沙盒也能终止。此接口不同于用户历史 Modal Function call。
- 工程用法：比较执行工具时，先区分脚本退出、容器结束和控制面失联。记录沙盒或函数调用的服务端标识及日志。回放是否还原同一任务状态。需要重用环境时，测初始化占比、隔离和恢复成本。

### `https://modal.com/docs/guide/sandbox-networking`

- 原文：[Modal · Networking and security](https://modal.com/docs/guide/sandbox-networking)
- 范围：出站控制、block_network、allowlist 与实验性 sidecar 适用条件。
- 方法与限制：可阻断沙盒出站或配置访问范围。sidecar 的部分流量规则有独立语义。应用内允许查看的文件仍由挂载和调用工具决定。
- 工程用法：离线决策回放可只给当时文件与必要工具。若需要远端验证，准备限定输入和明确的网络依赖，先检查隔离是否真的成立。避免后验报告或环境差异混入模型能力比较。

### `https://transluce.org/docent/blog/terminal-bench`

- 原文：[Transluce · Diagnosing a performance regression on Terminal-Bench with Docent](https://transluce.org/docent/blog/terminal-bench)
- 范围：Steps 1–3、超时对照与脚注。核对作者报告的总体与筛选口径。
- 方法与结果：先做成功/失败轨迹比较，再查 metadata。报告总体 89 题平均 reward 约 0.27 对 0.39，超时率约 49.4% 对 29.1%。去掉两侧任一 rollout 超时的任务后排名反转。筛选可能保留更容易的任务，无法建立单一因果解释。
- 工程用法：Agent 调优评估要先报全体任务与失败类型，再把同一任务在固定工具与预算下分类。对超时分支，先验证作业是否启动和是否可恢复。若排队占主要差距，加入可恢复流程或收紧实验规模后重比，保留全体分母。

### `https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/`

- 原文：[METR · Measuring AI Ability to Complete Long Software Tasks](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/)
- 范围：TH1.1 标注、Summary、任务长度、成功率口径和人类时间对照。
- 方法与限制：用人完成任务所需的时间近似难度，拟合模型在这些任务的成功率，50% 时间跨度是模型成功率在半数位置所对应的人类时长。任务族、样本和环境会影响估计。该页面的历史榜不再追新模型。
- 工程用法：把选下一项实验、解释短 trace 与长任务交付分别做样本。记录实际人工主动时、等待、审阅和返工，另报按任务族的有效率。若成功集中于输入齐备的短题，不能用总体得分外推多日优化任务。
