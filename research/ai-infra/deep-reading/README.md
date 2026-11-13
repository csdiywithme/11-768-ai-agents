# 登记来源精读

本轮按原阅读索引的 755 个标识逐项推进。其中 634 个由课程引用、课件、作业、Project 或官方媒体登记，121 个来自此前的外部扩展。一个标识可能有多个 URL 和重复出现位置。课程明确标注的版本单独核对。已有摘要、同论文其它版本或入口记录保留在原索引，并在本轮另记实际读取范围。

截至 2026-11-13，755 个登记标识均有本轮记录。原件取得和阅读范围如下。

| 本轮状态 | 课程 | 扩展 | 合计 |
|---|---:|---:|---:|
| 正文重点已精读 | 565 | 105 | 670 |
| 替代版本精读 | 4 | 1 | 5 |
| 字幕文本已读 | 0 | 9 | 9 |
| 静帧已核 | 2 | 0 | 2 |
| 示例已核 | 15 | 0 | 15 |
| 仅核对入口 | 11 | 2 | 13 |
| 原件受限 | 37 | 4 | 41 |

正文状态按每项记载的章节、文件和图表计算。同一固定版本的不同登记项可共用一份阅读证据。字幕和静帧仅覆盖对应的文字或画面。原件受限条目注明已尝试的入口及可读旁证。

[逐项精读台账](../sources/deep-reading-ledger.jsonl) 每行对应原索引一个标识。保留其 URL、课程入口、原阅读状态和本轮状态。每条新记录链接到专题笔记及行号。待核查时按标识查找即可，现有交付的索引与历史结论仍可核对。

## 本轮采用的阅读口径

- 论文先核对版本，再读问题、方法、主要实验与限制，检查关键数字的基线和分母。优先验证会改变工程判断的段落。读过的章节、图表和仍缺的附录写在每项笔记。
- 文档和代码按登记的入口定位。保存版本、commit 或查阅日期，核对契约、实际实现、边界与对应测试。上游页面不能代替本地运行证据。
- 每条笔记回答一个能落地的判断。写出工作输入、可改变的动作、适用条件、最低成本的验证以及什么观察会改变判断。跨主题资料先放到同一个工程问题上对照。
- 录像、第三方字幕、原件访问受限、示例 URL 与可读正文分别记账。工具只返回入口或摘要时，回查作者或正式出版入口，记录取得的证据。原件和可替代版本分列。
- 沿原站点继续扩展。笔记记录链接从何处找到、实际读到哪一部分、与原资料有什么差别。新发现另记，既有 755 项的分母固定，便于核对是否遗漏。

## 阅读批次

