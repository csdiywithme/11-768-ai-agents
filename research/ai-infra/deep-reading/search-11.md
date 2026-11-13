# L14 深读 · OpenHands SDK 的状态、工具和运行边界

本批细读十二条登记来源，包括官方课件、报告和课程锁定的 SDK 代码。示例为源码阅读，未执行模型、容器或远程工具。

### `https://www.cmu-agents.com/slides/lecture-14-openhands.pdf`

- 原文：[L14 OpenHands 官方课件](https://www.cmu-agents.com/slides/lecture-14-openhands.pdf)
- 范围：通读缓存 PDF 44 页的文本和图示文字。重点核对 pp.13–20、22–28、30–36、39–43。文件 SHA-256 为 `1b4626e1b25b6eaac463e8b82360c9a7b4ffc34f6ccc4b9ab162761d5427a60a`。纯画面的演示页未按完整运行复核。
- 机制：课件展示模型、工具、对话和工作区的分工。对话持久化、上下文压缩、远程执行分别处理不同状态。单 Agent 可共享定位和验证上下文。分工增加独立审查与并行机会，也增加交接。自动化示例要求保存任务触发、适用输入、运行产物与检查结果。
- 改变的动作：先用相同任务比较固定脚本与单 Agent，测可读证据、失败恢复和人工交接。只有可独立核查的定位、实验复核等工作，才增加并行分工。
- 推翻条件：若上下文恢复遗漏关键实验条件，或分工导致重复提交、证据失联、净等待增加，缩小分工或恢复简单流程。课件的示例和引用结果分别核查。

### `arxiv:2511.03690v1`

- 原文：[The OpenHands Software Agent SDK](https://arxiv.org/html/2511.03690v1)
- 范围：arXiv v1。细读 §§3.1–3.4、§§4.1–4.7、§§4.9–4.10 和 §5。核对 Tables 1–2 和 Figure 8。
- 机制：把不变的 Agent 配置和会话可变状态拆开。会话元数据与按事件追加的日志分别落盘，恢复后从未处理事件继续。Action、Executor 和 Observation 定义工具边界。程序测试、真实模型集成和完整基准分三层。Table 2 在 SDK `54c5858`、benchmarks `88f1d80` 上报告 Sonnet 4.5 的 SWE-bench 分数 72.8%。
- 改变的动作：用提交、运行配置和工具版本界定实验，记录模型指令、动作与结果。先用离线检查测协议与恢复，再用少量真实模型检查，最后跑代表性 GPU 对照。把恢复会话与外部运行状态核对分开。
- 推翻条件：若中断时恢复动作不确定或重复执行，增加运行身份和重试验证。分数依赖模型、基准和 SDK 版本，不能据此估计 AI Infra 任务的收益。

### `https://github.com/OpenHands/software-agent-sdk/tree/69e26889401fe69157fff536e6a69049e6644cb3`

- 原文：[OpenHands SDK 课程锁定版本](https://github.com/OpenHands/software-agent-sdk/tree/69e26889401fe69157fff536e6a69049e6644cb3)，[固定版 README](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/README.md)
- 范围：通过 GitHub 接口核对 commit 与同 commit 的 README 和九个示例。README blob SHA 为 `386dc8945133cb51969192b0b29c8396719cbff6`。未核查整个源树和所有服务器实现。
- 机制：README 说明 SDK、Agent Server 与 TypeScript client 同属此仓库。Agent Canvas 通过客户端接入，自动化仓库处理调度和运行记录。基础示例通过相同对话接口接本地或独立工作区。不同层可独立配置和测试。
- 改变的动作：把 GPU 分配、运行命令和结果采集留在现有执行系统，工具只描述输入、输出和运行身份。进度展示或调度从同一记录读取。比较单脚本、SDK 适配与远程服务各自的维护时间。
- 推翻条件：若适配导致同一测量从两个服务重复提交，先厘清运行责任。若单脚本已有可靠恢复及留痕，远程和 GUI 的开销应单独说明。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/01_hello_world.py`

- 原文：[OpenHands hello world 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/01_hello_world.py)
- 范围：读完整 29 行，blob SHA 为 `832378e26c594e05ea31dad1fded28df00affeef`。
- 机制：分别配置 LLM、终端、文件编辑器、任务跟踪和当前目录工作区，再显式提交消息与运行。示例生成文本产物，用打印表达程序结束。
- 改变的动作：设计最小 AI Infra 基线时选择一份已存在的本地 trace，指定允许的观察和输出位置。用独立断言检查报告读到了对应运行和指标。
- 推翻条件：若默认配置的模型、路径或工具无法重现，显式固定这些条件。程序结束和产物存在都不足以验证内容正确。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/02_custom_tools.py`

- 原文：[OpenHands custom tools 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/02_custom_tools.py)
- 范围：读 L1–L229 的类型、执行、结果转换、工具注册和回调。blob SHA 为 `e55d42027c046e6e7eb3473d234c7ee3c6c12722`。
- 机制：结构化搜索参数进入 executor，搜索和终端可共用执行器。executor 限制为前 100 行，Observation 给模型显示更小的示例。代码对文件路径按字典顺序排序，说明文字却写按修改时间排序。示例数不是完整命中数。
- 改变的动作：对日志与 profiler 查询返回运行、筛选条件、观察范围、截断状态和可回查的原始位置。用同输入比较自由终端、专用工具与确定脚本的失败率。
- 推翻条件：若截断或输出顺序改变下一步选择，保留明确的全量接口。若专用工具没有减少误读或重复调用，先修观察格式和证据关联。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/10_persistence.py`

- 原文：[OpenHands persistence 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/10_persistence.py)
- 范围：读完整 104 行，核对保存和用相同会话身份重建。blob SHA 为 `32da4e3efffb767198bad8feb28559d6b8d500be`。
- 机制：通过相同的 persistence directory、conversation ID 和工作区重建会话。示例先完成两轮，再恢复并询问历史。工作目录独立传入。
- 改变的动作：恢复 GPU 调优前分别校验对话 ID、代码 checkout、运行 ID 和远端结果。进程重启后先检查在途实验，再决定是否重试或继续判断。
- 推翻条件：用模拟中断、已完成但本地未接收及中途改动三类场景检查恢复。若仍重复运行，补充显式的外部状态核对。示例未覆盖远端任务或机器重启。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/14_context_condenser.py`

- 原文：[OpenHands context condenser 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/14_context_condenser.py)
- 范围：读完整 137 行的压缩设置、事件回调、恢复和汇总成本。blob SHA 为 `deb3ab7c15992e4c6cf91a9178915376b9a8ad63`。
- 机制：`LLMSummarizingCondenser` 的模型副本有独立使用标识，示例把事件阈值设 10，保留开头 2 个事件。超限后用模型摘要替代较早事件的一部分。示例汇总会话成本。
- 改变的动作：用同一系列实验分别比较完整历史、摘要和可复查记录。重点确认约束、否定结果、GPU 类型和计时口径是否仍影响下一步。保留压缩前后的成本。
- 推翻条件：若摘要导致把不同运行的数据合并、漏掉失效方案或重复实验，改用固定状态与原始链接。示例的事件阈值与实际 token 上限须分别测量。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/02_remote_agent_server/02_convo_with_docker_sandboxed_server.py`

- 原文：[OpenHands Docker workspace 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/02_remote_agent_server/02_convo_with_docker_sandboxed_server.py)
- 范围：读 L1–L126 的架构和镜像选择、工作区生命周期、RemoteConversation、事件及清理。blob SHA 为 `1ee0d4d8c47e29b8d1c74a3536be6096b297718a`。
- 机制：本地代码建立容器工作区，创建远程会话并通过回调接收事件。CI 条件可选对应提交的镜像，其他路径用浮动镜像。示例记录运行状态，额外等事件平息，最终关闭会话。
- 改变的动作：GPU 任务固定运行镜像、驱动与设备。把终态与可下载产物当两个可观测条件。重连后查相同 run ID，区分未启动、运行、执行失败和产物未齐。
- 推翻条件：若同接口在本地和容器对设备、文件或超时行为不同，分环境测量。事件暂时安静不能确定产物已完备。源码示例没有实测 GPU 资源或隔离强度。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/04_confirmation_mode_example.py`

- 原文：[OpenHands confirmation mode 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/04_confirmation_mode_example.py)
- 范围：核对 L1–L145 中的防护流程，重点读待处理动作的显示、等待状态、拒绝和继续执行。blob SHA 为 `9bd15b1cc88a7abc1c74ac1456dee0f88ca6eabc`。
- 机制：确认策略可在动作发生前让会话暂停。示例会从事件取待确认动作，对用户显示摘要和截断后的原动作片段。拒绝会产生明确的观察，供下一轮重新决定。
- 改变的动作：把需要人决定的优化试验表示成可检查的动作，写明影响的工作区、资源与计划。人的判断应能看到实际参数和相应证据。记录拒绝与后来执行的差异。
- 推翻条件：若摘要不能辨认高成本或影响范围，先补真实动作和状态。若同一已授权动作反复暂停，检查状态关联。该示例展示交互机制，权限仍由执行边界检查。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/03_activate_skill.py`

- 原文：[OpenHands skill activation 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/03_activate_skill.py)
- 范围：读完整 134 行。核对 always-active、关键词触发、来源和回调。blob SHA 为 `c1356d9e17cb13c17bb708c4b98149717042234e`。
- 机制：示例分别设置常驻、按关键词触发和公共来源的 skill。系统或用户后缀也能改输入。skill 作为上下文传入，来源字段可保存文件位置。
- 改变的动作：给容易混用的 Nsight 命令、框架命名和项目测试入口附上适用版本。触发后核对实际消息含哪些材料。把清晰的工程步骤写入技能，把可判定检查留给程序。
- 推翻条件：若关键词在不同框架或问题中触发了错误版本，加入上下文匹配或人工选择。若常驻材料增加阅读成本但无新行动，把它保留为可检索文档。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/07_mcp_integration.py`

- 原文：[OpenHands MCP integration 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/07_mcp_integration.py)
- 范围：读完整 90 行。只记录已配置工具、工具可见性、回调与防护。blob SHA 为 `87bb6d488d6edecd1c2c9411bbda569cf5faff3e`。
- 机制：示例为两类 MCP 工具配置固定版本，其中一类仅显示选定能力。和本地工具共同进入 Agent 的工具集。事件回调可观察模型所见消息。
- 改变的动作：若从多个工具拿日志或源码，先给出当前实际可见的只读查询能力，并保存工具版本及结果来源。用同一个小任务比较直接文件读取与结构化工具。
- 推翻条件：若工具集变化使模型调用错误增多或误取另一仓库资料，校验当前会话的 schema 和工作区。可见性筛选不承担实际授权和运行隔离。

### `https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/25_agent_delegation.py`

- 原文：[OpenHands agent delegation 固定源码](https://github.com/OpenHands/software-agent-sdk/blob/69e26889401fe69157fff536e6a69049e6644cb3/examples/01_standalone_sdk/25_agent_delegation.py)
- 范围：读完整 139 行的角色工厂、并发限制、主 Agent 合并和追加任务。blob SHA 为 `d222109f20acd0c06a764d2bad5b86f1682141c2`。
- 机制：用工厂注册两个无工具的专门角色，主 Agent 通过任务工具发出分工，并将并发限制为二。示例随后要求主 Agent 合并结果和继续问一个子任务，成本按会话汇总。示例未提供质量对照。
- 改变的动作：对同一次性能调研让两个只读分工分别检查算子时间线和服务指标，交接时给相同运行和原始证据。再用单 Agent 基线测结论、并行节省与重复阅读。
- 推翻条件：若并行报告基线不一致、遗漏关键相互依赖或人工复核时间更长，改为单 Agent 或共享结构化输入。示例并发能力不能证明目标问题适合分工。
