# 长任务恢复与分工的原始证据

核验日期 2026-11-12。以 L3 pp.47–48、L5 的规划与预算、L14 pp.22–23 的单 Agent 基线为入口。从实验记录易丢失的条件，以及后续动作是否值得委派这两个问题反查。

## 压缩后的状态是否可恢复

- [TRACE v2 §§3.1–3.3、Figure 5](https://arxiv.org/html/2608.06503v2) 从同一 AppWorld 状态分别用压缩前、压缩后的上下文继续执行，记录被阻断和重复的动作。两边都有原工具反馈。该对照能区分最终完成率相同但中途返工更多的情况。v1 HTML 的访问失败不影响 v2 的正文核查。
- [Anthropic · Context engineering for long-horizon tasks](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents) 将压缩、持久笔记和子代理分开讨论。文章指出，过早删掉看似次要的信息会在后续产生代价。它是作者工程经验。文中的示例不能估计本地 Infra 的节省。
- [OpenAI · Increasing application legibility](https://openai.com/index/harness-engineering/#increasing-application-legibility) 展示隔离 worktree 和可查询的日志、指标、trace。[Enforcing architecture and taste](https://openai.com/index/harness-engineering/#enforcing-architecture-and-taste) 说明团队把不变量放进 linter 和测试。内测产品的开发速度是作者估计，缺同任务的人工对照。

**可检验的迁移。** 固定同一运行标识、版本和只读证据。在已出现 PDL 反转或传输探针失败后，从原上下文和精简状态分别继续。保留原文位置、适用条件、失败状态、当前决策与可回查的产物 ID。比较首个动作、重复读取和危险误判。只有在续做问题高频出现，且补索引仍无法恢复时，再比较专门的状态管理。

## 多 Agent 的对照先固定角色和预算

[Samway 等 v1 §5、附录 C.2–C.5、Table 3](https://arxiv.org/html/2610.17471v1) 将规划、实现、验证阶段分别委派。ProgramBench 的 Sol 实验中，规划委派可在较低开销下接近全阶段团队的平均 reward。附录 C.2 也给出失败条件。某些主代理很少委派，另一组主代理虽然委派，收益仍未达到单 Agent。每题结果有波动。该论文的预算、模型和评分口径限于其软件任务。

课程的 MACU 结果和上述实验都说明，先找真正可以独立核对的子问题。若一项问题要跟随同一运行的状态连续解释，多次交接可能丢掉约束。若只是同时核对源码断言、测量协议和数据分组，分工有可比较的输出。对照时固定总预算和通过条件，分别记录重叠工作、汇总返工与缺失证据。[L5/L7 阅读记录](../coverage/harness-reading.md) · [L17–L18 阅读记录](../coverage/people-coordination.md)