| 批次 | 入口与内容 |
|---|---|
| L1–L2 | [工具调用、执行反馈和约束](tool-use.md) · [工具协议与实际执行](tool-contracts.md) · [模型工具格式与接口](tool-implementations.md) · [L2、L11、L12 和 L20 讲义](course-slides.md) · [L1 讲义与最小编码循环](search-12.md) · [L1 与 L14 的原始工程报告](search-13.md) · [L1 人员、组织入口和演示补查](search-14.md)。 |
| L3 | [上下文保真、注意力与缓存](context-memory.md) · [长上下文架构与检索](context-memory-02.md) · [课件、字幕、计费与压缩续跑](context-memory-03.md) · [模型实现、长程状态与检索](context-memory-04.md) · [运行时上下文与计算机使用接口](runtime-context.md)。 |
| L4 | [记忆、技能、反思和准入](memory.md) · [可复用工作流和代码技能](memory-02.md) · [其余课表原文与代码](memory-03.md) · [课件、字幕和页内引文](memory-04.md)。 |
| L5 | [分解、能力约束和重规划](planning.md) · [可执行计划、状态和错误传播](planning-followup.md) · [开放调查、回放和分工](planning-03.md) · [成本、选型和交互](planning-04.md) · [讲义、字幕和页内引文](planning-media.md)。 |
| L6、L20 | [候选生成、聚合和选择](search.md) · [代码评估和过滤](search-02.md) · [定位、验证器和连续改动](search-03.md) · [推理开销、实现和复查](search-04.md) · [L20 核心论文](search-course-readings.md) · [工程成本与训练语料](search-05.md) · [跨框架与观测](search-06.md) · [源码和变更接口](search-07.md) · [编译反馈与代码评估](search-08.md) · [L6 讲义与剩余书目](search-09.md)。 |
| L7 | [计算机使用与评估](computer-use-evaluation.md) · [视觉、状态及基准口径](computer-use-02.md) · [计算机使用与真实任务补查](computer-use-03.md) · [计算机使用工具、模型与样例补查](computer-use-04.md)。 |
| L8 | [SFT 数据、格式和纠错](sft-data.md) · [L8 训练报告与 L7 评估补查](search-10.md)。 |
| L9–L11 | [训练算法与评分](training-algorithms.md) · [L9 讲义、字幕与 REINFORCE](rl-foundations-media.md) · [研究与检索](research-retrieval.md) · [检索训练和版本对照](research-retrieval-02.md) · [示范与蒸馏](distillation.md) · [评估与奖励偏差](evaluation-validity.md) · [课堂引用与交互环境核对](research-retrieval-03.md)。 |
| L12、A3 | [推理调度、训练流水线与容量](training-systems.md) · [异步训练、运行时对齐与重复计算](training-systems-02.md) · [A3 API 和运行契约](a3-operations.md)。 |
| L13 | [隔离、凭据、恢复与观测](isolation-and-traces.md) · [防御评估与判据](defensive-evaluation.md) · [事件、评估与来源限制](defensive-followup.md)。 |
| L14 | [OpenHands 讲义、SDK 和固定源码](search-11.md) · [OpenHands 仓库、论文与录像核查](search-12.md) · [原始工程报告与实例](search-13.md) · [演示和早期项目入口](search-14.md)。 |
| L15–L17 | [持久化、执行恢复与评估](execution-and-eval.md) · [人工协作与真实工作评估](human-work.md) · [任务构造与人类参与](human-work-02.md) · [编排和监控](orchestration-02.md) · [真实工作与公开资源补查](human-work-03.md)。 |
| L18 | [多 Agent 协作与通信](multi-agent.md) · [协作状态、交接和仿真](multi-agent-02.md)。 |
| 作业、Project | [A1–A3 与 Project](coursework-and-project.md) · [教学 URL 和本地端点](course-examples.md) · [A1 与 A3 的接口、环境和实验记录](coursework-references.md)。 |
| 外部工程 | [GPU 性能、计时和候选搜索](gpu-performance.md) · [NVIDIA Dynamo、AIPerf、llm-d](serving-baselines.md) · [vLLM、NVSHMEM 和服务诊断](serving-diagnostics.md) · [新增同站来源](site-followups.md) · [候选优化、状态和版本对照](infra-research-followup.md) · [扩展原文和版本对照](external-evidence-02.md)。 |
| 来源核对 | [同一固定版本的登记对照](same-version.md) · [课件、原录像与第三方字幕](media-followup.md)。 |

## 如何用到当前 AI Infra 调研

[工作判断与验证对照](work-decisions.md) 将各批读到的方法放回真实的选题、定位、候选、执行、评估和训练决策。

**选择下一项实验。** 更准确地找文件或生成语法合法的动作，只验证了决策链中的一步。LocAgent 的定位提升没有等比例转化为最终修复。Zero-Shot Planners 的可执行性提高也伴随人工任务正确率下降。应把证据是否定位正确、动作是否能执行、是否针对目标、最终是否得到区分性结果分开判断。 [定位与验证](search-03.md) · [规划对照](planning.md)

**控制工作流的成本。** 复用经验、动态规划或多候选搜索都可能有效，也可能让模型重复已失效的动作。ReasoningBank 的更多记忆在一个子集上变差，AWM 的简单归纳在一个 WebArena 对照中与模型归纳相近。保留可恢复的真实状态、条件分支和简单提取作为基线。开放调查要在同样证据、工具与预算下表现出额外价值。 [记忆与反例](memory.md) · [流程归纳](memory-02.md) · [运行恢复](execution-and-eval.md)

**区分工作负载问题和引擎问题。** 对 vLLM 先查实际 batch、配额、KV 和引擎路径。对 DeepInfer 按代表性请求检查 acceptance、时间线和指标。Dynamo 的请求回放可复现结构、队列和缓存输入，文档默认的合成内容仍需要另行验证内容相关的接受率。当前最小验证继续比较新证据能否改变下一步，现成的服务调优闭环进入基线。 [推理和训练系统](training-systems.md) · [服务基线](serving-baselines.md) · [原工作机会与判据](../OPPORTUNITIES.md)
