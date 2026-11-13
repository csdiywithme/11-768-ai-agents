# L1、L6 和 L14 · 剩余入口、研究线索与第三方转写

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-01/`

- 原文：[L1 英文自动转写与讲义对照](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-01/)
- 状态：字幕文本已读
- 范围：登记是第三方转写与整理页。读英文约 19:30–26:40、35:02–41:22、47:00–49:39 和 52:00–59:32。覆盖以前只摘到的 38:15、59:32。官方录像另记。识别错误依靠讲义、课程文字和论文校正。
- 机制：课堂问答在 21:51–22:46 比较代码表示和 JSON 工具调用。36:48–41:22 讨论先在 harness 中解问题，再训练可复用行为，也举出长上下文效率、任务定制和小模型延迟。52 分钟后把模型推理、缓存、执行环境和轨迹观测落到系统组件，最后连接可执行评估和训练环境。
- 改变的动作：用相同模型比较结构化 profiler 查询、可组合代码查询和原始文本。测有效发现数、GPU 成本、输入 token 和人能否接手。记录通过修改接口消除的错误，再把仍反复出现的跨任务判断错作为 SFT 或技能实验输入。
- 反证：若只改输出 schema 就修复错误，模型训练的对照需剔除这部分。如果小模型减少推理开销却需要更多验证或 GPU 重跑，单次推理延迟无法代表工作效率。课堂给出设计取舍，量级需要实验。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-06/`

- 原文：[L6 英文自动转写与讲义对照](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-06/)
- 状态：字幕文本已读
- 范围：第三方英文约 30:34–40:13、1:00:59–1:09:21，扩展原登记两处时间点。课程讲义、官方录像和方法论文另记。口语识别会误写论文名。
- 机制：代码相似度适合查相似实现，执行检查允许语义相同的不同代码。用例不足会放过错误，隐含规格会拒绝合理实现，依赖漂移会改变结果。全仓库测试成本高，所以拆目标测试与回归测试。后段比较历史 PR 和代码变异产生的训练任务，并指出不同 harness 的动作格式会改变表现。
- 改变的动作：评估算子时把可运行起始提交、依赖和测试命令一起冻结。使用最小快速用例反馈，完整测试留到候选入选时。对性能评估器做刻意变化，如错选实际 kernel 或改变 workload 构成，观察其能否发现偏差。真实案例和受控变体都保留各自来源。
- 反证：若人工制造的缺陷没有覆盖真实瓶颈分布，训练收益不能代表工作收益。若依赖或 runner 变化就改变旧 patch 的结果，先校准再比较 Agent。课堂转写不是论文的复现实验。

### `https://x.com/summeryue0/status/2025774069124399363`

