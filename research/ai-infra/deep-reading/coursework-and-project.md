# A1–A3 与 Project · 把课程方法转成可比较的工程决策

### `https://github.com/cmu-agents/assignment-1`

- 原文：[A1 ASSIGNMENT · 67498d8](https://github.com/cmu-agents/assignment-1/blob/67498d87e36f7b45a2a4ce82d65d1139fb6a51c0/ASSIGNMENT.md)
- 范围：Rules、Parts 1–3、Grading、Submission，回读本地固定提交的工具契约和公测边界。
- 方法与限制：通过修复 chess app 和 chess agent 分开评估工具集成、上下文压缩和观察表示。技能按需加载。对局中先用模拟接口探索，再用实际接口提交一个动作。作业条件下的实现和测试定义已有定位，本批未运行。
- 工程用法：先保存同一实验的只读状态快照，用快照比较下一项测量，再由执行接口落实已选择的动作。用相同问题对照完整日志、紧凑字段和只给摘要，检查时间、输入长度和错误状态。用不同版本的实验复现重测能否保留关键限制。

### `https://github.com/cmu-agents/assignment-2`

- 原文：[A2 ASSIGNMENT · f609e76](https://github.com/cmu-agents/assignment-2/blob/f609e76d2efd302b4b8babee06c5592899010b8b/ASSIGNMENT.md)
- 范围：Parts 1–4、各标签、人工评分、Harbor 验证和报告。本地核对 harbor/example 和 validator 约定。
- 方法与限制：先读真实 seed 轨迹里的误报漏报，修改 verifier，再自建分布外任务压力测试。每类报告 MCC，按规则汇总。参考实现与两个故意错误的 mutant 可以揭示评估覆盖。例子中太小而难以阅读的图仍可能过规则。课程私人评估和模型运行未执行。
- 工程用法：对 kernel benchmark 写一个数值正确且工作量相同的对照，再故意给一个输出错误但很快的结果，以及一个数值通过却缺代表 workload 的结果。观测误拦、误过与适用范围，发现遗漏后再修 gate。评估调度方案可给首次分歧和终态分别建反例。

### `https://github.com/cmu-agents/assignment-3`

- 原文：[A3 ASSIGNMENT · 88d1b1f](https://github.com/cmu-agents/assignment-3/blob/88d1b1fa2aea58938d6d251847ce5559c4ac6558/ASSIGNMENT.md)
- 范围：数据流水线、同步和异步训练、消融、TextCraft-Synth、短探针、提交与诊断指标。
- 方法与限制：记录采样版本、mask、action token logprob，区分策略梯度、baseline 和更新。异步训练有有界队列、丢弃和 snapshot。报告需按指标的定义读质量、等待和 staleness。G=1 对 GRPO 同时改变算法和组大小，不能解释成纯组大小消融。
- 工程用法：在任何昂贵矩阵前用短探针验证配置、日志和正确性，用固定一组输入区分参数变化、运行拓扑与任务难度。若后端换了但新结果失败，先用最小轨迹比对版本和 mask，再测吞吐。

### `https://github.com/harbor-framework/harbor`

- 原文：[Harbor 作者仓库](https://github.com/harbor-framework/harbor) · [Tasks Overview](https://docs.harborframework.com/tasks/overview)
- 范围：README 的任务和运行入口，官方 Tasks Overview 的目录、指令、环境、参考方案、验证器与产物。
- 方法与限制：以独立任务目录固定初态、指令和验收。oracle 用于检查参考方案能通过。参考和验证器隔离后，还须看是否覆盖真正需求。文档给接口，未运行 A2 的官方 harness。
- 工程用法：用只读的 GPU 运行日志与版本化输入建一个离线决策题。让 verifier 检查原 run ID、允许的下一动作和引证。如果多个有效下一步可区分同一假设，分别纳入评分。用合理但错误的重试或性能结论检验 verifier。

### `https://www.cmu-agents.com/`

- 原文：[CMU 11-768 · Research Project](https://www.cmu-agents.com/#/project)
- 范围：官网 2026-11-13 公共资源快照的 Project scope、Project Hours、Poster、Report 和 FAQ，包括数据、基线、负结果和报告限制。
- 方法与限制：项目由 2–4 人完成，Project Hours、Poster、Report 分别为课程成绩 5%、15%、30%。早期要能说明问题、先前结果、要用的任务数据与基线，以及一个能检验想法的实验。海报展示与报告是两种交流任务。主文上限 9 页。FAQ 认可证据清楚的负结果。
- 工程用法：在决定工具、harness 或训练之前，先写明一项能改变工作判断的实验。用实际可比基线、边界和人工时间说明选择。如果看不到增益，把有效条件和负结果交付给下一轮，不为符合方案预期继续跑同类试验。

### `https://github.com/acl-org/acl-style-files`

- 原文：[ACL 官方 LaTeX 模板](https://github.com/acl-org/acl-style-files)
- 范围：仓库 README 的作者说明、模板文件和排版说明入口。
- 方法与限制：模板定义交付的排版和引用方式。课程自己的页数、报告内容和贡献说明仍看课程项目页。
- 工程用法：将最重要的配对条件、正确性和负结果放进正文，原始记录和详细 shape 放到可查附件。交付前检查任何速度主张能否从正文或一个直接链接找到分母。

### `https://cmu-l3.github.io/anlp-spring2026/assignments/assignment3&4`

- 原文：[CMU Advanced NLP Spring 2026 · Project](https://cmu-l3.github.io/anlp-spring2026/assignments/assignment3&4)
- 范围：报告 Data、Method、Results、Discussion、Limitations 及评分与提交。11-768 将该页标为项目指导的改编来源。
- 方法与限制：数据范围、基线选择、实现条件、对照指标、定性错误和统计结果都有独立作用。两门课的项目规则各自生效。
- 工程用法：目标若是选下一项实验，报告至少给出当时可用证据、选中动作、何时翻转、任务族和剩余缺口。对照没有增量时仍写清哪些输入和条件排除了方案。

### `https://smith.langchain.com/public/5e17b93a-17af-4386-ada0-9bd033f98a74/r`

- 原文：[L15 p.18 · 教学 trace 课件截图](https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf#page=18)
- 范围：登记的公开 LangSmith trace 网页未返回可读原始 span。对照 PDF p.18 的节点、模型调用和搜索截图。
- 状态：原件受限
- 启发：父 span 包含子 span，不能将两者相加。把 trace 的阶段归属与无 profiler 的端到端对照分开。

### `https://smith.langchain.com/public/aeddf201-687b-4389-acdb-137dd9683000/r`

- 原文：[L15 pp.19–22 · 审批和 trace](https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf#page=19)
- 范围：登记 trace 本次未取得原始 span。课件与标签确认对应 billing 审批前的教学示例。
- 状态：原件受限
- 启发：区分等待人审、已发送远端请求和执行成功。用已保存的状态与时间对照检查是否跨过审批，而不是从总时长猜测。

### `https://smith.langchain.com/public/416d3c74-4b43-4883-b189-92a57431a819/r`

- 原文：[L15 pp.19–22 · 审批和 trace](https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf#page=19)
- 范围：登记 trace 本次未取得原始 span。课件与标签确认对应 billing 审批后的教学示例。
- 状态：原件受限
- 启发：给恢复调用独立标识与关联事件，核对批准后是否完成所需的远端动作，避免把 workflow 恢复视为远端完成。

### `https://docent.transluce.org/dashboard/9bafeb94-3f6e-4d03-bd01-1ae8476e3f6e/analysis-plan/81230a32-88e0-42af-82c9-101660ab1d89`

- 原文：[Docent 作者报告](https://transluce.org/docent/blog/terminal-bench)
- 范围：登记 dashboard 本次未取得数据。作者报告的 Steps 1–2 提供成功失败配对与失误聚类的公开叙述。
- 状态：原件受限
- 启发：轨迹聚类提供值得核查的失败模式，实际根因要用每个匹配任务的运行状态、预算和原始日志确认。
