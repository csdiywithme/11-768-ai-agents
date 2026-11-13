# 评估、编排与运行监控

## A2 · 先验证证据，再判断方法

A2 的 validator 看任务、输入、轨迹和输出，按错误类别给出证据。代码逐类计算 MCC，保留正负样本数。无支持类和常数预测都可能得到 0，只有前者带 insufficient-support。课程用自建压力题找失败，再用私有集检查泛化。[A2 讲义](../../../assignment-2/ASSIGNMENT.md#evaluation-format) · [计分器](../../../assignment-2/workflow/evaluate.py#L124)

可用确定性检查验证路径、版本、数值和配置关系。证据能否支持结论仍需人审或有证据定位的模型检查。A2 要求两个 Harbor mutant，其中一个可信错误解通过现有检查。迁移到本地，故意保留正确数字但去掉硬件、精度或测量范围。再构造一次正确性失败，确保它真的被检查器拦截。[A2 Harbor 要求](../../../assignment-2/ASSIGNMENT.md#package-your-tasks-as-harbor-environments)

查看 baseline 还发现一个评估输入的退化点。context overflow 会先压缩文字，若仍失败再缩图。缩图可以使调用继续，也会改变对细节的可见性。迁移到 Infra 时保留原始数据引用，并记录评估器实际看到了哪个切片。区分原始 log 已缺失与提交给模型的上下文截断。[A2 baseline](../../../assignment-2/validator/baseline.py#L124)

## L13–L14 · 状态、证据与工具边界

L13 pp.18–29 区分 agent 与工具所在的隔离范围，讨论共享状态、生命周期与权限。pp.30–42 区分动作和结果的可见性、规则检查与模型检查。读源码、检查已保存的 profiler 产物，与修改 repo 或提交 GPU 作业是不同的能力。操作风险和结果证据应体现在工具接口和日志里。[L13](https://www.cmu-agents.com/slides/lecture-13-agent-safety.pdf#page=18)

L14 pp.14–28 从工具动作、观察、持久化、远程执行和确认扩展到 skills 与分工。pp.34、42–43 讨论人工投入、从历史运行改进技能及留出任务的结果。可以先给单个助手按需加载一项实验检查技能。把重复步骤交给工具，把并行委派用于独立查证。评价时保留人工纠错和审阅时间，避免只看模型调用次数。[L14](https://www.cmu-agents.com/slides/lecture-14-openhands.pdf#page=14)

## L15 · 固定工作流与开放式调查

L15 pp.3–12 给出固定流程、条件路由和显式状态。pp.16–22 讨论 checkpoint、恢复、重试、暂停与人审，区分节点测试和端到端回归。固定流程让已知检查有明确的入口，也要求显式维护信息流。开放式调查适合证据会改变下一个工具选择的场景。可以把版本识别、配置校验和数据格式检查固定下来，把竞争假设的生成留作可比较的模型步骤。[L15](https://www.cmu-agents.com/slides/lecture-15-orchestration-and-monitoring.pdf#page=3)

L15 pp.24–36 讨论逐条轨迹阅读的成本，展示 Docent 的配对分析和 Hodoscope 的动作簇。聚合有助于定位重复失败。对策是否成立仍需回到原始动作和结构化指标。课程的固定节点测试例子也提醒我们，本地教学回放不能代替完整路径和版本对应的回归。[配对分析与范围](../sources/exploration.md#评估方法--从-l15-的引用回查原案例)

## Project · 让竞争解释变成可检验的研究问题

2026-11-12 官网项目页给出 Project Hours、Poster 和 Report。Project Hours 要求明确问题、相关工作、数据/基线/指标/预算和检验实验。报告要求写明环境、方法、硬件与超参数，分析失败并限定结论。负结果可以贡献清晰的适用边界。用这一结构为每个 Infra 案例写下可推翻的假设，再设计一项最能区分它们的测试。[Project scope、Report 与 FAQ](https://www.cmu-agents.com/#/project)