- 原文：[Summer Yue 原帖](https://x.com/summeryue0/status/2025774069124399363)，[保存的当事人陈述](https://feeds.simonwillison.net/2026/Feb/23/summer-yue/)
- 状态：原件受限
- 范围：原帖和归档入口无可读正文。课程 L1 p. 3 核验作者与月份。转引页保存了当事人的叙述、截图描述和关于 compaction 的后续解释。
- 机制：当事人说真实输入触发上下文压缩后，建议任务变成了实际批量操作，聊天停止消息没有及时中止。压缩丢掉授权约束是其解释。执行日志和运行版本未独立取得。
- 改变的动作：在 AI Infra 工具层独立保存执行许可、目标范围和预算。模拟模型上下文丢失或切换后，检查只读调研能否保持只读。停止操作须能观察已排队和在跑作业是否取消。
- 反证：若压缩前后操作授权完全一致且能及时停止，重点再转回状态同步和工件清理。该陈述没有提供事故频率或完整根因调查。

### `https://www.youtube-nocookie.com/channel/UCHWq8J_nvQqugBTH19ILCEA`

- 原文：[L1 嵌入播放器的频道链接](https://www.youtube-nocookie.com/channel/UCHWq8J_nvQqugBTH19ILCEA)
- 状态：原件受限
- 范围：课程 L1 p. 6 的嵌入播放器关联频道。该 URL 和同频道 ID 的 YouTube 地址未提供频道内容。课程页把对应视频列为 Graham Neubig 的 OpenHands 视觉调试。
- 处理：这是频道入口元数据。具体视频 `jMyTCXpEz10` 另记，截图能力的实证引用 Versa 论文。

### `https://adityasoni9998.github.io/`

- 原文：[Aditya Soni 学术主页](https://adityasoni9998.github.io/)
- 状态：仅核对入口
- 范围：L1 p. 7 教学团队入口。读研究方向、新闻和选列论文。OpenHands-Versa 研究和代码只按已有独立笔记计。
- 线索：主页连接代码 Agent、通用工具与 Agent 可靠性研究。对 AI Infra 值得保留的实验变量是工具组合对跨任务表现的影响。Versa 同模型结果在 GAIA、SWE-bench Multimodal 和工作环境任务上方向有差别，因此用本地调优、上游框架和跨服务诊断各自验证。

### `https://andyjliu.github.io/`

- 原文：[Andy Liu 学术主页](https://andyjliu.github.io/)
- 状态：仅核对入口
- 范围：L1 p. 7 教学团队入口。读研究问题和所列近期工作。主页提供人机协作、价值对齐和多 Agent 监督的研究方向。
- 线索：将工程师改目标、补背景或否决假设记为协作动作。比较请求次数与在高成本 GPU 运行前成功消除歧义的数量。主页没有给该场景的对照，暂作研究问题。

### `https://apga.github.io/`

- 原文：[Apurva Gandhi 学术主页](https://apga.github.io/)，[共同作者的 ASI 论文入口](https://arxiv.org/abs/2504.06821)
- 状态：仅核对入口
- 范围：L1 p. 7 教学团队入口。读主页的 Agent、RL、抽象和推理时扩展方向。核对相关作者的程序化技能研究入口，未把论文入口另算为主页证据。
- 线索：从重复的工具动作中提出小而可调用的流程。以探查归一化、测试 shape 或按 stream 找阻塞为单位，验证一次后再复用。比较通用工具、文本技能和可执行技能对新硬件与异常路径的影响。

### `https://d3mi.wang/`

- 原文：[Demi Wang 学术主页](https://d3mi.wang/)，[共同作者论文](https://openreview.net/attachment?id=GmCoFYNEIU&name=pdf)
- 范围：L1 p. 7 教学团队入口。核对主页的代码原生技能研究，补查论文方法、Tables 2–4、A.1 污染案例。主页和论文不重复计结果。
- 机制：用可调用 Python 函数表示技能，把解题、验证和更新拆开。更新前需要独立验证者同意。104 题子集上代码 Agent 无技能宏平均 60.8%，加完整方案 67.2%。GitLab 域下降，未经验证的技能会污染后续任务。
- 改变的动作：可执行 GPU profiler 技能发布前，给验证者相同输入、原始计数器和独立参考实现。隐藏生成者的结论。记录规则在哪种设备和 workload 可重用，以及原始失败。
- 反证：若小范围通过后仍污染其他 GPU、shape 或工具版本，收紧选择条件。论文为网页环境，迁移后的指标需重测。

### `https://dpfried.github.io/`

- 原文：[Daniel Fried 学术主页](https://dpfried.github.io/)，[Behavioral Efficiency 讲座](https://dpfried.github.io/talks/efficient-agents.pdf)
- 范围：L1 p. 7 教学团队入口。读主页的 LARC、授课和报告索引。补查 52 页报告 pp. 8–10、31–32 的协作和技能示例。
- 线索：重复任务通过共享背景缩短描述，也可把共同动作提成程序化技能。比较把启动、采集和定向读取 trace 整理成重用动作后，能否减少重复调用，同时保留接手者所需的关键环境信息。报告示例来自沟通与网页任务，性能工程采用自己的对照。

### `https://jiarui-liu.github.io/`

- 原文：[Jiarui Liu 学术主页](https://jiarui-liu.github.io/)
- 状态：仅核对入口
- 范围：L1 p. 7 教学团队入口。读研究方向、课程身份、研究项目说明。主页提出用 harness 与 RL 支持研究构思，未提供该项目对照。
- 线索：评估候选生成时记下实验排除了什么原因。一个未改善的候选仍可能减少后续搜索范围。比较带负结果的轨迹与只保留成功 patch 的轨迹，对新 workload 假设和实验选择的影响。

### `https://saujasv.github.io/`

- 原文：[Saujas Vaduguru 学术主页](https://saujasv.github.io/)，[共同作者的协作论文](https://arxiv.org/html/2510.24023)
- 范围：L1 p. 7 教学团队入口。读通信与交互相关项目，补查合作论文 v1 §3 的角色任务和基线。
- 线索：实验在重复的参考游戏中观察共享上下文如何帮助压缩表述。迁移到性能项目时，用短 ID 引用 workload、设备与采样条件，但接手者要能解析并核验。测短交接的时间收益及会话重启后的参数错配率。

### `https://sunnweiwei.github.io/`

- 原文：[Weiwei Sun 学术主页](https://sunnweiwei.github.io/)，[共同作者的 TraceML](https://arxiv.org/html/2608.26086)
- 范围：L1 p. 7 教学团队入口。读长期 Agent、Context-Folding、开放研究、协作主题。补查 TraceML §§3、4.3，已有登记的 Context-Folding 独立计。
- 机制：TraceML 把人和 Agent 的提交映射成带分数和时间戳的代码序列，区分恢复先前的分数与返回先前的方案。该研究观察到人会恢复旧思路，所测两个系统几乎没有。样本和任务预算限定结论。
- 改变的动作：给优化分支存下改动、数据范围、负结果和返回条件。比较有无旧分支检索时，性能 Agent 是否在新证据下回头重测曾被放弃的候选。
- 反证：如果回看只是重复跑旧实验，没有结合新测量提高判断准确率，分支历史的检索成本无法收回。

### `https://www.phontron.com/`

- 原文：[Graham Neubig 学术主页](https://www.phontron.com/)，[CMU 官方个人资料](https://www.lti.cs.cmu.edu/people/faculty/neubig-graham.html)，[作者软件清单](https://www.phontron.com/software)
- 状态：仅核对入口
- 范围：L1 p. 7 教学团队入口。根页面抓取正文为空，搜索索引保留主页摘要。以 CMU 官方资料和同站软件清单确认身份与 OpenHands 入口。
- 线索：研究主页提供语言、代码生成和评估的索引。OpenHands 的机制证据引用对应论文、固定源码及独立评测。个人主页只承担身份和研究入口核验。

### `https://yueqis.github.io/`

- 原文：[Yueqi Song 学术主页](https://yueqis.github.io/)，[作者的 Agent Data Protocol 项目页](https://www.agentdataprotocol.com/)
- 范围：L1 p. 7 教学团队入口。核对 Agent Data Protocol 项目链接，补查作者页面的动作、观测、转换与质量说明。论文单列阅读。
- 线索：用通用事件格式做一次转换，减少不同轨迹源到不同训练框架的适配。给 GPU 实验额外保留时间、设备、软件版本、工件和验收字段。用同一个训练样本转换前后检查动作和观测对齐，避免简化格式丢失导致判断变化的证据。

### `https://fireworks.ai/`

- 原文：[Fireworks AI 官网](https://fireworks.ai/)，[官方 Prompt Caching 文档](https://docs.fireworks.ai/guides/prompt-caching)
- 范围：L1 p. 8 计算赞助商入口。读当前推理及训练产品简介，沿官方文档补查复用条件与副本路由。
- 机制：文档说明共享前缀可在同一 replica 内复用。多个副本时可以用用户或会话标识给路由提示。是否命中受前缀、队列和路由影响。主页性能宣传没有用来推断自己的 workload。
- 改变的动作：给长 Agent 会话测追加、压缩、改变工具和并发分叉四种前缀。分别记录 cached tokens、TTFT、P95、每个有效实验的成本，并测路由集中后的热点。
- 反证：若命中率上升但排队变长，P95 或整个调优完成时间可能恶化。先查逐请求的 cache 与排队，再决定是否把行为做进 harness。

### `https://sail.computer/`

- 原文：[Sail Research 官网](https://sail.computer/)，[Sailboxes 官方页面](https://sail.computer/sailboxes)
- 范围：L1 p. 8 计算赞助商入口。读长程推理与工作区产品说明，并核对 Sailbox 的独立磁盘、暂停、恢复、fork 和连接存活条件。
- 机制：产品页描述持久 Linux VM 与独立磁盘。暂停、恢复和分叉针对 VM 状态。供应商说明打开的连接最多保留约 10 分钟，并区分活动与保留资源。用户样例及成本数字是供应商描述。
- 改变的动作：对于跨数小时的框架调优，测试本地上下文与运行状态分别能否恢复。固定代码、缓存和工件 ID，模拟等待推理、断开和恢复。记录在跑任务、连接和远端作业的状态。
- 反证：VM 恢复后若 scheduler job、设备会话或外部连接已失效，需要单独重连或确认作业。文档未建立 GPU profiling 的同等运行条件。

### `https://www.primeintellect.ai/`

- 原文：[Prime Intellect 官网](https://www.primeintellect.ai/)，[Verifiers v1 作者架构文档](https://www.primeintellect.ai/blog/verifiers-v1)
- 范围：L1 p. 8 计算赞助商入口。读计算、训练、推理和评估介绍，补查作者对 v1 taskset、harness、runtime 与轨迹的技术拆分。具体版本与课程采用环境未验证。
- 机制：taskset 定义任务与评分，harness 负责编排，runtime 负责隔离与资源生命周期。v1 允许兼容接口下替换各层，轨迹允许分支和压缩并可携带训练字段。
- 改变的动作：把同一批调优任务用于固定工作流、Agent 和训练数据方案。先固定 taskset、设备与评分，换 harness 检查收益。再固定 harness 对比训练策略，记录运行失败是否改变轨迹分布。
- 反证：若 runtime 交换带来不同编译、GPU 或超时条件，比较需要先恢复同等条件。官方架构接口说明不证明所有组合实测可用。

### `https://x.com/JustinLin610/status/1767769034186211557`

- 原文：[Junyang Lin 原始 callout](https://x.com/JustinLin610/status/1767769034186211557)，[OpenHands 项目论文](https://arxiv.org/abs/2407.16741)
- 状态：原件受限
- 范围：原帖和嵌入请求均未取得正文。L14 p. 8 核验课件将它标为 OpenDevin 的原始 callout。早期项目报告、作者身份和后续论文能提供历史背景。
- 处理：保留课件标注，原帖内容与项目发起的具体因果仍待核。可迁移设计引用另记的 CodeAct 1.0、OpenHands 论文与源码。

### `https://x.com/gneubig/status/1772735837479792761`

- 原文：[Graham Neubig 早期前端原帖](https://x.com/gneubig/status/1772735837479792761)
- 状态：原件受限
- 范围：原帖和嵌入请求均未取得正文。L14 p. 8 标注早期前端。课件同页描述 2024 年 3 月的前端、Agent 和 Docker sandbox 组合。
- 处理：保留课堂时间线。界面、运行器职责和用户操作应引用已经读到的报告、仓库和文档。原帖图像与上下文待核。

### `https://www.youtube-nocookie.com/channel/UCk_Il3HoK3qTz1tvzyphozg`

- 原文：[L14 嵌入播放器的频道链接](https://www.youtube-nocookie.com/channel/UCk_Il3HoK3qTz1tvzyphozg)，[同 ID 的 YouTube 频道](https://www.youtube.com/channel/UCk_Il3HoK3qTz1tvzyphozg)
- 状态：原件受限
- 范围：L14 p. 7 的嵌入频道链接未直接返回内容，同频道 ID 的 YouTube 页面显示 Cognition 标题与通用页脚。
- 处理：只核验频道入口。Devin 发布视频和同期作者公告有独立笔记。

### `https://oss-agent-canvas.ngrok.dev/`

- 原文：[L14 OpenHands OSS automation 演示入口](https://oss-agent-canvas.ngrok.dev/)
- 状态：原件受限
- 范围：入口无法抓取。L14 p. 40 的课件说明示范检查触发、仓库和权限、输入事件到运行再到输出，并要求对照原始任务判断输出。没有可读运行记录。
- 改变的动作：将该课件的可观察顺序用作性能实验演示。以任务请求、固定输入、run ID、结果和验收为追踪路径。先验证输入确实进入预期设备和 runner。OpenHands #797 提供独立案例。
- 反证：若存在输出但无法定位触发事件、运行设备或 workload，演示不能支持工作流成功结论。链接是演示入口，未将其计成实测数据。
