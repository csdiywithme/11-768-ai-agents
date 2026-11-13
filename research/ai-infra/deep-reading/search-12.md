# L1 和 L14 · 最小控制循环、代码代理和运行环境

### `https://www.cmu-agents.com/slides/lecture-01-agents.pdf`

- 原文：[L1 官方讲义](https://www.cmu-agents.com/slides/lecture-01-agents.pdf)
- 范围：官方 PDF 39 页的文本、代码和图示标题。SHA-256 为 `16ad94b7cea06da50ff1736fba9ba3afcbb753fdea51c57b8df1ae3f9a7dd26c`。pp. 2–6 的演示另有登记。pp. 33–38 是讲义当时的课程安排。
- 机制：pp. 9–15 将任务、工具定义、历史和模型动作组织成循环。模型选择动作和参数，harness 解析、验证、执行，再回传观测。pp. 17–25 把六种能力映射到 harness 与训练，例如 schema 校验与工具调用轨迹 SFT、上下文压缩与长上下文训练。pp. 26–31 分别介绍 harness、sandbox、推理、训练和观测。
- 改变的动作：先把算子调优写成实验循环。工具参数包含 workload、设备、精度和预算。将失败分为输入格式、访问权限、环境状态和模型判断。固定模型替换工具或观测，再固定工具对照模型或训练样本，定位改进发生在哪一层。
- 反证：若工具已返回完整 shape、计数器和资源限制，模型仍反复选错瓶颈，扩展 schema 的边际作用有限。若换模型仍触发同一个 runner 状态丢失，检查环境和恢复。讲义提供对照方法，未测 GPU 调优效果。

### `https://github.com/SWE-agent/mini-swe-agent/blob/main/src/minisweagent/agents/default.py`

- 原文：[mini-swe-agent 默认控制循环](https://github.com/SWE-agent/mini-swe-agent/blob/main/src/minisweagent/agents/default.py)
- 范围：`main` 文件，blob `ca310e37d7c61888c576f73f5e1790ac769407b1`。读配置、模板、运行、预算、执行与序列化。课件第 15 页展示简化代码。
- 机制：模型和环境分别注入。控制循环查询模型、追加消息、执行动作并追加观测。格式错误计入模型费用，多次连续出错后有独立退出状态。配置输出路径后每轮在 `finally` 保存轨迹，序列化包含模型和环境信息。预算在模型调用前检查，已启动的单轮远程动作仍可超出软上限。轨迹直接写文件。
- 改变的动作：区分格式失败、编译失败、作业等待、预算耗尽、数值失败和完成。远端 GPU 实验用 job ID 和 run ID 保存进展，分别限制启动和取消时间。调试与训练可共用结构化轨迹。
- 反证：在提交作业后、保存轨迹前中断。若恢复重复提交或只有 transcript 没有现存 job ID，须在运行状态层补足恢复，再测事故后的人工时间。

### `https://github.com/SWE-agent/mini-swe-agent/blob/main/src/minisweagent/config/default.yaml`

- 原文：[mini-swe-agent 默认模板与观测配置](https://github.com/SWE-agent/mini-swe-agent/blob/main/src/minisweagent/config/default.yaml)
- 范围：`main` 文件，blob `7753df3e42e7c9e792da9d063311221b299ba4d6`。读动作、任务、观测模板、默认值与格式错误回传。该文件和当前 CLI 的 `mini.yaml` 是不同入口。
- 机制：该示例用单个标记代码块传动作。每次执行进入新的 subshell，目录和环境变量变化需要显式重建。观测含返回码。输出达到 10,000 字符时取首尾各 5,000 字符并标记省略量。模型发终止信号。成本和步数设为 0，运行器由此关闭相应上限。
- 改变的动作：给长 profiler 输出提供结构化摘要和原始文件定位，显式返回缺字段、截断和异常。让模型按时间范围、设备或 kernel 查完整片段。用独立的数值、性能、兼容性检查判定交付。
- 反证：把关键 counter 放到长日志中段，比较首尾截断和按 kernel 查询后的定位质量。如果两路结果相同，保持轻量观测。

### `https://github.com/swe-agent/mini-swe-agent`

- 原文：[mini-swe-agent 官方项目主页](https://github.com/swe-agent/mini-swe-agent)
- 范围：`main` 的 README、目录、v2 提示、设计取舍及 SWE-agent 对照。文件行为以相邻源码条目为准。
- 机制：README 把短控制循环、线性历史、单次环境执行和可替换环境作为简洁基线。需要改变工具集或 history processor 时，作者列出接口更多的 SWE-agent。单步不依赖持续 shell，文件可以留在 workspace。
- 改变的动作：在同一模型、设备和预算下比较 shell 基线和结构化 profiler 工具。测成功率、GPU 空等、人工介入以及参数或单位错误引起的无效运行。用相同任务评估控制循环和 SFT 样本的可移植性。
- 反证：若定制工具只减少提示长度，换硬件或框架时因字段不兼容引发更多失败，缩小封装范围并保留基线。项目自报分数只对应原 SWE-bench 条件。

### `arxiv:2310.06770v1`

- 原文：[SWE-bench v1 全文](https://arxiv.org/html/2310.06770v1)
- 范围：2023-10-10 v1，§§ 2、4–5、7、Appendix A.3–A.4。与另行登记的后续版本分开引用。
- 机制：从 12 个 Python 项目构造 2,294 个 issue–PR 实例。固定旧提交及问题描述，分修复前失败而修复后通过，以及前后都通过两组测试。候选 patch 应用后，两组测试均须出现且通过。Table 5 中，Claude 2 用 BM25 选文件解决 1.96%，用参考 patch 改过的文件解决 4.80%。后者仍可能缺关键调用链，且信息来自事后。
- 改变的动作：为历史性能任务分别定义目标 workload 的修复或改善，以及已有 workload 的数值和性能回归约束。用事后已知调用链做诊断对照，判断瓶颈在找代码还是改代码。日常运行用当时可获取的源码、日志和环境。
- 反证：给正确函数和测试也无法提高修复率时，扩大检索预算作用有限。单测全通过却在 TTFT、ITL 或兼容性上失败时，补性能和兼容验收。论文 v1 测静态 patch、特定 Python 项目和当时模型。

### `https://github.com/SWE-agent/SWE-agent`

- 原文：[SWE-agent 官方仓库](https://github.com/SWE-agent/SWE-agent)
- 范围：`main` README，blob `e241ede47c338b69eb13cbc5423047e1edf1d50f`。读软件工程用法、YAML 配置、批量评估入口与维护说明。
- 机制：工具和行为配置集中在 YAML，提供交互及批量 benchmark。README 指向维护重心 mini-swe-agent，并列出不同时间、模型和分片的作者榜单结果。论文 ACI 对照是独立证据。
- 改变的动作：版本化工具启用、提示、环境和预算。比较只换一项配置时如何影响定位、候选质量或消耗。若研究不同历史处理，选能替换相应接口的框架，再算两套 harness 维护成本。
- 反证：相同 YAML 在不同镜像、框架提交或模型版本发生不同退出时，补记环境和模型版本，再测差异是否归于工具。

### `https://github.com/OpenHands/OpenHands`

- 原文：[OpenHands Agent Canvas 官方仓库](https://github.com/OpenHands/OpenHands)
- 范围：`main` README，blob `32c308ddc1d5a9e82c72ccbae497395ac403e11c`。读产品、后端、集成和仓库边界。历史论文另记。
- 机制：Agent Canvas 是可自托管的开发操作界面。用户创建会话并观察自动化。后端可以在本地、Docker、VM 或云环境。界面接入不同 ACP Agent。自动化可按计划或事件触发。
- 改变的动作：若工程师主要花时间盯作业、切工作区和接手会话，先测共用操作界面的价值。用同一运行器和工件格式，让工程师可从异常回到源 run，检查假设并决定下一轮。若主要问题是候选质量，按同一预算直接比较候选策略。
- 反证：运行器相同时，若接手耗时几乎不变，新增界面又增加状态同步成本，产品层暂时不产生收益。README 未报告后端等价性。

### `https://github.com/OpenHands/software-agent-sdk`

- 原文：[OpenHands Software Agent SDK 官方仓库](https://github.com/OpenHands/software-agent-sdk)
- 范围：`main` README，blob `386dc8945133cb51969192b0b29c8396719cbff6`，与 L14 固定提交的 README blob 相同。登记入口各自保留，架构结论只算一份证据。
- 机制：SDK 和 Agent Server 管 Agent、工具、会话、workspace、事件和 REST/WebSocket。TypeScript client 对接服务端契约，Agent Canvas 消费 client。automation 仓库持有排程、历史和派发。SDK 执行会话。本地与远端共享会话接口。
- 改变的动作：为本地预检和远端 GPU 抽取统一作业和结果契约。应用负责队列、预算、配额与人工授权。换后端时比较排队时间、事件延迟、工件完整度和设备信息。
- 反证：若协议一致仍无法匹配 GPU、驱动和时钟条件，运行器可替换的假设不能支撑有效性能对照。先补硬件字段和错误状态。

### `https://github.com/OpenHands/software-agent-sdk/tree/69e26889401fe69157fff536e6a69049e6644cb3/examples/02_remote_agent_server`

- 原文：[固定提交的 remote Agent Server 示例目录](https://github.com/OpenHands/software-agent-sdk/tree/69e26889401fe69157fff536e6a69049e6644cb3/examples/02_remote_agent_server)
- 范围：目录 HTML 未返回内容。同一固定提交的 GitHub 文件接口返回 `01_convo_with_local_agent_server.py`、`04_convo_with_api_sandboxed_server.py`、`08_convo_with_apptainer_sandboxed_server.py`、`11_conversation_fork.py`。精读这四份源码。Docker 示例有独立登记。
- 机制：01 分离客户端会话 hook 和服务端动作或结束 hook。结束 hook 拒绝后，会话可从短暂 finished 回到 running。04 和 08 以 API 与 Apptainer 管 workspace，示例从 CI SHA 构造镜像标签，也有可变标签回退。11 复制服务端持久化事件和会话状态，未断言等量复制客户端短暂 WebSocket 事件或验证文件系统副本。
- 改变的动作：把会话结束、独立验收和 GPU 作业完成表示成各自事件。为两条优化假设分叉时固定同一提交与设备，各自使用独立工作目录、工件和 run ID。HPC 路径核对镜像 digest、设备可见性和宿主驱动。
- 反证：两个分支若共享修改过的文件，或客户端显示 finished 时远端仍在执行，必须依据作业与工件核对状态。示例展示 API，GPU 性能、隔离与恢复需要运行验证。

### `arxiv:2506.03011v1`

- 原文：[OpenHands-Versa v1 全文](https://arxiv.org/html/2506.03011v1)
- 范围：2025-06-03 v1，§§ 2–5、Table 1–3、附录中的工具与失败边界。论文基于 OpenHands v0.28.1。
- 机制：浏览器同时给 AXTree 和带控件 ID 的截图，只保留最近 k 个完整浏览观测。加入搜索 API 和多格式 Markdown 转换，每隔若干步提示重规划。用同一 Claude 3.7 Sonnet，GAIA 37.21%→51.16%，SWE-bench Multimodal 31.72%→31.33%，The Agent Company 完成率 26.29%→30.86%。GAIA 验证集上搜索 API 不同也会改变结果。案例出现盲信摘要、未运行测试和提前结束。
- 改变的动作：保留日志及计数器的结构化查询。图形时间线能补充信息时，返回附时间、设备、stream 或 NVTX 范围的可定位截图。用同一模型对比文本与文本加时间线，测瓶颈排序、回源率、token 和无效重跑。
- 反证：若视图只提升复述质量，或经常漏掉截图范围外的同步事件，优先补结构化查询。论文的对照改变了多种工具，整体收益无法归因于单个工具。

### `arxiv:2507.08149v1`

- 原文：[Code with Me or for Me? v1 全文](https://arxiv.org/html/2507.08149v1)
- 范围：2025-07-10 v1，§§ 3–5、Figures 3–4、任务与结果附录。读人类主动时间的估计方式和样本边界。
- 机制：20 名经常用 Copilot 的学生都使用两种工具，顺序随机，每种约 40 分钟。任务包括数据分析、加功能和修 bug。OpenHands 固定 Claude 3.7 Sonnet，Copilot 模型可选。作者报告任务正确率提高 `35% ± 15%`，`p=0.02`。对正确完成的任务，用户主动时间按作者定义由 25.1 分钟降到 12.5 分钟，Agent 条件墙钟约 27.9 分钟。Agent 主动时间用事件之间的人工输入窗口估计。Copilot 用总用时估计。
- 改变的动作：同时测工程师主动分钟、作业等待、总墙钟、重返任务成本、GPU 分钟和验收结果。让 Agent 接手标准环境准备和重复测量。把 workload 选择、解读和变更时机作为可比较的协作变量。试验控制模型、工具与实验条件。
- 反证：若节省的主动时间由审阅和返工吃回，或 GPU 成本上升且未完成更多有效实验，自动化没有达到预设收益。论文反映新手 Agent 用户、短任务与 2025 年春季工具，未测跨天协作和 GPU 调优。

### `https://github.com/OpenHands/docs/pull/797`

- 原文：[Agent Canvas 软件工厂文档 PR #797](https://github.com/OpenHands/docs/pull/797)
- 范围：已合并 PR 说明、文件清单及 `software-factory.mdx` 补丁。手册要求 Agent Canvas 1.24.0 或更高版本。作者的演示未独立运行。
- 机制：四个任务分别接收请求、实现、独立检查和观察交付状态。独立 reviewer 能写状态，不能推送改动。watchdog 检查待合并分支当前提交的验收与 CI 状态。分支因上游合并而过期时，更新后重新检查。作者报告六条演示 PR 都通过并合并，最后两条审阅在优雅重启后继续。
- 改变的动作：把优化候选、性能判定和采用记录绑定到同一 commit、镜像、设备和 workload。更新 base 或 benchmark 后让旧状态失效。GPU 任务的目标和发布决定由人把握，先测流程能否减少重复核对和重跑。
- 反证：若评审和开发共享可写工件、拿过期指标验收，或同一 run ID 在重启后重复计算，需要重新检查结果。作者只展示六条交付，缺复杂项目的人力与正确率对照。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-14/`

- 原文：[llm_learn L14 页面](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-14/)
- 状态：原件受限
- 范围：查课程简介、资料清单及中英文逐字稿面板。页面列 1 项资料，即官方讲义，内容产物 0 项。中英文逐字稿均显示尚未生成。
- 处理：讲义和可取得的正式录像画面各有独立笔记。该网页仅提供入口证据。若出现逐字稿，用讲义和录像时间点检查讲师口述和问答，再登记实读范围。

### `youtube:VAzZkT68h2Q`

- 原文：[L14 OpenHands 官方录像](https://www.youtube.com/watch?v=VAzZkT68h2Q&t=1800s)
- 状态：原件受限
- 范围：实看可取得的 storyboard 并核对缓存记录。30:00、30:40、31:10 的画面分别给出上下文管理、专门角色和单 Agent 基线。公开播放页提供的英文字幕请求此前返回空正文。全程音频未核验。
- 机制：画面先给旧事件压缩并保留原始指令与最近工作，再展示按角色拆分，随后给出定位、编辑和测试共享上下文的单 Agent 基线。
- 改变的动作：比较单 Agent 与专门 profiler 角色两路。测传递数值假设、复现环境和上一轮负结果需要的时间，以及遗漏造成的重复运行。
- 反证：角色增加而有效实验次数不变，且每次交接都重建背景时，分工收益没有出现。抽样画面只覆盖设计取舍，讲师对阈值、反例或问答的口述仍需验证。
