# 深度调研如何分辨找不到资料和读错资料

## 从有固定答案的搜索到开放式判断

复杂调研需要多次查找和综合证据。一个回答可以表述得很像参考答案，却把「在特定条件下成立」写成「普遍成立」。L10 用准确性、覆盖、适用条件和证据支持分别衡量这类结果。DeepResearch Bench 的 FACT 把报告拆成「主张—URL」对，逐对判断原页能否支持主张。ResearchQA 的作者还审查 rubric 本身。在其专家审查中，纯模型生成的 rubric 有 15% 属于空泛、有误或不清楚的项目。这个比例来自作者设置。对 Infra 的启示是先核对评估条目，再用条目评答案。 [L10 p.35–49](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=35) · [DeepResearch Bench v1 §3.2、附录 E](https://arxiv.org/html/2506.11763v1#S3.SS2) · [ResearchQA v2 §3.2](https://arxiv.org/html/2509.00496v2#S3.SS2)

把「图里有正确数字」与「结论有证据」分开。B300 评估案例可以附一个真实测量和一个条件不同的测量。给评分者留原始计时范围、设备与分母。把答对数字但选错 baseline、把 NCU replay 写成墙钟收益当作不同失败。对 DeepInfer，引用到组件结果页只满足定位，原页是否支持端到端收益还要单独判。 [B300 NCU 对照口径](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/experiments/m4_lowprec_20261022/NCU_ANALYSIS.md:3) · [DeepInfer 组件测量边界](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:15)

## 先给出正确证据，再比较检索

BrowseComp-Plus 用固定语料、人核验的支持文档和困难负样本减少搜索环境变化。在 §4.8.1 的 oracle 条件中，模型收到已标注的所有支持文档。gpt-4.1 的准确率从 BM25 条件的 14.58% 到 oracle 的 93.49%。这比较的是该实验的完整搜索条件与证据直给条件，也同时改变了搜索路径。另一个模型在 oracle 条件中仍有文档超出上下文的问题。对个人 Infra 案例，先固定一个材料快照、少量确切支持片段和几个近似但不适用的来源，再比较两种输入条件。 [L10 p.21–30](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=21) · [BrowseComp-Plus v1 §3.2、§4.8.1](https://arxiv.org/html/2508.06600v1#S4.SS8.SSS1)

- **普通检索条件**。给相同助手、任务、工具、预算与冻结的案例目录。记录找到的证据、来源位置、第一次误判和最终判定。
- **证据直给条件**。给同一个助手经人工核验的关键片段。保持其他条件可比。能给出的证据包括实际配置与测试结果，不提供最终答案。分别评分证据获取、判断准确性、漏掉的验证项和人工纠正。
- **结果分流**。直给明显改善，优先比较索引或证据收集脚本。直给后仍缺测量，优先补可执行工具或测试。信息完整、动作可行后仍有稳定的判断错误，再评估结构化提示、人工示范或训练数据。

这个对照是待验证方案。它能先排查「找不到」和「拿到也不会用」。如果资料缺失本身就是被考察的能力，保留缺失并把下一步实验计划列入验收。vLLM 的固定教学回放可提供状态分歧片段，仍要把真实上游测试的状态标成待核验。 [vLLM 离线回放与测试边界](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:15)

## 多轮检索的输入和训练信号

AgentIR 将当轮搜索意图与 query 共同输入 retriever，用中间信息需求构建训练对。其 §5.1 拆开输入改变与训练数据改变。§5.2–5.3 对照加入旧历史的方式。在该实验中，直接纳入更多旧查询、旧推断或旧文档未必有益。已被排除的猜测和无关片段会污染检索。迁移时可先试明确的检索字段，例如「当前要验证的问题」「固定的 commit、模型和硬件」「已排除的候选」。可比较只有 query、query 加这些字段和 query 加完整日志摘要三组。按是否取回正确版本和是否解决当前证据缺口评估。论文实验针对 web 问答与对应模型，当前没有验证对本地代码与日志的收益。 [L10 p.100–124](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=100) · [AgentIR v1 §3.3、§5.1–5.3](https://arxiv.org/html/2603.04384v1#S5)

L10 把短答案 exact match 与开放报告 rubric 分开。它还说明如何从有出处的事实合成题目，并从 teacher 输出筛出可用轨迹。对 Infra 可先给一个已核验事实链设置不同的输入状态，检验是否能找对证据和保留条件。所有源于同一底层案例的改写应归在同一数据分组。DR Tulu 用搜索到的证据和候选回答更新训练 rubric。其作者报告跨运行的最高训练奖励未必对应最好下游成绩。个人工作可先保存按版本审查过的 rubric 和失败样本。训练条目如需调整，仍需用固定的独立评估检查收益。 [L10 p.58–98](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=58) · [DR Tulu v1 §3.2、§6.2、§8、附录 F.5](https://arxiv.org/html/2511.19399v1#S8)
