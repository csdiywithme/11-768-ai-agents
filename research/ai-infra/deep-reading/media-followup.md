# L7、L8、L11 和 L12 · 转写与公开运行示例

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-07/`

- 原文：[L7 第三方英文转写与课件对照](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-07/)
- 状态：字幕文本已读
- 范围：读英文转写约 34:29–39:34、57:45–1:01:48。扩展此前的 34:29、57:45 两处。第三方英文转写和机器翻译可误写模型、术语及专名。官方视频与课件另记。
- 机制：长任务用状态检查、过程和最终结果提供不同反馈。Odysseys 示例有多项评分细则，允许部分分。OSWorld 2.0 强调专业软件、长作业与可验证工件。训练段分开截图定位、短动作、人类或合成演示和环境反馈。数据量、噪声与精确度的取舍不同。
- 改变的动作：调优评估分别记录阶段进展、运行约束和最终数值与性能结果。标注样本时区分识别 profiler 元素、下一步动作和完整决策。将来自不同 GPU 与 workload 的不确定性保留在样本里。
- 反证：如果高阶段分允许错误目标通过，修改评分细则。若短动作准确而长任务反复重跑，下一步训练或工具实验应覆盖状态和恢复。转写不用于确认具体榜单成绩。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-08/`

- 原文：[L8 第三方英文转写与课件对照](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-08/)
- 状态：字幕文本已读
- 范围：读英文 10:35–16:08、40:36–42:38、45:21–54:56。此前核对过 11 个时间点。官方 PDF、训练框架和论文各有独立证据。
- 机制：动作 token 产生训练损失，观测通常只作为输入。问答指出世界模型是可另做的对照，预测观测会占用模型容量。说话者比较单源与混合数据时固定模型、harness 和评估格式。另一段指出模板中即使空格和条件性块也可能改变前缀。部署后偏离演示轨迹时，训练 loss 不能预测恢复效果。
- 改变的动作：转换轨迹后检查每个工具调用、配对观测、EOS 和 token mask。保留 profiler、日志和人类约束为输入，决策和合法动作作目标。把同一轨迹不同序列化方式做成对照，并按 repo、GPU 或 workload 家族留出测试。
- 反证：若 loss 下降但跨工作区的重试和异常恢复退步，调整数据或目标。若差异来自训练与部署模板，先对齐再量化训练收益。讲者对 SFT 与 RL 的相对作用包含个人判断。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-11/`

- 原文：[L11 第三方整理页](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-11/)
- 状态：原件受限
- 范围：核对页面的中英文逐字稿面板，均未生成。页面的学习笔记和 Blog 解读基于官方讲义。读学习目标、稀疏终局、组平均示例和蒸馏总结，对应讲义 pp. 4–8、52–58。讲话、问答和音轨不在此证据范围。
- 机制：讲义整理区分同题候选的相对反馈、不同策略版本间的稳定更新、评分可靠性。组平均示例 1 成功 3 失败，四条轨迹都有相对信号。蒸馏段从逐动作分布增加反馈密度。整理稿记录排版或图表待核项。
- 改变的动作：把同题成功和失败轨迹共同保留。标注失败是判断错、GPU 资源耗尽还是验证器拒绝。先试有确定效果的短反馈，再比较相同 workload 下的候选策略。
- 反证：若组内所有轨迹同分，或评分误判主导，组内对照无法补偿奖励失真。数字、公式及异步效应以官方讲义及原论文为依据。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-12/`

- 原文：[L12 第三方整理页](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-12/)
- 状态：原件受限
- 范围：中英文逐字稿均未生成。读按讲义编写的训练与推理拆分、同步、token 轨迹和监控段，约对应官方 pp. 9–26、43。该页没有提供音轨、课堂问答或逐字稿。
- 机制：两侧的缓存、重算与并行优化依赖不同工作负载。异步 rollout 需要权重版本和 token 级一致性。观测建议同时看 rollout、update、stall、staleness 和 importance weight 等，识别是数据来不及、权重落后，还是训练和服务结果不一致。
- 改变的动作：为性能调优 Agent 的试验或训练按阶段计 GPU 时间，保留推理权重、tokenizer 与任务环境版本。比较调度变更前后的有效轨迹吞吐和独立质量，避免把模型指标下降误归于分配策略。
- 反证：若训练吞吐提高但结果重放在相同输入上不一致，检查版本和轨迹契约。讲义整理中的模型与数值只用于定位原文。

