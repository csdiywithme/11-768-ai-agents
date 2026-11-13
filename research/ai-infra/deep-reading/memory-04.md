# L4 课件、字幕和课件补充引用

核验日期 2026-11-13。课件、字幕、作业与文档的实际读取范围分别记录。前三批核对了 L4 全部课表 readings 和 references。本批补四个课件页内引用、课件和第三方英文字幕，单列官方录像的访问限制。

### `https://www.cmu-agents.com/slides/lecture-04-memory-and-skills.pdf`

- 原文：[L4 官方 PDF](https://www.cmu-agents.com/slides/lecture-04-memory-and-skills.pdf)
- 范围：课程缓存 PDF 46 页。扫描全部可提取文字，重点读 pp.2–6、8–15、16–20、22–36、38–45。用原论文核对 pp.12–13、33、38–43 的指标和版本。未逐张查看所有图。PDF SHA-256 为 `15165b0ab74f7ebe2704bbfbc341f2273d27e66041f3bfedf2c887c02e6a549d`。
- 机制与条件：课件将一次任务完整轨迹、可更新事实、可复用操作分别处理。上下文保真但成本高。外部产物可审查、可修改，需要选取和维护。改权重更新成本高，且难以看出是哪项经验造成变化。课件还把技能的学到、准入、选择、执行和维护作为不同环节。代码技能节省重复模型动作，可独立测，但受接口变化影响。文字技能提供更多调整空间。
- 来源校验：p.33 的图按达到的检查点和外层动作步数评估，数值应按 ASI v2 §4 Table 4 的定义理解。p.43 的 SAGE 表用 scenario goal completion，和整任务完成率分开。p.39 的失败轨迹结论限于 ReasoningBank 特定子集及记忆表示。
- 工程动作与反证：将可恢复的本次状态、旧实验原始证据、跨任务经验和可调用诊断分开保存。对同一性能工单，依次测找证据、选择动作、运行和判定结果。用新项目及不适用的形状检查规则是否过度触发。若固定接口已经表达全部需要的步骤，比较维护时间后再决定是否加入语言模型。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-04/`

- 原文：[L4 第三方英文自动字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-04/)
- 范围：逐段检查网页所载英文部分 64 段，从首段 00:01.520 到末段起点 1:14:04.960。网页说明字幕由自动字幕整理并合并连续段，时间戳为每段起点。HTML SHA-256 为 `c9d3715973f5e8f268a49c8111eeb2be1d8b8db71708dee9eec7b69371fef6e6`。用官方课件与固定版本论文复核技术表述。
- 方法与证据：04:47–11:35 区分窗口内、外部材料、权重更新与保存内容。17:28–25:14 讨论技能索引、发现误差与按需展开。28:15–32:14 讨论无技能、人工整理技能和线上修订。33:48–43:57 对比外部记忆更新、同任务反思、整条示范。48:07–56:25 对比顺序任务和文本与代码。59:19–1:04:09 讨论验证代码、减少交互和跨站接口变化。1:06:08–1:14:04 讨论从失败抽取策略、判分误差、检索数量与训练。
- 工程动作与反证：把「能否发现、载入、执行和验收」分开观察。按工具版本和成本判断走哪条流程。在同一决策回放给无关技能、正确技能和可直接调用工具，检查有无过度执行。口述近似值及个别控件类型，以论文和课件明确版本复核。
- 状态：字幕文本已读

### `youtube:6zigF2a-2Pw`

- 原文：[L4 官方录像入口](https://www.youtube.com/watch?v=6zigF2a-2Pw&list=PLSN0qpDfUvTM&index=4)
- 范围：核对课程索引和官方播放列表中的录像入口。本次网页读取返回 Cache miss，未获得可播放画面、音轨或可核验的官方字幕。第三方字幕另有独立记录。
- 状态：仅核对入口

### `https://agentskills.io/specification`

- 原文：[Agent Skills Specification](https://agentskills.io/specification)
- 范围：读取当前规范的目录结构、frontmatter、可选资源、Progressive disclosure、File references 和 Validation。
- 方法与边界：技能目录至少含带名称、说明和正文的 `SKILL.md`。发现用名称和说明，激活读正文，资源在需要时读。名称等字段有格式约束，`compatibility` 可描述要求，`allowed-tools` 是实验字段，支持随宿主变化。示例验证器检查格式与命名。
- 工程动作与反证：把适用版本、GPU、profiler 以及必需输入写在能及时发现的位置。将长字段表和判别例分到独立引用文件。格式验证后用宿主实际读一次，并用旧版和无 profiler 两种情况验证脚本的拒绝或退回。若目录合规但模型总加载错技能，缩窄说明或提供确定性入口。

### `https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills`

- 原文：[Anthropic Equipping agents with Agent Skills](https://www.anthropic.com/research/equipping-agents-for-the-real-world-with-agent-skills)
- 范围：L4 p.11。读登记 URL 的当前重定向正文，核对 Anatomy、Skills and code execution、编写与评估章节。
- 方法与边界：名称和说明先用于发现。正文选择后读入。长资源继续按需读取，确定性计算可以运行独立脚本并返回结果。作者建议从代表性任务暴露的能力缺口开始，观察过度依赖和错误加载。文章为实现说明，缺独立的对照结果。
- 工程动作与反证：对 trace 数据先用程序输出带来源、单位与分母的短结果，再按需追原记录。给模型两份同名、不同 schema 的报告，测短结果是否足以发现冲突。若误判源于缺读数，扩展结果字段。若流程仍被错用，收窄触发或用确定性接口。

### `https://github.com/cmu-agents/assignment-1/blob/main/ASSIGNMENT.md`

- 原文：[A1 官方作业说明主分支](https://github.com/cmu-agents/assignment-1/blob/main/ASSIGNMENT.md)
- 范围：L4 p.11。下载官方仓库当前主分支讲义的文本用于比对。与本地课程提交 `67498d87e36f7b45a2a4ce82d65d1139fb6a51c0` 的同文件字节一致。SHA-256 为 `20886a35b3b348aaf6e1e992d38d6d4f3c4ba86bdb3e2fbe7943f0384ee00995`。本项核对开头、§1.4、Part 2、Grading 与 Submission。未执行课程测试。
- 方法与证据：发现子目录中的 `SKILL.md`，解析元数据和正文。缺失、格式错误和重名有明确处理。向模型先展示目录项，调用时再给正文。压缩只影响活动上下文，原始调用记录独立。评分把技能调用、patch 回放、保持行为和产物完整度作为不同证据。
- 工程动作与反证：对 AI Infra 助手测发现、加载、动作和报告四段。构造重名、旧版本、缺依赖和未生成原始结果四类测试。在未运行昂贵实验时，保留判断及未满足的前置条件。若加载成功但对照或可追溯证据缺失，该工具只通过接口验收。

### `https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/`

- 原文：[Hermes Agent Skills System](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills)
- 范围：L4 p.14。读当前重定向文档的 Using Skills、Progressive Disclosure、SKILL.md Format、External Skill Directories 及更新会话的说明。与本次读取的两份主分支源文件选段对照。
- 方法与边界：索引、正文、引用文件分别加载。正文可声明工具与平台前提。当前文档描述会话期间的技能变化先用提示通知，系统提示在下次压缩更新，以便维持缓存。共享目录如果可写，技能维护可能修改其内容。
- 工程动作与反证：在长性能任务中保留当前技能路径及版本，启动测试时记录配置指纹。模拟更新后同一会话继续，核查读到的是当前规则，历史报告仍可追到旧规则。若主提示延迟刷新导致错误引用，先用清晰的路径与版本字段校正。
