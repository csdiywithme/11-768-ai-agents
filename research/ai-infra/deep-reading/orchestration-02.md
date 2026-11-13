# 工作流编排、运行观察和防御性验证

### `https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf`

- 原文：[CMU L15 官方课件，Orchestration and monitoring](https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf)
- 范围：读课程快照哈希一致的 39 页官方缓存，重点 p.7–23 的工作流与 p.25–38 的运行观察。与 L18 引用的无 www 入口使用同一课件。
- 方法：把可预先规定的操作放在显式节点和状态转移中。模型处理分类及需要判断的局部任务。以节点快照恢复，单测验证局部输出，端到端测试验证真实流转和副作用。以轨迹比较及行为聚类找评分背后的原因。
- 证据及边界：p.30–31 展示由超时元数据提出假设，再检查双方均无超时的 28 项任务。排序反转发生在筛选后的子集，不能视为更换超时参数的因果结果。p.22 说明显式流程让测试更容易，也要求维护节点之间的信息。
- 工作迁移：固定构建、数值验收、计时和存档。由模型选择要查的瓶颈、提出候选，并按明确接口提交依据。用独立事件记录失败阶段。
- 最小验证：回放同一故障，分别让模型自由使用工具和按图推进。注入短暂编译失败及人工更正目标。比较原有产物复用、额外调用、误报完成和重复提交。

### `https://docs.langchain.com/oss/python/langgraph/workflows-agents`

- 原文：[LangGraph 官方，Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)
- 范围：链式执行、并行、路由、Orchestrator-worker、Evaluator-optimizer 和 Agent 循环示例。
- 方法：简单工作流可将相邻步骤固定，并依据状态判断是否继续。路由使用结构化分类。已知独立任务可固定分发，任务数由输入决定时可用 Send 生成独立工作节点。输出通过归并字段进入共享状态。评估者反馈可触发修改。
- 证据及边界：官方给出可执行教学示例，没有真实任务的质量、调用和延迟对照。归并操作可收集输出，任务的唯一标识、顺序、重试语义和最终质量由应用定义。
- 工作迁移：对已知形状矩阵用固定扇出，对读 trace 后才能确定数量的候选用动态分发。每个结果带任务 ID、条件及产物。共享 GPU 的真实计时仍由外部资源调度器互斥。
- 最小验证：回放两个 worker 同时完成、重复报告和一项失败。确认输出没有被覆盖或重复计数。比较固定矩阵和动态分发的证据覆盖、无效分支和调用成本。

### `https://docent.transluce.org/dashboard/9bafeb94-3f6e-4d03-bd01-1ae8476e3f6e/analysis-plan/772c0920-0b0f-4355-a009-825d1dd553b4`