### `https://github.com/MinorJerry/WebVoyager/tree/main/results/examples/taskGitHub--0`

- 原文：[WebVoyager 的 GitHub 示例目录](https://github.com/MinorJerry/WebVoyager/tree/main/results/examples/taskGitHub--0)，[对应消息记录](https://github.com/MinorJerry/WebVoyager/blob/main/results/examples/taskGitHub--0/interact_messages.json)
- 状态：示例已核
- 范围：核对目录内一份消息 JSON 和五个 PNG 文件名，读 JSON 的完整消息，blob `d2940295602c60318e9d513705cf24684b21c4dd`。未查看原始 PNG 或重跑动态 GitHub 任务。
- 机制：每轮给可交互元素标签，轨迹从普通搜索到排序后才按观察报告结果。标签绑定当前观察。早期截图在消息记录中省略，后续图片是占位，原 PNG 另存。日志可回放动作意图与排序条件，不能验证所有像素或搜索结果完整性。
- 改变的动作：用 profiler 或服务面板时，把每次点击或查询绑定截图 ID、指标过滤器和时间范围。优先从明确排序或结构化查询定位热点，再把观测绑定运行工件。
- 反证：如果 UI 更新后标签仍引用旧截图，或者只看到首屏就判断全局热点，重读当前可见状态并独立确认。旧轨迹的排序结果不等于当前实时结果。

### `https://github.com/trycua/cua`

- 原文：[Cua 仓库](https://github.com/trycua/cua)，[Cua Driver Calculator 示例](https://cua.ai/docs/cua-driver/quickstart)
- 范围：`main` README 及官方 Calculator quickstart，README blob `2fd153f7bbd28e26c69dc67bd436393b0e6b6337`。读当前包职责、隔离环境与操作验证。未执行安装或示例。
- 机制：Driver 为 Agent 提供程序化桌面操作。Lume 管本地虚拟机。SDK 管 sandbox 并允许使用自己的 harness，Cua Bench 支持任务和轨迹。Calculator 教程要求由界面状态复核动作结果。无抢占焦点操作依赖平台与应用支持。
- 改变的动作：把只存在 UI 的 Nsight、平台面板或实验 dashboard 作为可替换观测。写同一任务的 API/CLI 和 UI 两路，比较定位、可重放、人工接手和不同平台错误率。
- 反证：UI 通过但无法从原始数据证明区间和设备时，增加独立查询。README 和简单演示无法证明 GPU 工具支持或隔离性能。

### `https://cua-gym.xlang.ai/`

- 原文：[CUA-Gym 项目入口](https://cua-gym.xlang.ai/)，[作者代码仓库](https://github.com/xlang-ai/CUA-Gym)，[arXiv v2](https://arxiv.org/abs/2605.25624v2)
- 状态：原件受限
- 范围：项目 URL 超时，常规 index 路径不可读。作者实验室的 OSWorld 2.0 站确认链接，补查 CUA-Gym 仓库的任务生成、筛选和环境说明，并用 v2 摘要对照。论文阅读另有独立登记。
- 机制：共同生成任务说明、初始与目标环境以及评分器。由不同角色产生状态和奖励，执行后进行多轮筛选。v2 摘要和 README 正文报 32,112 个、110 环境。README 表另写 32,122，两个数保留差异。
- 改变的动作：以真实硬件和框架实验为样本，固定初始与验收状态，另设 scorer，验证至少一个失败与一个成功例。对性能阈值附近和不允许的状态变化做对照。
- 反证：如果评分通过但改动未触发目标 kernel，或使用不匹配硬件，评分器需要复核。项目入口未能提供额外证据。样本数以具体版本数据集核验为准。
