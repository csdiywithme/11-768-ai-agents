# L10 精确版本和课堂材料与 L07 交互环境

### `arxiv:2511.19399v1`

- 原文：[DR Tulu，arXiv v1](https://arxiv.org/html/2511.19399v1)
- 范围：精读 §§3.1–3.3、6 的有限预算消融、§8 与附录 F.5。保留 v1 的章节号。v3 另列。
- 机制与证据：初始 rubric 依赖外部搜索，训练中根据多条 rollout 补充区分正负结果的项。评分后从 active buffer 移除组内零方差项，并按得分标准差保留有限数量。初始项持久保存。附录 F.5 的无 SFT 冷启动实验可获更高训练奖励，却有更低的下游分数并使用更多工具调用。
- 工程动作与反证：保留数值正确性、基线可比性等固定验收项。在开发集加入反复出现的失败项，如误把减少 host 同步当作 GPU kernel 加速。用独立留出集与工程师盲判检查变化。若增加规则只改善开发评分，或让正确但不同路径的诊断失分，重审该条规则。

### `https://akariasai.github.io/`

- 原文：[Akari Asai 研究主页](https://akariasai.github.io/)
- 范围：2026-11-13 可见版本，读研究方向和新闻。核对 2026-09-24 的课程分享记录与材料入口。
- 机制与证据：主页把主动获取知识、现实环境的可靠性和科研支持列为三条方向，分别给出 DR Tulu／AgentIR、When Not to Trust LMs／FAVA、OpenScholar／ScholarCatalyst 等入口。新闻把课程讲座与后续两个公开研究项目按日期列开。主页给出发现路径，实验细节归各原始材料。
- 工程动作与反证：沿讲者的论文索引寻找能改进候选生成、可靠性审计和多文档综合的方法。若链接只提供项目说明，就保留为发现记录，等原始正文核对后再纳入试验依据。

### `https://lti.cs.cmu.edu/people/faculty/asai-akari.html`

- 原文：[CMU LTI Akari Asai 院系页](https://lti.cs.cmu.edu/people/faculty/asai-akari.html)
- 状态：示例已核
- 范围：2026-11-13 核对院系页的姓名、任职和地点字段，对应 L10 pp.4–5 的单事实查找例子。
- 示例与证据：在主体和字段明确时，直接访问主管该信息的官方记录即可核对。院系页的更新日期与办公室访问权限需独立确认。
- 工程动作与反证：对已知符号的默认值先查对应 release 的 schema 或代码声明。若多个声明在不同构建选项下生效，打开调用点或实际运行配置，检查是否应进入跨工件诊断。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-10/`

- 原文：[llm_learn L10 学习笔记与解读](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-10/)
- 范围：2026-11-13 读课程笔记的学习目标、概念与待核项，并读解读的任务、评测和检索部分。它基于官方课件文本提取，网页的中文和英文逐字稿栏目均标尚未生成。
- 机制与证据：整理稿按搜索复杂度、领域专业性、答案质量和证据支撑对比基准，并标出课件的图、公式和未核课堂内容。解读给出“高分与专家对比”分开核对的例子，其结论需回到课件和论文所测人群与任务。
- 工程动作与反证：用导读定位需要检查的题型和机制，再取正式工件验证。如从摘要读到“吞吐升高”，回查指标定义、评估对象与测量条件。若链接指向缺页、图注歧义或不同论文版本，暂停该条因果推断并登记缺口。

### `youtube:nKUBrXFQBUM`

- 原文：[L10 官方录像约 25:00](https://www.youtube.com/watch?v=nKUBrXFQBUM&t=1500s)
- 状态：静帧已核
- 范围：播放页已有元数据为 CMU AI Agents 2026: 10. Deep Research，Graham Neubig 频道，时长 4,345 秒。目视读取官方 storyboard 的 24:40、24:50、25:00、25:10、25:20 静帧。图片 SHA-256 为 `081307beb3701273f16b65b4e8831abd24f3627cf93bb736223b3e6b93acedf7`。公开字幕请求缓存正文为空，本次没有连续音视频。
- 示例与证据：五帧都显示 Deep research agent inference。图上方随时间添加工具调用及返回块，下方的 token 片段也随阶段变化。可用它定位讲解所处阶段，具体文字和口述仍需原材料。
- 工程动作与反证：回放一次 profiler 引导诊断时，标记候选假设、下一次采样、工具结果和计划更新。若采样结果改变而下一条动作和阈值完全相同，检查回填是否影响了决策。

### `https://stanfordnlp.github.io/wge/`

- 原文：[Stanford NLP Workflow-Guided Exploration 项目页](https://stanfordnlp.github.io/wge/)
- 范围：读 Motivation and Setup、Framework Overview、Demonstrations 的图文和比较设置。核对 ICLR 2018 项目页所示 MiniWoB 任务。
- 机制与证据：少量示范构成不同粒度的工作流，工作流每一步限制可选动作集合，探索在约束内采样。工作流策略学习哪些约束更合适。页面把 10 个示范的 WGE 与 100／300／1,000 个示范的 BC+RL 放在若干示例任务中比较。
- 工程动作与反证：从调优示范提取阶段前置条件和可选动作。给新 trace 留出先查 occupancy、访存或调度的不同分支。若离开示范工作负载后有效动作被工作流排除，放宽对应阶段的选择并用新案例再测。

### `https://webshop-pnlp.github.io/`

- 原文：[Princeton WebShop 项目页](https://webshop-pnlp.github.io/)
- 范围：读 Abstract、Environment 状态与动作表、Trajectories 和仿真迁移说明。
- 机制与证据：以 118 万商品和 12,087 条人工指令建模拟购物环境。搜索和选择操作根据页面状态改变。奖励用程序检查商品属性、类型、选项和价格。项目页报告最佳模型完成率 29%、规则启发式 9.6%、人类专家 59%，均为该基准设置。
- 工程动作与反证：用合成任务让助手在已授权的 benchmark 库中按显卡型号、输入形状、精度和时限选配置。分别比较搜索成功、正确配置和真实运行结果。若候选满足可编码字段但无法编译或达到误差阈值，补执行检查。

### `https://jykoh.com/vwa`

- 原文：[VisualWebArena 项目页](https://jykoh.com/vwa)
- 范围：读 Example VWA Tasks、Execution-Based Evaluation、Set-of-Marks 与 Baseline 表。项目页可包含后续增补结果。
- 机制与证据：910 个视觉相关任务分布于 Classifieds、Shopping、Reddit。执行判据检查结果，Set-of-Marks 为可交互元素画编号和边框。项目页同列设置中 GPT-4V 的普通多模态输入为 15.05%，SoM 为 16.37%。Gemini-Pro 对应为 6.04% 和 5.71%，表明输入改动的收益依模型而变。
- 工程动作与反证：对 profiler 截图比较原图、原图加可交互编号、结构化时间线，检查事件选中和最终瓶颈判断。若编号提高点击命中却遮住重叠边界，改用更少标注或结构化事件并复测。

### `https://webarena.dev/og/`

- 原文：[WebArena 原版项目页](https://webarena.dev/og/)
- 范围：读环境介绍、跨站示例、Observation Space 与 Evaluating Functional Correctness。
- 机制与证据：可自托管站点模仿不同业务环境，任务可跨知识来源、地图和仓库。观察可选截图、HTML DOM 或 accessibility tree。信息题对比返回答案，执行题用程序检查运行过程中到达的状态属性。项目页示例要求先搜博物馆和地图路线，再修改指定 README。
- 工程动作与反证：在可重置测试环境准备“找到正确 workload，再写回测量摘要”的跨工件任务。分别核对实体定位、运行 ID、提交目标与原始数据引用。若相同语义的合法路径因为页面流程不同而被拒绝，修改判据来验证目标状态。

### `https://osworld-v1.xlang.ai/`

- 原文：[OSWorld v1 项目页](https://osworld-v1.xlang.ai/)
- 范围：读 Environment Infrastructure、Data Statistics 和 2025-07-28 benchmark notice。该页同时链接 Verified 与 2.0，读数需标版本。
- 机制与证据：原版公开 369 个任务，任务通过初始环境配置、交互、结束处理、结果工件和执行验证函数表达。8 个依赖外部云盘的任务可能需要手动配置，也可排除后按 361 道统计。页面将这两种口径都写明。
- 工程动作与反证：给运行 profiling 工件的任务集定义初始化失败、工具失败和完成失败三类事件。对资源不可达的任务保留总集和可执行子集各自的分母。若切换运行主机仅改变可执行率，先判断环境波动，再比较助手能力。

### `https://cmu-l3.github.io/gym-anything/`

- 原文：[CMU Gym-Anything 项目页](https://cmu-l3.github.io/gym-anything/)
- 范围：读 The Pipeline、CUA-World-Long leaderboard、Key Results 与 Gym API。论文版本另有记录，本条核对项目页的预算和任务入口。
- 机制与证据：安装、填充软件初始数据后由独立审计检查，再从运行过的种子任务扩充实例并形成 checklist。项目页 CUA-World-Long 的 Gemini 3 Flash 在 500 步 direct 下通过率 7.5%，2,000 步 direct 为 11.5%，同样步数加过程审计为 14.0%。标准条目另有费用限制，排名不能直接归因给模型。
- 工程动作与反证：为 Nsight 或日志查看器创建可重置样本。用同模型同预算比较定期审计与只在结束时审计。若审计以同等成功率消耗更多时间，测仅在关键状态异常时触发是否更合适。

### `https://github.com/xlang-ai/OSWorld/blob/main/evaluation_examples/examples/multi_apps/8e116af7-7db7-4e35-a68b-b0939c066c78.json`

- 原文：[OSWorld 跨应用任务配置](https://github.com/xlang-ai/OSWorld/blob/main/evaluation_examples/examples/multi_apps/8e116af7-7db7-4e35-a68b-b0939c066c78.json)
- 状态：示例已核
- 范围：核对公开 `main` 快照的 JSON、初始配置、任务结果和 `evaluator`，未运行 VM 或下载任务文件。[原始 JSON](https://raw.githubusercontent.com/xlang-ai/OSWorld/main/evaluation_examples/examples/multi_apps/8e116af7-7db7-4e35-a68b-b0939c066c78.json)
- 示例与证据：启动状态打开表格，输入包含图片和 PDF。验证器结束时激活窗口并保存，再用 `compare_table` 对照结果文件和参考。规则同时检查原有单元格范围、特定文本、部分数值的 0.1 近似容差。主动保存是 evaluator 行为，会影响对任务完成的解释。
- 工程动作与反证：给性能报告分别验证原始结果保留、关键字段、数值容差及文件是否在正常流程中写出。若后处理代替助手生成必需工件，增加前置完成检查并记录后处理的职责。