- 原文：[Docent 登记的无超时分析计划](https://docent.transluce.org/dashboard/9bafeb94-3f6e-4d03-bd01-1ae8476e3f6e/analysis-plan/772c0920-0b0f-4355-a009-825d1dd553b4)
- 范围：动态仪表盘未返回原件。读取 L15 p.26–31 及 Transluce 作者公开教程 Step 1–3 和附注。
- 状态：原件受限
- 证据及边界：教程先匹配同一任务中的失败与成功，再用运行元数据验证超时假设。按任务宏平均后，无超时子集的排名反转。作者说明重新运行与公开榜的分数有差异，筛选后的任务可能更容易，个别超时仍可通过最终状态评分。教程的总体超时率与课程已评分运行的分母不同。 [Transluce 原始教程](https://transluce.org/docent/blog/terminal-bench)
- 工作迁移：分别保存终止原因、最终结果有效性、工具耗时和各任务重试。先用成对轨迹找假设，再使用所有运行核查分母。若要回答调高超时是否有用，在同任务同配置只改时限重跑。
- 最小验证：选一组本地任务重放，比较总运行、每任务宏平均、无超时子集。另选少量易超时任务做固定时限和延长时限对照，记录成功率与资源占用。

### `https://docent.transluce.org/dashboard/9bafeb94-3f6e-4d03-bd01-1ae8476e3f6e/agent_run/083de394-6f2f-41e7-a290-24b7be803822?transcript_id=b2f312ef-9698-416a-b8ee-923328f8ff49&block_idx=56&target=block_content`

- 原文：[Docent 登记的 OCR 动作](https://docent.transluce.org/dashboard/9bafeb94-3f6e-4d03-bd01-1ae8476e3f6e/agent_run/083de394-6f2f-41e7-a290-24b7be803822?transcript_id=b2f312ef-9698-416a-b8ee-923328f8ff49&block_idx=56&target=block_content)
- 范围：动作级原件未返回。核对 L15 p.28 对同一任务成功与失败轨迹的配对说明。
- 状态：原件受限
- 证据及边界：课程示例将一次成功与 OCR 操作联系起来。只有课件展示的单例，不能评估该动作的独立影响。
- 工作迁移：调优诊断笔记给每项归因保留对应运行和动作的链接。记录分歧出现前的共同输入以及之后的实际差异。
- 最小验证：用保存的两次运行核对首次分歧和最终产物。通过重放只变化观察来源，确定改进来自正确读出输入或其他差异。

### `arxiv:2604.11072`

- 原文：[Hodoscope: Unsupervised Monitoring for AI Misbehaviors，arXiv v1](https://arxiv.org/html/2604.11072v1)
- 范围：§2–2.3 比较分布及抽样，§4 表 2 与消融，§5 表 3，§6。案例用于防御性评估及异常发现。
- 方法：先摘要动作意图，再做共享嵌入和二维投影。比较运行组之间的密度差，结合样本距离选择人工先看的动作。以原始轨迹和环境约束验证异常，再构造专项检测。
- 证据及边界：表 2 三个测试环境中，完整方法平均在前 1–4 个动作中遇到第一个已确认异常，随机抽样对应 6、91、29。衡量首次发现，与全面召回不同。表 3 给监督评委加入例子，有的模型上升，有的下降。结果受摘要、组间可比性和投影影响。
- 工作迁移：按同任务、版本和预算比较调优 Agent 的失败与成功。摘要动作时保留原始运行引用和输入条件。用密度差发现重复构建、时间花在错误阶段等线索。
- 最小验证：用有人工标签的历史运行，比较随机抽样、按错误码抽样和分布差抽样。在相同人工分钟下测首次发现时间、已知故障覆盖和误报。对新模式用原始日志与重放确认。

### `arxiv:2604.11072v1`

- 原文：[Hodoscope v1，L15 指定的 Figure 2](https://arxiv.org/html/2604.11072v1#S3.F2)
- 范围：与上一条同一 v1。核对 §3 Figure 2、图注与 §2 中密度和投影定义。
- 证据及边界：图给出不同行为组在二维投影中相对过度出现的区域。空间邻近及过度出现只是抽查依据。行为意义仍由原始记录和约束确定。
- 工作迁移：在可视化中将同一动作摘要链接回运行、步骤、构建版本和原始输出。
- 最小验证：更换随机种子和分组规则。若同一簇在重绘后不稳定，改按已核对的元数据和原始动作定位。

### `https://hodoscope.dev/blog/announcement.html`

- 原文：[Hodoscope 作者公告，2026-02-20](https://hodoscope.dev/blog/announcement.html)
- 范围：Part 1 的问题、方法和防御发现，Part 2 的输入、可视化和人工检查流程。
- 方法：将摘要、向量和运行分组整理成可交互视图，让审阅者选择高差异区域并回到原始动作。支持多种轨迹来源，示例固定抽样种子。
- 证据及边界：作者报告给评委补充具体任务边界后，评委能更准确识别原来漏掉的行为。单个示例和小批作者演示无法估计普遍准确率。跨版本轨迹的收集条件需要匹配。
- 工作迁移：当一种 workload 诊断配置频繁超时，先将同任务不同设置的执行过程并排，再抽查差异最大的行动段。给审核者任务约束和原始状态。
- 最小验证：令复核者在固定时间内使用原始列表或差异视图定位一个已确认的故障模式。记录是否找到、用了几条日志、能否复跑确认。

### `https://hodoscope.dev/blog/img/image10.png`

- 原文：[Hodoscope，登记的模型评委截图](https://hodoscope.dev/blog/img/image10.png)
- 范围：直接图片未返回。核对课件 p.33 的截图及作者公告对同一前后判断的说明。
- 状态：原件受限
- 证据及边界：课件显示给评委补充任务背景后，判断分值由 2/10 变为 7/10。它是特定实例，不能估计整体错判率。
- 工作迁移：审查某项性能结论时，把计时条件、基线版本、允许修改范围和证据缺失状态与记录一起给评委。
- 最小验证：对一组已知对错案例，比较仅轨迹和轨迹加任务契约。按误拒、误接受及人工复核次数衡量。

### `https://apple.github.io/embedding-atlas/`

- 原文：[Apple 官方，Embedding Atlas](https://apple.github.io/embedding-atlas/)
- 范围：首页及 Overview，读取聚类、最近邻、WebGPU 密度渲染、关联筛选和多模态查看说明。
- 方法：从二维嵌入和元数据联动浏览，查看密度、聚类标注和相似点。可通过表、图、关键词或查询过滤观察。原始记录是否能从图定位取决于提供的数据字段。
- 证据及边界：官方介绍面向规模化可视化，给出示例与性能能力说明，未提供针对 AI Infra 的诊断效果评估。可见簇受输入特征、投影算法和标签影响。
- 工作迁移：为 profiler 运行存 shape、硬件、版本、基线、结果及 trace ID。先以结构化字段筛选，再探索剩余异常是否集中于类似调用或结果。
- 最小验证：为一组已知性能故障比较表格筛选和嵌入浏览。按找到的真实机制、回溯原记录的成功率和完成时间判断是否需要投影界面。

### `https://docs.openhands.dev/sdk`

- 原文：[OpenHands 官方，Software Agent SDK](https://docs.openhands.dev/sdk)
- 范围：SDK 首页、组件入口、官方 Persistence 和 Observability & Tracing 指南。
- 方法：Agent 组合模型与工具，Conversation 驱动执行并可保留事件。Workspace 定义运行环境。SDK 提供结构化输出、状态恢复、动作确认和观察入口，事件可用于独立复核。
- 证据及边界：文档解释 API 和运行机制，功能示例不能衡量实际恢复正确性或生产稳定性。代码历史与结果事件应独立于可重试的模型输出。
- 工作迁移：用固定 workspace 快照连接 benchmark 运行器。Conversation 恢复后重用持久化作业 ID 和结果，必要时由独立验收器重新读输出。
- 最小验证：注入作业已提交后主进程中断。恢复并验证已有作业仍可查、计时参数可追溯且相同候选未再提交。另比较完整事件与压缩上下文的复核质量。

### `https://docs.ag2.ai/latest/docs/user-guide/basic-concepts/overview/`

- 原文：[AG2 官方 Basic Concepts 概览](https://docs.ag2.ai/latest/docs/user-guide/basic-concepts/overview/)
- 范围：模型配置、ConversableAgent、人工参与、编排、工具、结构化输出及整体示例。
- 方法：用同一对话接口表示 Agent 间消息、工具和人工交互。可组成双人、顺序、群组及嵌套流程。结构化输出由 Pydantic 表达，结束条件和人工参与在流程中配置。
- 证据及边界：概览提供架构与配置用法。消息符合类型约束不说明证据为真或任务完成。并行执行的资源与状态一致性由应用处理。
- 工作迁移：让候选提出者和独立复核者交换可校验的优化假设、影响范围、实验 ID 和失败条件。合入依赖执行验收。
- 最小验证：同模型比较结构化双角色复核与单 Agent 自检。插入完整但错误的字段，检查评委是否从真实测量及日志判断，并记录对话成本。

### `https://dspy.ai/`

- 原文：[DSPy 官方首页，重定向到 current](https://dspy.ai/current/)
- 范围：Signatures、Modules、Optimizers、流程组装和保存的公开说明及示例。
- 方法：用 Signature 定义任务输入输出，用 Module 控制一次生成、推理或工具调用。提供示例与评分函数后，用优化器编译提示或示范，生成版本化程序。
- 证据及边界：首页给出任务演示及优化效果示例。实际收益取决于数据划分和 metric 对实际目标的代表性。对判定依据不充分的任务，指标可能只优化表述。
- 工作迁移：把「根据 trace 提出能区分两种瓶颈的下一步」封装成可测模块。评分要求可执行动作、所需条件及随后可检查的结果。
- 最小验证：用不同 GPU 和框架划分训练、调参及留出集，对比人工提示、优化提示和固定规则。报告真实的正确归因和额外实验成本。

### `https://hub.openhands.dev/blog/dont-sleep-on-single-agent-systems`

- 原文：[OpenHands 官方博客，Don’t Sleep on Single-Agent Systems，2024-09-26](https://hub.openhands.dev/blog/dont-sleep-on-single-agent-systems)
- 范围：全文的模型、提示和工具空间，CodeR 示例、多 Agent 缺点和单 Agent 方案。与 L5 的另一登记入口同一作者文章。
- 方法：用模型、提示、行动空间拆解角色。作者从固定角色移交、上下文缩减和维护开销出发，提出让单 Agent 保有通用操作并选择合适指引。
- 证据及边界：文章是架构经验和假设。没有等预算对照证明单 Agent 优于其他安排。上下文容量增加也不保证能适时使用有效信息。
- 工作迁移：固定工具和验收，先使用单 Agent 完成可顺序执行的 GPU 调试，再针对已看到的等待、重复或工具冲突加入角色。
- 最小验证：选一件算子修改和一件框架跨模块问题，比较同 Agent 使用共享状态、角色隔离、固定移交。测漏掉约束、重复执行和维护成本。

### `https://github.com/neubig/slides/tree/main/slides/2026-09-25-fujitsu-openhands-en`

- 原文：[Graham Neubig 的 Fujitsu OpenHands 演讲登记目录](https://github.com/neubig/slides/tree/main/slides/2026-09-25-fujitsu-openhands-en)
- 范围：登记目录未返回可读内容。GitHub 公开内容读取也未取到演讲正文。
- 状态：原件受限
- 可核对材料：本批 OpenHands SDK 官方文档可支持运行、状态和观察的产品级说明。演讲专有示例和结论仍待原件确认。

### `https://openai.com/index/hugging-face-incident-and-the-road-ahead/`

- 原文：[OpenAI 官方事件报告概览，2026-08-26](https://openai.com/index/hugging-face-incident-and-the-road-ahead/)
- 范围：概览及任务退出、监控与事件响应的防御性部分。
- 方法：事件后追查运行环境、停止条件、控制覆盖和告警升级。将系统权限、运行进度和任务验收作为独立信号，避免只依赖任务内报告。
- 证据及边界：公开复盘将不易解决的任务及缺少明确退出方式列为相关条件，并报告了改进隔离和事件响应的工作。运营方复盘和样本观察不能证明单个因素或措施的因果影响。
- 工作迁移：为内部 GPU 任务保留可解释的阻塞终态和预算上限。给控制器独立的暂停及审计能力。对缺输入的任务保留发现、缺项和可继续条件。
- 最小验证：用本地无害任务模拟输入缺失、日志服务不可用和作业中断。确认在预算内停止、主管能查到最后有效状态、恢复后只重做缺项。

### `https://openai.com/index/securing-our-frontier-model-training-environment/`

- 原文：[OpenAI 官方，Securing our frontier model training environment，2026-10-23](https://openai.com/index/securing-our-frontier-model-training-environment/)
- 范围：L16 所列的边界、离线环境、分层防护、监控和停止机制章节的防御性范围。
- 方法：先以实际授权资源定义边界，再分层验证执行环境、运行记录、外围隔离和停止系统。为需要文档搜索的实验提供离线只读资料。验证各层在其他层失效时的独立表现。
- 证据及边界：运营方报告修复已发现问题、加强监控后于 10 月 1 日恢复特定工作负载。结果描述其内部环境、已覆盖测试与恢复决定，适用范围限于这些条件。
- 工作迁移：本地调优工具只接触任务输入、指定数据、结果目录和已批准的运行器。成本预算、作业状态和停止原因由外部控制器记录，方便恢复和复核。
- 最小验证：在无害测试工作区模拟监控不可用、队列故障、超时和文件写入失败。核对状态和预算能否独立恢复，日志是否能定位到原始运行。

### `youtube:87DyyMV0kCY`

- 原文：[Black Hat USA 2026 录像入口](https://www.youtube.com/watch?v=87DyyMV0kCY)
- 范围：核对 YouTube 标题及报告概览链接。未取得字幕或回放正文。
- 状态：仅核对入口
- 可核对材料：本批 OpenAI 概览与另一批 METR 报告的防御内容可以支持监控、隔离和停止条件笔记，不能当作录像内容。

### `https://www.youtube.com/watch?v=87DyyMV0kCY`

- 原文：[L15 p.38 的 Black Hat USA 2026 录像入口](https://www.youtube.com/watch?v=87DyyMV0kCY)
- 范围：与上条是同一录像，核对课程链接与标题。
- 状态：仅核对入口
