# L06 深读 · 生成约束、编译反馈与行为预测

本批细读十项登记来源。重点是工具产生的观测、候选筛选和便宜代理指标的适用范围。下列 AI Infra 应用为待验证的假设。

### `https://proceedings.mlr.press/v119/yasunaga20a.html`

- 原文：[DrRepair 出版记录](https://proceedings.mlr.press/v119/yasunaga20a.html)，[PMLR 正文](https://proceedings.mlr.press/v119/yasunaga20a/yasunaga20a.pdf)
- 范围：ICML 2020 出版 PDF。细读 §§2–4 与 Tables 1、3–4，核对编译反馈、人工扰动与完整修复的定义。
- 机制：把编译器诊断里的符号和程序中的同名符号连起来，辅助定位根因。训练数据由可编译程序经有目的的扰动产生，再用真实编译反馈监督修复。在 DeepFix 的完整修复指标上，有编译信息的基础模型 62.5%，加图后 66.4%，再预训练后 68.2%。样例集中在入门 C/C++ 的编译问题，单步模型用首条诊断定位并修一行。
- 改变的动作：处理 CUDA 模板或 Triton lowering 的失败时，让助手同时看第一条有效诊断、报错位置、符号定义和实例化来源。按真实故障类型构造可复查的失败样例。编译通过之后再运行数值正确性和性能验收。
- 推翻条件：比较原日志、提取首条诊断、诊断加符号关系。若只减少日志长度却增加错误的根因定位，保留完整上下文。若大部分失败来自环境配置，优先检查版本和依赖。

### `arxiv:2207.01780`

- 原文：[CodeRL: Mastering Code Generation through Pretrained Models and Deep Reinforcement Learning](https://arxiv.org/html/2207.01780v3)
- 范围：arXiv v3。细读 §§3.3.1–3.3.4、§§4.3–4.5、§5，核对 Tables 3–4 的训练与采样消融。
- 机制：以编译错误、运行错误、测试失败和测试通过四类信号监督 critic。推理时使用公开样例测试和 critic 决定继续生成、保留前缀或修复。APPS 的 Table 4 中，基础设置 pass@1000 为 17.78%，同时启用 refine 和单个 repair 候选为 20.98%，其 1@1000 从 7.71% 到 8.48%。这个单项结果仍依赖候选过滤。
- 改变的动作：对算子优化先编译，再做廉价正确性检查，最后进入固定环境的测量。用失败类型决定重试方向，比较原候选修复与抽一个不同机制。生成与评价分别计成本。
- 推翻条件：若候选池中存在通过全部验收的方案，但筛选常选错，优先改善选择和测试。若同成本的简单过滤足够，保留简单方法。APPS 的单测结果不能证明目标 workload 的性能收益。

### `https://aclanthology.org/2023.findings-acl.308/`

- 原文：[Code Execution with Pre-trained Language Models](https://aclanthology.org/2023.findings-acl.308/)，[ACL 正文](https://aclanthology.org/2023.findings-acl.308.pdf)
- 范围：ACL Findings 2023。细读 §§3–5、§§6.1–6.2，核对 Tables 2–4 的数据、训练课程和追踪口径。
- 机制：真实 Python 解释器产生逐步的变量状态，模型学习根据代码和输入预测输出与轨迹。课程由单行操作、教程和变异的题库逐步组成。CodeNetMut 上输出准确率 48.06%，完整轨迹 33.38%。去掉训练课程后分别是 45.93%、30.98%。简单题的准确率较高。
- 改变的动作：标注 runtime 状态时，先记录容易验证的输入、选定分支和状态变化。再加入并发请求、批处理和调度交互。让模型的预测先形成可验证的假设，保留真实 trace 作为裁决。
- 推翻条件：若训练只改善单条函数执行，却不能定位调度状态中的首个分歧，补多请求的轨迹。若静态断言和短重放能更便宜地给出状态，保留程序检查。Python 逐行状态不能直接类比 GPU 并行和非确定执行。

### `https://aclanthology.org/P17-1041/`

- 原文：[A Syntactic Neural Model for General-Purpose Code Generation](https://aclanthology.org/P17-1041/)，[ACL 正文](https://aclanthology.org/P17-1041.pdf)
- 范围：ACL 2017。细读 §§3–4、§§5.1–5.3 与结果脚注，核对 Tables 2–3。
- 机制：先按语法产生式扩展 AST，再生成或复制终端变量和常量。语法缩小可选动作，单子节点规则合并可减少动作，但也可能增大规则空间。Django 片段基准的准确率为 71.6%，移除复制终端机制后 32.3%。其准确率基于与参考匹配，论文明确指出语义等价代码可能被漏计。
- 改变的动作：在实验配置和工具输入中先约束字段与可选动作，将模型自由生成集中在假设和候选值。GPU 名、kernel 名、单位和 dtype 可从实际环境引用。对生成的代码继续验证真实形状与设备。
- 推翻条件：若约束提高格式通过率却漏掉有效方案，检查 schema 是否少了支持的配置。若更复杂的语法使实现难维护，保留少量高价值约束。论文中的早期 Python 代码生成准确率不适用于当前模型。

### `arxiv:2009.10297`

- 原文：[CodeBLEU: a Method for Automatic Evaluation of Code Synthesis](https://arxiv.org/html/2009.10297v2)
- 范围：arXiv v2。细读 §§2–4.4，核对 Tables 2、4–5 的任务及分析单位。
- 机制：将普通和加权 n-gram、语法树匹配和数据流匹配加权。实验用 Java 与 C# 任务，对比少数系统的平均分与人工评分。代码细化任务里，CodeBLEU 与人工评分的相关系数为 0.979，BLEU 为 0.923，精确匹配为 0.999。不同任务的最高单项和组合变化。
- 改变的动作：若候选改动很多，结构和数据流相似性可用于分组近重复，或查是否与一类参考实现一致。仍给数值、内存和时延单独的验收。对不同算法的正确实现保留多种参考。
- 推翻条件：用结构相近但结果错误，以及结构不同但正确更快的候选检查排序。若分数不反映工程决策或筛掉有价值方案，保留为分析标签。系统级相关不能用于单补丁的验收。

### `https://aclanthology.org/2023.emnlp-main.859/`

- 原文：[CodeBERTScore: Evaluating Code Generation with Pretrained Models of Code](https://aclanthology.org/2023.emnlp-main.859/)，[ACL 正文](https://aclanthology.org/2023.emnlp-main.859.pdf)
- 范围：EMNLP 2023。细读方法、实验、分析与 Limitations，核对 Tables 1–2。
- 机制：模型联合编码需求上下文与参考、候选代码，再用代码 token 的向量相似度评分。正文的评分阶段去除上下文 token 和多数非字母数字符号。CoNaLa 人偏好实验的 Kendall tau 为 0.517，chrF 为 0.470。另用四种语言 HumanEval 的执行结果检查相关性。指标依赖参考、编码器和算力。
- 改变的动作：若需要筛选待人工评审的训练样例，先比较人工简查、结构规则和嵌入打分。将 workload 需求与改动共同评估，检查向量排序有没有帮助挑出对比价值高的案例。
- 推翻条件：若高分候选仍经常违反同步、索引或精度约束，增加明确的程序检查。若打分消耗的时间和资源超过少做的人工评审，保留简单方式。文中相关性不能代替独立执行或性能验收。

### `arxiv:2405.15383`

- 原文：[Generating Code World Models with Large Language Models Guided by Monte Carlo Tree Search](https://arxiv.org/html/2405.15383v2)
- 范围：arXiv v2。细读 §§3–4.1、§§5.1–5.3、§6.1 与 Appendix C 的动作消融，核对 Table 1 的 pass@20。
- 机制：把环境的状态转移和奖励写成可执行程序。搜索结点含部分程序及用于评价的完整版本。根据观测选择继续生成、用执行反馈改善预测或修复程序。APPS 的 Table 1 给定同为 Llama 3 70B、pass@20，零样本 CoT 为 23.2%，WorldCoder 为 25.1%，GIF-MCTS 为 28.3%。搜索过程已经使用测试评分，论文限制了该结果的 pass@1 解释。
- 改变的动作：为排队、batch 形成或 cache 变化建立一个小型可计算近似。用已有 trace 检查下一状态和约束，再让模型在其上提出便宜的测量。真实 GPU 只用于能改变工程决定的候选。
- 推翻条件：若近似在独立负载上无法预测瓶颈方向，或在关键边界失准，缩小其使用范围。该论文主要依赖确定、可完全观察的环境。GPU 实验的并发、外部干扰和噪声需用真实观测确认。

### `arxiv:2510.02387`

- 原文：[CWM: An Open-Weights LLM for Research on Code Generation with World Models](https://arxiv.org/html/2510.02387v1)
- 范围：arXiv v1。细读 §§2.1–2.3、§§7.1–7.3、§8.3，核对 Table 4 的中训消融与推理筛选。
- 机制：数据包括可执行仓库、Python 真实轨迹和多步的仓库交互。8B 中训消融里，PR 数据上增加 tracing，CruxEval-O 从 44.6 到 73.9，SFT 后 SWE-bench Verified 从 18.6 到 18.4。再加入 ForagerAgent 交互数据，后者到 22.1。全模型的推理扩展另外生成候选和测试，再筛选提交。
- 改变的动作：把「根据 trace 预测下一状态」和「能选择下一项有效实验」做成独立任务。数据分别记录原始观测、工具动作与经独立复核的结果。先测离线判读和真实任务上的变化，再决定是否需要训练数据。
- 推翻条件：若 trace 预测改善但实验成功率或人的判断不变，把轨迹示例用在分析训练。若推理筛选成本超过收益，减少候选。来源所释模型面向非商业研究，作者未验证生产表现。

### `arxiv:2407.01489`

- 原文：[Agentless: Demystifying LLM-based Software Engineering Agents](https://arxiv.org/html/2407.01489v2)
- 范围：课程无版本链接解析到 v2，与已有定向核查同版。补读 §§3.1–3.3、§§5.1–5.2，核对 Tables 2–4 和公开复现测试的局限。
- 机制：固定三阶段定位、修复、验证。SWE-bench Lite 中，300 项有 213 个生成测试能报告复现，应用参考补丁后仅 94 个能正确报告解决。筛选消融从单独投票的 77 项，增加回归测试到 81 项，再加复现测试到 96 项。生成测试也会增加成本。
- 改变的动作：把固定流水线作为自主工作方式的对照。对现有错误，先证明用例能区别原版和一条已知正确实现；对性能优化，用已知差异或插入的变体检查度量敏感性。结果需要现有回归继续通过。
- 推翻条件：若复现用例会误判已知正确实现，不用它排除候选。若固定步骤在证据变化时重复无效实验，增加明确的状态分支，再比较总耗时和验收率。参考补丁仅作离线测试校验。

### `https://mini-swe-agent.com/latest/`

- 原文：[mini-SWE-agent 概览](https://mini-swe-agent.com/latest/)，[控制流](https://mini-swe-agent.com/latest/advanced/control_flow/)，[v2 迁移说明](https://mini-swe-agent.com/latest/advanced/v2_migration/)
- 范围：截至 2026-11-13，细读登记概览，并对照同站控制流示例与 v2 迁移指南。未运行项目。
- 机制：项目把历史顺序追加，工具动作在独立进程中执行。概览仍有无需工具调用接口的描述，v2 迁移说明明确默认使用 native tool calling，可配置为文本解析。控制流示例分别记录步骤、费用、时限及格式错误，循环每步保存。跨模型的消息结构可能不同。
- 改变的动作：建立一个足以跑固定工作样例的简单对照，保存任务、工具输入输出、退出状态和预算。给持续运行的 GPU 任务保留独立 run ID 与路径，避免依赖临时 shell 的状态。固定实际版本和消息格式。
- 推翻条件：若最小流程经常丢失重要运行状态，再增加明确状态与恢复入口。若换工具接口的表现变化来自解析失败，先修复适配。概览与迁移页的差异应保留在版本记录。
