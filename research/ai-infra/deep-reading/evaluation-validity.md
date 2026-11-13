# L11 · 测试是否代表目标，奖励是否奖励了正确动作

### `https://openai.com/index/introducing-swe-bench-verified/`

- 原文：[Introducing SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/)
- 范围：Our Approach、Annotation Criteria、Annotation Results、Discussion & Limitations。
- 方法与证据：人工筛查 issue 是否明确、FAIL_TO_PASS 是否误拒正确解，并用容器减少环境差异。标注 1,699 项，保留 500 项。回归测试和新增行为测试需同时通过。发布时用原 SWE-bench 与 Verified 取得的分数受题集和环境同时影响。
- 工程用法：为开源框架优化案例先运行原实现与已知正确实现，验证 runner 能报回归、数值错误、编译失败和目标性能未达。隐藏用例保护泛化，公开验收目标与允许运行环境。若两种符合目标的实现中只有历史 PR 能过，修测试或补问题条件。

### `https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/`

- 原文：[Why SWE-bench Verified no longer measures frontier coding capabilities](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/)
- 范围：Background、Too narrow and too wide tests、Contamination、Discussion，核对 138 项审计的采样条件。
- 方法与证据：138 项是模型经常失败的子集，59.4% 指该子集有实质题目或测试问题。原文另测模型在受限信息下再现 issue 与 gold patch 的迹象。越能复现训练中可能见过的答案，越可能满足过窄的测试，二者会相互放大。
- 工程用法：留出未被调研或训练使用的后续项目。以可观察契约做 oracle，检查性能目标和跨 shape 回归。把「题不明确」「实现不对」「测试不对」「环境坏了」独立记录。对公开熟题可加新 workload、形状和新版本对照，分开报告熟题复现与独立工作表现。

### `arxiv:2512.07850`

