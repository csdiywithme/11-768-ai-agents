# L10 延伸阅读｜检索命中、证据可读性与主张核验

## OpenScholar 的正式论文与预印本

- **版本与分数。** L10 p.6–11 用 OpenScholar 说明多来源文献综合。正式论文表 1 与 2024 年预印本 §4.2 表 2 的 Scholar-CS rubric 分数相同。OpenScholar-8B 为 51.1，无检索的 GPT-4o 为 45.0，PaperQA2 为 45.6。前者分别高 6.1 和 5.5 分。用表中同列读数比较。预印本摘要中的 5% 和 7% 与这个表格口径对不上。正式论文 Methods 的 “Correctness” 说明，该分数由 GPT-4o Turbo 按规则评估，包含专家指定的答案要素和通用要求。Results 的 “Limitations of ScholarQABench” 指出启发式规则可能偏向更长的回答。 [L10 p.6–11](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=6) · [Nature 表 1、Methods 和局限](https://www.nature.com/articles/s41586-025-10072-4) · [arXiv v1 摘要与 §4.2](https://arxiv.org/html/2411.14199v1#S4.SS2)
- **语料版本。** 预印本 §4.1 和正式论文 Methods 的 “Scientific paper collection and data store construction” 都将 peS2o v2 用于主要评估。正式发布的语料库基于 peS2o v3，包含截至 2024 年 10 月的 4500 万篇论文。v2 的截止时间为 2023 年 1 月。Methods 的 “Initial paragraph retrieval” 还列出 Semantic Scholar 与网页搜索入口。单个索引的截止时间只限定该索引。在 Infra 证据清单中分别记录代码 commit、文档版本、运行时间和搜索入口。 [Nature Methods 全文](https://pmc.ncbi.nlm.nih.gov/articles/PMC12935541/) · [arXiv v1 §4.1](https://arxiv.org/html/2411.14199v1#S4.SS1)
- **引用存在与引用支持。** 正式论文 Methods 的 “Citation verification” 用引用附近的句子核对证据。评价方法 “Citation accuracy” 分开检查每条引用是否支持句子，以及句子的内容是否由引用覆盖。正式论文表 2 中，无检索 GPT-4o 的引用标题在作者的匹配流程中未找到的比例，计算机科学为 78.7%，生物医学为 94.8%。报告这一数字时保留标题匹配口径。在 Infra 复核中，记录准确的文件位置后，还需逐条核验该片段能否支持相应版本、条件与结论。 [Nature 表 2 与 Methods](https://www.nature.com/articles/s41586-025-10072-4) · [L10 p.35–49](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=35)
- **增加来源与上下文的代价。** 正式论文 “Ablations of retrieval” 与 Extended Data Table 2 固定 8B 生成器和重排方式，关闭反馈与引用核验。Scholar-CS 上，网页搜索、Semantic Scholar、两者组合的 rubric 分别为 45.9、47.9、49.6，引用 F1 分别为 12.6、39.1、47.6。该节还报告未微调的 Llama 3.1 8B 在超过 10 个片段后退化。个人案例先试小份可核查片段，并记录读入成本。论文结果依赖该语料、模型和评估设置。 [Nature “Ablations of retrieval” 与 Extended Data Table 2](https://www.nature.com/articles/s41586-025-10072-4) · [arXiv v1 §4.3 图 4](https://arxiv.org/html/2411.14199v1#S4.SS3)

## 反例｜找到了文件，仍缺少判断所需的行

L10 p.27–28 使用 BrowseComp-Plus 的 ACL 2026 标题，链接指向早期预印本。正式版 §5.3 与预印本 §4.8.1 都报告 GPT-4.1 的短答案准确率，BM25 条件为 14.58%，给出全部人工标注支持文档后为 93.49%。该对照改变了获取证据的路径，也给了模型更多已筛选内容。它度量该设置下的改进空间。 [L10 p.27–28](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=27) · [ACL §5.3，pp.22355–22356](https://aclanthology.org/2026.acl-long.1023.pdf#page=7) · [arXiv v1 §4.8.1](https://arxiv.org/html/2508.06600v1#S4.SS8.SSS1)

正式版 §5.5 表 3 固定 Qwen3-Embedding-8B。系统原先只提供文档前 512 tokens 的预览。增加全文读取工具并改动提示后，GPT-4.1 的准确率从 35.42% 到 43.61%，平均搜索调用从 8.67 到 10.03，另有 1.85 次全文读取。Qwen3-32B 从 10.36% 到 11.69%，平均只读取 0.27 次全文。工具、提示和调用量一起变化，表中差值包含这些因素。它说明在检索器固定时，后续读取方式和工具使用仍会影响结果。 [ACL §5.5 表 3、附录图 9](https://aclanthology.org/2026.acl-long.1023.pdf#page=8)

AgentIR 的 v3 §5.1 表 2 保留了 L10 p.115–123 所引 v1 的对应读数。Tongyi-DR 在关闭 visit 工具时，单用 Qwen3-Embed-4B 的准确率为 48.67%，加当轮检索意图为 55.54%，仅训练检索器为 59.40%，两者合用为 66.27%。§5.2 表 3 中，历史查询、推断和文档一起作为输入时为 60.00%，历史因上下文限制只保留最近约三轮。每种输入各自训练检索器。Infra 可先在现有检索中加入公开的待验证问题、版本和约束，再观察变化。 [L10 p.115–123](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf#page=115) · [AgentIR v1 §5](https://arxiv.org/html/2603.04384v1#S5) · [AgentIR v3 §5.1–5.2](https://arxiv.org/html/2603.04384v3#S5.SS1)

## 对证据包评估的影响

先保留普通检索与证据直给的两组设计。固定同一个助手、任务、材料快照与预算。证据包给出人工核验的原始片段与适用条件，答案单独存放。如果直给有收益，再加一组，只给正确文件和定位线索，让助手在同一预算内自行读范围。记录四项可观察结果。是否命中适用来源，关键行是否进入模型输入，结论是否由这些行支持，以及是否指出还缺哪项观测。定位线索组也改善时，先比较索引和范围读取。关键行已提供仍反复误判时，再对照硬检查、人工示范或训练数据。 [课程案例评估决策](../DECISIONS.md)

| 可留出的案例类型 | 需要测试的判断 |
| --- | --- |
| vLLM 另一个调度 PR 或 commit 家族 | 限制条件藏在测试文件后段。查到文件后，检查能否读到断言并区分教学回放与实际运行结果。 |
| B300 另一个 kernel 家族或独立 workload | 两份相似计时报告使用不同分配或计时条件。评审接受任何能证明适用条件的原始片段。 |
| DeepInfer 一次独立 profiling 事件 | 有组件结果，缺端到端依赖 trace。合格回答限定已测范围，并说明还要观测的依赖。 |

这些是待找材料的留出类型。把同一 PR、运行批次或根因的变体放进同一分组。先由人工确认每题能从材料中回答到哪一步，再固定评估集。覆盖到 L10 相关页、OpenScholar 正式正文与上述图表、BrowseComp-Plus 正式与预印本的指定小节、AgentIR v1/v3 的指定消融。正式 OpenScholar 全文经公开 JATS XML 核验。本轮未做复现或留出评测。 [正式论文 JATS 全文](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12935541/fullTextXML)
