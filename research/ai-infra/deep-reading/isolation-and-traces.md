# L13–L15 · 运行隔离、身份、恢复和观测

### `https://www.cmu-agents.com/slides/lecture-13-agent-safety.pdf`

- 原文：[L13 · Agent Safety](https://www.cmu-agents.com/slides/lecture-13-agent-safety.pdf)
- 范围：43 页官方缓存，重点 pp.18–29 的隔离与身份、pp.30–42 的监控与评估。SHA-256 为 `6a1efa5cf21ab696750710c6aa77cba640b9da548b91eba2188001a7e64e611c`。
- 方法与限制：运行环境约束文件、网络、进程和保留状态。凭据应关联调用方与任务权限。监控对执行前拒绝和执行后发现有不同作用。评估需要覆盖合法任务和错误动作，课件表中计数是示意。
- 工程用法：同一个历史任务分别重放只读、隔离工作区写入和外部运行三类动作。比较正确完成、错误写入、额外人工、遗漏异常和重试成本。隔离可减少错误影响，监控保留实际状态与结果以判别何时需要接管。

### `https://docs.docker.com/engine/security/`

- 原文：[Docker Engine security](https://docs.docker.com/engine/security/)
- 范围：namespace、control group、daemon、capabilities、签名和内核额外限制。2026-11-13。
- 方法与限制：容器共用 host kernel，namespace 隔离可见性，cgroup 约束资源。daemon 与容器进程是不同权限边界。镜像来源、挂载、网络和能力配置会影响实际隔离。
- 工程用法：把 GPU runner 的项目目录、缓存、环境变量、设备、网络与权限写入实际启动契约。不同作用域的运行用等价小任务测环境可用性和资源可控性。比较隔离开销时固定实际容器镜像和 GPU 资源。

### `https://docs.e2b.dev/sandbox/persistence`

- 原文：[E2B Sandbox persistence](https://docs.e2b.dev/sandbox/persistence)
- 范围：状态迁移、完整与仅文件系统保存、暂停被拒绝、恢复与终止。
- 方法与限制：完整 pause 保存沙箱文件与内存，filesystem 模式从磁盘重新启动。一次 pause 可因上次快照仍在处理而返回忙，sandbox 继续运行。文档提示错误类型的区域上线和 SDK 版本。外部系统的运行状态不属于该快照承诺。
- 工程用法：按 sandbox ID 与外部 job ID 分别接回。恢复前核对是仍在运行、暂停、重启，还是已终止。用暂停拒绝与回复丢失的测试检查是否会重复提交 GPU 任务，另记重新 warm-up 时间和计费区间。

### `https://docs.e2b.dev/sandbox`

- 原文：[E2B Sandbox lifecycle](https://docs.e2b.dev/sandbox)
- 范围：创建、连接、超时设置、到期动作、暂停与终止。核对 runtime timeout 与 stream 说明。
- 方法与限制：sandbox lifetime 和客户端的一条流不是一个超时。到期可以暂停保留状态或按配置结束。恢复读取的状态依赖此前选择和服务计划，不能从 stream 超时推出进程已经终止。
- 工程用法：遇到日志暂时中断，先查 sandbox 与 job 当前状态，再查产物。用三种干预模拟 stream 断开、VM 停止和外部任务继续。只有任务确实终止且没有有效产物时，按预算再提交。

### `https://apptainer.org/docs/user/main/quick_start.html`

- 原文：[Apptainer Quick Start](https://apptainer.org/docs/user/main/quick_start.html)
- 范围：CLI、SIF、镜像来源、文件访问和 host/container 集成说明。2026-11-13。
- 方法与限制：面向 HPC 的容器执行保留便捷的主机 I/O 访问。SIF 可携带可复用运行环境。文件挂载、用户身份和系统配置会影响可见性。便携镜像没有自动给出独立 VM 边界。
- 工程用法：共享 GPU 集群先记录用户身份、镜像摘要、必需挂载和数据目录。用只读挂载及输出路径清单验证能否读取依赖并落盘。若文件或网络隔离需求高，按同一计算任务比较该平台实际支持的隔离环境。

### `https://www.qemu.org/docs/master/system/introduction.html`

- 原文：[QEMU System Emulation · Introduction](https://www.qemu.org/docs/master/system/introduction.html)
- 范围：virtualisation accelerators、machine/CPU 架构与设备模型、VirtIO 和设备直通。
- 方法与限制：QEMU 提供虚拟机设备模型，可选硬件加速或 TCG 模拟。VirtIO、offload 和设备直通改变 host/guest 交互及性能。支持取决于主机和架构。
- 工程用法：CPU 验证和 GPU 性能实验可以用不同运行环境，但要单独定义证据。判断 VM 上编译通过能否替代 GPU 实测时，核对目标指令、真实设备、driver、设备呈现、host 计时。需要真机效应的断言要另跑。

### `https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens`

- 原文：[GitHub Docs · Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- 范围：个人 token 与 GitHub App 用途、fine-grained 的 owner、repository 和权限、classic 限制和有效期。
- 方法与限制：PAT 代表个人，权限受用户能力和 token 范围共同约束。fine-grained 能限定 owner、仓库与权限，支持范围受组织规则和功能限制。
- 工程用法：上游资料读取和提交修改应有可区分授权。对本地诊断无需写仓库的工作选择只读路径。审计记录凭据来源与范围的标识，不记录凭据本身。若只读可取完整信息，就能用同题评估更宽权限是否带来额外价值。

### `https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app`

- 原文：[GitHub Docs · Installation access tokens](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app)
- 范围：安装身份、仓库和权限缩限、过期及新 token 格式提示。2026-11-13。
- 方法与限制：凭据表示特定安装。签发时可在安装允许的范围内缩小仓库和权限。SDK 可以管理刷新。文档提醒 token 格式分阶段切换，因此长度不能作为稳定接口。
- 工程用法：把允许的 repo、API 操作、审批关联和结果写入任务状态。遇到过期先查原写操作是否已落地再续跑。以返回权限和 API 成功验证有效性，避免记录 token 或按固定字符数作身份判断。

### `https://www.rfc-editor.org/rfc/rfc8693.html`

- 原文：[IETF RFC 8693 · OAuth 2.0 Token Exchange](https://www.rfc-editor.org/rfc/rfc8693.html)
- 范围：§§1.1、2.1.1、2.2、4.1、6，区分 subject、actor、resource、audience、scope 与部署策略。
- 方法与限制：委托可保留调用者与被代表者，冒用语义下后端主要看到被代表的主体。标准给交换语义与字段，实际授权策略、密钥管理、期限和撤销关系依赖部署。
- 工程用法：若后续接入远端 GPU 或部署工具，保留发起者、执行者、目标受众和范围，再决定授权读取、修改或收集产物。恢复时核对当前权限与原任务意图。先做 actor/subject 能否区分的回放测试。

### `https://github.com/NVIDIA/OpenShell/blob/ec49209da25be39840742df29b64ec694d159c2f/docs/about/architecture.mdx`

- 原文：[NVIDIA OpenShell architecture 固定 commit](https://github.com/NVIDIA/OpenShell/blob/ec49209da25be39840742df29b64ec694d159c2f/docs/about/architecture.mdx)
- 范围：组件分工、inside boundary、mediated channel、四种 runtime、policy 与 credential provider。
- 方法与限制：runtime 建立隔离，可信 supervisor 做策略和凭据决策，gateway 交付策略与审计。Docker、Kubernetes、MicroVM 使用不同底层机制。文档是所固定架构说明，不能代替本地策略回归。
- 工程用法：工具执行入口可按相同任务比较三种权限作用域，测正常 bench 是否可完成、意外写入能否阻止及恢复能否继续。把策略决策、远端实际动作与缓存状态分别记录。模型的请求描述不能用来证明权限已经执行。

### `https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/`

- 原文：[OpenTelemetry Gen AI Semantic Attributes Registry](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- 范围：1.45.0 注册表，conversation、tool call ID、model、usage 与 content 字段。页面将若干字段标为迁至独立 GenAI 仓库。
- 方法与限制：有 tool call ID、名称、输入、结果和 token usage 等独立属性。内容字段可含敏感数据。schema 版本与字段采样应核对采集端，不能据文档假设所有 span 都保留同样字段。
- 工程用法：需要解释下一步为何选 profiler 时，关联 run、工具调用、外部 job 与产物 ID。常规观测保留版本和状态，回放必要内容使用已授权的有限样本。抽样检查两条有相似时间的请求是否可正确归属。

### `https://transluce.org/docent/blog/introducing-docent`

- 原文：[Transluce · Introducing Docent](https://transluce.org/docent/blog/introducing-docent)
- 范围：环境问题发现、分析样例、分析流程及人工核验示例。
- 方法与证据：作者从轨迹定位缺包后修复环境。在其 InterCode 设置中 GPT-4o solve rate 由 68.6% 到 78%。这是环境介入后的差异，不能算成模型能力提升。
- 工程用法：全组失败时自动聚类报错可辅助找到缺包、镜像或权限问题。取原始日志复核，再以同任务重跑。只改环境就恢复时，记录为环境修复，后续训练数据需区分这类失败和错误决策。

### `https://transluce.org/docent`

- 原文：[Transluce · Docent](https://transluce.org/docent)
- 范围：产品页四步流程，ingest、rubric、spot-check、量化。2026-11-13。
- 方法与限制：可把单 Agent 和多 Agent 文本轨迹转为可检索集合，围绕 rubric 识别行为并定位原文证据。初始标签和匹配需抽样校对。实际运行状态要靠埋点输入。
- 工程用法：定义可观察行为，例如“worker 仍在运行时又提交”或“报告把 profiled time 写作最终计时”。从日志验证关联，再估计发生率。若缺乏负例分母或 judge 把合理重试误报，就收窄标签并保留人工一致性。

### `https://cline.bot/blog/post-mortem-unauthorized-cline-cli-npm`

- 原文：[Cline 官方事后报告](https://cline.bot/blog/post-mortem-unauthorized-cline-cli-npm)
- 范围：影响面、时间线、发布恢复和长期防护。记录工程防御与证据边界。
- 方法与证据：官方记录一次未经授权的 npm 发布，约 8 小时暴露窗口。影响范围按分发渠道和发布版本核对。撤销凭据、下架版本和重建发布身份属于不同恢复动作。
- 工程用法：将处理外部 issue 的工作流与发布镜像的执行身份分开。在性能实验里记录实际拉取镜像摘要、来源、发布作业、依赖与运行入口。发现异常发布时先查受影响部署和原始清单，避免只看版本字符串。

### `https://openai.github.io/openai-agents-python/`

- 原文：[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) · [Running agents](https://openai.github.io/openai-agents-python/running_agents/) · [Handoffs](https://openai.github.io/openai-agents-python/handoffs/) · [Tracing](https://openai.github.io/openai-agents-python/tracing/) · [Human-in-the-loop](https://openai.github.io/openai-agents-python/human_in_the_loop/)
- 范围：2026-11-13 官方 SDK 入口及运行循环、交接过滤、RunState 暂停恢复、trace 数据选择。未运行示例。
- 方法与限制：runner 循环到最终输出、交接或处理工具结果。交接元数据、可见历史和应用状态来自不同接口。RunState 可保存暂停位置，外部计算生命周期仍需由应用关联。trace 配置决定是否采集完整生成与工具内容。
- 工程用法：以一个定向诊断任务对照固定步骤和 SDK 循环。传入 commit、当前假设、证据与原始 job ID，模拟暂停后重新接入以及另一专家接管。验证是否保留正确证据、能否复核工具作用域、是否重复启动，最后比较人工介入和总时延。