- 原文：[SABER v1](https://arxiv.org/html/2512.07850v1)
- 范围：§§3–6、8，表 2–3，区分最早分歧的定义、日志回归与干预消融。
- 方法与证据：将改变外部状态的动作与读取动作分开。用动作计数相对成功参考的偏差做回归。它是轨迹级关联，不能给所有单次动作确定因果。系统增加变更前校验、定向反思和上下文清理。论文报告 Qwen3-Thinking 在指定 Airline 的相对增益为 28%，其用户校验依赖模拟和题集设定。
- 工程用法：从历史 GPU 任务前缀比较只读诊断、创建计算任务和修改部署。对费用或状态有影响的动作校验预期资源、当前运行 ID、版本及重复提交。用错误修改率、阻断合理动作率、额外人工等待和总完工时间评估。若诊断主要因日志缺失出错，先补证据，增加确认不会生成缺少的事实。

### `https://taubench.com/blog/tau3-task-fixes.html`

- 原文：[τ³-Bench · Fixing Airline + Retail](https://taubench.com/blog/tau3-task-fixes.html)
- 范围：错误 expected action、歧义、不可达约束、缺 fallback、修订前后 4 次试验与分数。
- 方法与证据：维护者改正任务、政策和评估状态之间的冲突。三个模型在修复后重跑，Airline 的 pass^1 增加 14.0–20.0 个点，Retail 为 −0.4 到 +5.5。这里改变的是题目和判据。同一任务四次全部成功的 pass^4 衡量重复成功，区别于任取一条最佳轨迹。
- 工程用法：为无可用 GPU、无法加载指定镜像和无法达到数值阈值写合法退出状态。先让两位评审以当时可见前缀判断合理动作。测四次能否复现和允许的接管路径。评分修订需单独对旧基线重跑。

### `https://lilianweng.github.io/posts/2024-11-28-reward-hacking/`

- 原文：[Lilian Weng · Reward Hacking in Reinforcement Learning](https://lilianweng.github.io/posts/2024-11-28-reward-hacking/)
- 范围：代理指标、RLHF 评估偏好与缓解章节。此条为课程登记的综述，实验细节分别回指其引用。
- 方法与限制：把任务目标通过环境、可见反馈与评估器变成代理信号时，格式、措辞、样本分布和评分器都能产生偏差。作者讨论用受控改写检查评分器对非目标特征的敏感性。不同案例的触发条件和防护证据不能合并成统一效果。
- 工程用法：为调优报告构造有相同数值结果和不同措辞的配对。再构造局部快而端到端慢、快速但数值错、全部样本平均快但关键 shape 慢的对照。评估器用独立测量和不可随候选修改的用例。若措辞仍改变排序，先修评审协议。

### `https://people.eecs.berkeley.edu/~russell/papers/icml99-shaping.pdf`

- 原文：[Ng、Harada、Russell · Policy Invariance Under Reward Transformations](https://people.eecs.berkeley.edu/~russell/papers/icml99-shaping.pdf)
- 范围：§3 定理 1、推论与路径上的折扣抵消，§4 的启发式。
- 方法与限制：在文中 MDP、折扣和终止假设下，把额外奖励写成 γΦ(s′)−Φ(s)，可保留最优策略。没有更多转移假设时，一般任意局部奖励无此保证。有限时域需处理终态势能，势能也应只由定义的状态决定。
- 工程用法：先按真实正确性、目标 workload 性能和成本验收。若加入「找到证据」等中间奖励，构造重复采样、反复读日志和绕远路的对照。没有可靠状态、折扣和终止约定时，可把中间指标保留作诊断，检查它是否使低价值长轨迹排名超过已经解决的轨迹。

### `https://michaelauli.github.io/papers/iclr2016_mixer.pdf`

- 原文：[MIXER · ICLR 2016 正式论文](https://michaelauli.github.io/papers/iclr2016_mixer.pdf)
- 范围：§3.2、§§4–5，核对 XENT 起点、逐步增加采样区间、greedy/beam 对照。
- 方法与证据：先用真值历史训练下一词。之后逐步将序列尾部交给模型采样，用 REINFORCE 优化序列级 BLEU 或 ROUGE，其余前缀保留交叉熵。指定 RNN 任务的 greedy 指标提升，部分设置接近 beam，但奖励仍是文本重合代理。
- 工程用法：先用有真实反馈的简短决策建立基本能力，再逐步扩大自选动作区间。分别测试遇到非预期编译输出、证据推翻先验、低质量数据时能否恢复。保持真实终局验收，避免把模仿某份日志的用词当作任务成功。

### `https://doi.org/10.1145/1553374.1553380`

- 原文：[ACM DOI](https://doi.org/10.1145/1553374.1553380) · [作者保存的 ICML 2009 正式论文](https://ronan.collobert.com/pub/matos/2009_curriculum_icml.pdf)
- 范围：出版链接返回 403。读同题、同作者和同会议信息的原论文 §§3–6，核对图形、玩具优化和语言模型。
- 方法与证据：定义随阶段变化的训练分布，将简单样本先提高权重，然后逐步回到目标分布。语言模型实验从 5k 高频词逐步扩到 20k，记录更新次数与留出排名。作者把优化路径变化视为假设，试验还包括图形分类和玩具设置。
- 工程用法：将「难度」定义为需要组合几类证据、未知约束和有代价动作的数目。先练单一症状再到多瓶颈案例。课程顺序、随机混合和按失败动态抽样用同标注预算对照。若简单样本引入错误捷径或最终长尾不改善，重设难度维度。

### `https://alignment.openai.com/prod-evals/`

- 原文：[OpenAI · Production Evaluations](https://alignment.openai.com/prod-evals/)
- 范围：数据处理、代表样本与目标采样、图 2–3、局限。只提取评估设计。
- 方法与证据：用经处理的实际交互前缀重采样，对代表样本估计发生率，对目标条件的富集样本做便宜回归。目标采样偏向已知失败，频率不能直接代入总体。监测器和场景会随模型变化，作者给出的检测结果仍受 judge 能力限制。
- 工程用法：对可用于调研的运行轨迹，单设代表性回放与高风险边界集。一个回答“多常出错”，另一个回答“修复后会不会复发”。用人工抽样复核分类，避免把少数精挑失败当作平均工作时间。

### `https://openai.com/index/where-the-goblins-came-from/`

- 原文：[OpenAI · Where the goblins came from](https://openai.com/index/where-the-goblins-came-from/)
- 范围：发布记录的分布、同题 reward 比较、训练曲线与 SFT 数据审计。
- 方法与证据：作者将稀有用词追到风格奖励。以同题样本对照，76.2% 的审计数据集对含指定词的输出显示正 uplift。在不带该风格提示的样本也观察到频率同向变化。模型输出再进入 SFT 可能继续强化该特征。这是单一生产案例的归因。
- 工程用法：若助手频繁选同一类 profiling 或追求长解释，按同任务原始证据比较评分与行为。检查对话格式、示范来源和模型自生成数据是否共用偏差。用不同 profiler、硬件、仓库和输出样式测试该动作是否只在有用时出现。
