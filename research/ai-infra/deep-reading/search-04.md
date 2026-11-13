# L06、L20 深读 · 搜索成本、有效反馈与推理预算

本批细读六项登记来源。两条不带版本的 arXiv 链接分别解析到先前核查过的 v2 和 v4，本轮补读实验对照。PMLR 一项核对了出版 PDF 与作者预印本的重点部分。下列 AI Infra 试验为待验证的迁移假设。

### `https://proceedings.mlr.press/v306/gai26a.html`

- 原文：[Differential Smoothing，PMLR 记录](https://proceedings.mlr.press/v306/gai26a.html) · [出版 PDF](https://raw.githubusercontent.com/mlresearch/v306/main/assets/gai26a/gai26a.pdf) · [作者 arXiv v2](https://arxiv.org/html/2511.19942v2)
- 范围：出版 PDF §§2.4–2.5 的假设和目标、§3.1–3.2 的实现、§3.6 与 Figure 4、附录中四组题的熵对照。与 arXiv v2 的算法与主要实验交叉核对，两个版本章节编号有变。PDF SHA-256 前缀 `8d5fd1b77f65`。
- 机制：在有限轨迹、确定性 token 状态与二值终点奖励等条件下，作者用不同规则处理正负 advantage，尝试扩展通过验证的解法，同时减少高频错误。四组各 200 题的比较中，增加熵对 Pass@8 的变化分别为 −9%、−6%、+1%、+3.4%。变化方向与任务具有多少不同解法相关，作者在数学基准中也报告了采样成功率的变化。
- 改变的动作：先确定什么算独立的优化机制，例如减少重复访存与隐藏拷贝延迟。对每个机制设正确性门槛，再记录候选能通过哪些 workload 的验证。如果固定预算下某些 shape 或服务请求存在互补解法，可增加这些组的生成覆盖。只有证据显示重复候选是实际瓶颈时，才比较改变采样、提示或训练的收益。
- 推翻条件：固定提议数、评估次数和总时间，比较按机制挑选与直接按初步分数挑选。若新增多样性主要来自未通过正确性检查的修改，或独立 workload 上没有新增可用方案，先改善约束和验证器。数学题的终点奖励与性能收益的连续变化、噪声和成本差异限制了迁移。

### `https://platform.claude.com/docs/en/build-with-claude/extended-thinking`

- 原文：[Anthropic Extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)
- 范围：截至 2026-11-13 核对官方文档中的手动及自适应 thinking、interleaved thinking、计费、缓存和兼容性。未调用线上模型验证。
- 机制：手动 thinking 的 `budget_tokens` 是思考预算，通常和总输出 `max_tokens` 同时约束调用，interleaved thinking 有单独规则。文档列出的 Claude 4.7 及后续模型拒绝手动配置，适用范围取决于具体型号。调整 thinking 预算可使已有消息缓存失效。支持的模型可通过 `usage.output_tokens_details.thinking_tokens` 查看实际消耗。
- 改变的动作：给 profiler 结果判读、源码定位与实验计划分别设置样例。固定模型、版本、证据和工具结果，按缓存命中情况记录实际思考 token、首 token 延迟、总耗时、选择的实验和是否会触发无效 GPU 运行。先确认增加预算是否能改变后续动作，再判断适用任务。
- 推翻条件：若增加预算只有解释更长、相同实验或更慢的反馈，或实际收益由不同缓存状态造成，保留较便宜的设置。相同名称的预算参数无法直接作为跨供应商成本对照。

### `https://github.com/deepseek-ai/DeepSeek-R1`

- 原文：[DeepSeek-R1 官方 README](https://github.com/deepseek-ai/DeepSeek-R1)
- 范围：细读 `main` 的 README §§4、6、Usage Recommendations 与 Official Prompts。GitHub README blob SHA 为 `40d1af36fdb96e26129527b0531bb3ac195f6713`。未复跑评测。
- 机制：报告采样评估以 0.6 temperature、0.95 top_p、最多 32,768 生成 token 运行，并以每题 64 个响应估计 pass@1。这个指标估计单次采样的平均成功概率。部署建议给出的 temperature 区间为 0.5–0.7，还依赖模型权重、chat template 和不同蒸馏模型的配置。
- 改变的动作：把推理预算分成生成与选择两项。固定权重、模板、tokenizer 和推理引擎，比较一次生成、同预算的短候选及有验证器的候选筛选。分别记录有效下一步的比例、生成 token、TTFT、端到端时延和请求队列，标明推理 workload。
- 推翻条件：若只改预算时收益消失，回查模板或模型差异。若输出更长只提高口头解释，或排队成本超过少做的实验，保留较短设置。论文评测的 64 次采样与部署时的 64 选 1 要分别计量。

### `https://github.com/stanford-iris-lab/meta-harness/blob/main/reference_examples/text_classification/agents/fewshot_memory.py`

- 原文：[Meta-Harness 的 FewShotMemory 源码](https://github.com/stanford-iris-lab/meta-harness/blob/main/reference_examples/text_classification/agents/fewshot_memory.py#L92-L99)
- 范围：核对登记的 L92–L99，补读文件中的 prompt 构建、`predict`、`get_state` 和 `learn_from_batch`。GitHub 文件 blob SHA 为 `72a8e407df38e98b0beab1e664201b96eaefdc2f`。对照仓库的固定提示、全部示例和验证集流程。
- 机制：程序用输入的 SHA-256 初始化抽样，得到同一输入可复现的示例组合。默认最多抽取 50 例，prompt 仍受字符上限约束。代码中的 `MAX_CHARS` 为 30,000，文件说明给出 50,000。返回的 `num_examples` 是记忆条目数，实际进入 prompt 的数量还受抽样和截断影响。存样例时优先保留 `raw_question`。
- 改变的动作：用无历史与固定 few-shot 做基线。示例可存独立的 workload 条件、原始症状、下一项测量与结果。记录实际注入的示例 ID、字符数和 token，保留验证和最终测试的案例边界，再比较按 GPU、框架版本和症状检索。
- 推翻条件：若所谓记忆收益依赖测试案例或近重复问题，重新划分案例再试。若复杂检索与固定示例的命中、实验收益或维护时间无差异，保留固定示例。原任务是文本分类，不能据此判断性能诊断的多轮收益。

### `arxiv:2310.01798`

- 原文：[Large Language Models Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798)；[解析到的 v2 全文](https://arxiv.org/html/2310.01798v2)
- 范围：课程登记无版本链接，本次解析到 v2，与原有 v2 定向记录同版。补读 §§2–5 的自纠、直接提示与多回答对照，核对相关结果表。
- 机制：论文将只看到自身回答的内部纠正，与有额外反馈的纠正分开。在所用 GPT-4 GSM8K 设置中，原始准确率为 95.5%，两轮内部修订后分别为 91.5%、89%。若用正确答案决定停止，报告 97.5%，部署场景通常没有这种标签。在另一组 GPT-3.5 GSM8K 对照中，九个回复下 debate 为 83%，同预算 self-consistency 为 88.2%。
- 改变的动作：给性能根因判断设四组。第一组直接按已有不变量作答，第二组同预算独立生成并投票，第三组只修订上一轮解释，第四组按计划补一次可区分原因的测量。逐例记录初始判断与最后判断、是否得到新观测、改正数、改错数和成本。
- 推翻条件：若在独立 workload 上，无新观测的修订稳定减少错误，保留它并量化条件。若仅改变措辞或把初始正确判断改错，先改善首轮约束，再把资源投到有区分力的反馈。结果随模型、任务和提示而变。

### `arxiv:2407.01476`

- 原文：[Tree Search for Language Model Agents](https://arxiv.org/abs/2407.01476)；[解析到的 v4 全文](https://arxiv.org/html/2407.01476v4)
- 范围：课程登记无版本链接，本次解析到 v4，与原有 v4 定向记录同版。补读 §§3.1–3.3、§§4.1–4.2、§5.1、§5.4 和 Appendix A.4.1，核对 Table 2。
- 机制：候选动作先实际改变浏览器状态，再由独立价值模型给观测到的分支评分。回到先前分支需要重置环境并重放动作。在限制最终路径最多五步的 VisualWebArena 设置中，GPT-4o SoM 从 18.9% 到 26.4%，WebArena 从 15.0% 到 19.2%。搜索增加模型调用，论文另做较小样本的计算量对照。其成本取决于分支数、观测和状态恢复。
- 改变的动作：先在低成本可恢复的源码和 trace 分析中比较先看单一线索、并排给几个下一步排序、实际读取后回溯。用 commit、配置、run ID 与观测出处恢复同一个分析状态。对有资源费用或外部状态的实验，先在计划层筛选，记录单次真实分支的运行和恢复成本。
- 推翻条件：用同一结果口径记录模型调用、工具观测和实际耗时。若直接排序在同预算下选到相同的判别性测量，保留简单排序。若恢复后环境或 workload 已经变化，暂停利用该分支成绩比较方法。
