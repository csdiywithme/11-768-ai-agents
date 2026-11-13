# 上下文、证据索引、技能与训练的边界

本专题对应 L3、L4 和 A1 的上下文压缩及技能实现。页码按 PDF 文件的顺序，从 1 开始。 [阅读记录](../coverage/harness-reading.md) · [工具、计划与验证](harness-tools-planning.md)

## 让模型继续工作的状态可以比完整轨迹短

L3 用线性增长的交互历史说明累计输入 token 可以呈二次增长。复用稳定前缀可以减少重复 prefill。API 缓存价格、缓存命中和服务时间是不同指标。压缩会改变模型能看到的历史，缓存命中只改变重复处理的成本。一次压缩也可能失去此前的前缀复用。评估要同时看完成率、实际输入或缓存 token、延迟和费用。 [L3 第 2–5、37–44 页](https://www.cmu-agents.com/slides/lecture-03-long-context.pdf#page=2)

L3 还区分三类容易混淆的状态。模型结构决定每层访问哪些历史，局部注意力只直接读取窗口内 token，递归状态把历史折叠进固定形状状态。服务端 KV 管理决定缓存如何分配与共享，vLLM 的 block table 映射逻辑块和物理块，SGLang 的 RadixAttention 用前缀匹配复用已有 KV。Agent 的工作状态决定下一轮给模型看哪些证据。对 DeepInfer 先区分页表元数据工作、KV cache 存储路径和应用层证据检索，再按服务负载定义 TTFT、TPOT、SLO 与每 GPU goodput。DistServe 的解耦可以减少 prefill/decode 相互干扰，同时增加 KV 转移，放置策略取决于网络拓扑和带宽。与实际 workload 有关的收益要在匹配拓扑和流量的服务对照中测。 [L3 第 7–9、13–25、36–44 页](https://www.cmu-agents.com/slides/lecture-03-long-context.pdf#page=7) · [PagedAttention v1 §4.2–4.3](https://arxiv.org/html/2309.06180v1) · [SGLang v2 §3](https://arxiv.org/html/2312.07104v2#S3) · [DistServe v2 §2–4](https://arxiv.org/html/2401.09670v2)

第 46–51 页把压缩视为有预算的状态估计。原样保留目标、约束和近期完整动作。摘要记录决策、当前状态、已排除方向和下一步。将原始日志留在可定位的证据位置。反复压缩可能逐次丢失条件，例如把精确 CUDA 版本改成模糊的大版本。测量方法是放入会影响后续判断的约束，经过一次或多次压缩后继续完成任务，观察它是否正确使用约束。 [L3 第 46–51 页](https://www.cmu-agents.com/slides/lecture-03-long-context.pdf#page=46)

A1 保留原始任务与至少最近一组完整 action 和 observation。压缩过程保留用于评分的原始 API 记录。讲义要求用 6000 token 阈值与关闭压缩两种条件在同一个 vendored Django 任务上比较，检查活跃上下文实际缩短，补丁通过测试，再分析 token 成本。6000 是作业对照条件，Infra 任务的阈值应随日志长度和证据查找成本检验。 [A1 Part 2](../../../assignment-1/ASSIGNMENT.md#part-2-implement-context-compaction) · [实现契约与评分测试](../../../assignment-1/tests/test_agent.py)

**Infra 迁移假设。** 检查一次历史实验时，原样保存 commit、硬件、dtype、shape、计时方法、数据路径和未验证范围。将一条已排除的解释与对应证据写入工作状态。用相同输入对照完整轨迹、简短状态、简短状态加按需回查。记录是否重做已失败的测试、是否丢关键条件、是否得到相同或更好的下一步。

## 重复流程可以留在外部，也可以进入模型权重

L4 区分完整轨迹、可更新事实和可复用技能。上下文直接保留证据，但每次占用输入。外部产物易检查和修订，需要检索、校验、淘汰。SFT 可以改变稳定的行为模式，更新较慢且模型特定。选择记录什么时，应先识别未来需要精确事实、通用操作顺序，还是重复出现的判断行为。 [L4 第 2–6、16–20、37–45 页](https://www.cmu-agents.com/slides/lecture-04-memory-and-skills.pdf#page=2)

技能的渐进加载把名称和描述、正文、引用或脚本放在不同阶段。A1 只做目录和按名获取正文的简化版本。Agent Skills 的官方格式还规定目录名、frontmatter 和附带资源。格式描述技能的发现与加载。脚本权限和特定 CUDA、GPU 或仓库版本的兼容性仍需宿主验证。代码技能容易重复执行和单测，接口变化时较脆弱。文本技能留出判断空间，但可能无法减少步骤。 [L4 第 9–15、29–36 页](https://www.cmu-agents.com/slides/lecture-04-memory-and-skills.pdf#page=9) · [A1 技能要求](../../../assignment-1/ASSIGNMENT.md#4-load-skills) · [Agent Skills 格式规范](https://agentskills.io/specification)

作者实验给出一个适用范围。SkillsBench v4 的 87 个任务、18 个模型与 harness 组合使用匹配的无技能和任务配对技能条件。精选技能使平均通过率从 33.9% 到 50.5%，组合间增益不同。作者把同一任务的 verifier 保持固定。部分任务会退化。三组自生成技能诊断中，自生成结果低于无技能，其中有未发现技能、编写技能占用执行预算、技能固化错误假设等原因。结果支持在 Infra 案例做成对测试，迁移收益待测。 [SkillsBench v4，§4、§5.1、附录 D.6](https://arxiv.org/html/2602.12670v4)

**Infra 迁移假设。** 已知可靠的同轮对照或断言可做成短技能或脚本，并保留环境适用条件。某一形状下某个 tile 提速等临时结论保留为证据条目。若工具输出和流程已经稳定，模型仍系统性误读关键路径或对照口径，再考虑收集人工核验示范。留出不同任务、软件版本或硬件拓扑检验迁移。轨迹完成后，另外检查哪些步骤值得学习。 [L4 第 24–25、38–45 页](https://www.cmu-agents.com/slides/lecture-04-memory-and-skills.pdf#page=24)
