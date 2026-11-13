# L16–L23 阅读覆盖

核验日期 2026-11-12。基于官网课表数据 `/assets/index-BoEP-SNx.js`、公开 PDF 和一手引用。PDF 页数按实际页序。读取了可提取的文本，未对每个图像单独作视觉复核。需要图中数值时优先查原始论文。

| 讲次 | 公开来源状态 | 实际阅读范围 |
|---|---|---|
| L16 Safety 2: Observability and Monitoring | 官网课表只有一篇官方阅读。当前条目无 slides、录像链接 | [OpenAI 官方阅读](https://openai.com/index/securing-our-frontier-model-training-environment/)，查阅概要和 Resuming frontier training with stronger defenses。只提取防御、监控和停止的设计经验。 |
| L17 Agents and the Future of Work | 有 [PDF](https://www.cmu-agents.com/slides/2026-10-27-zora-wang.pdf) | pp.1–62 文本扫描。重点 pp.16–25、27–32、34–45、46–53。官网条目未列录像。 |
| L18 Interaction 1: Multi-Agent Interaction | 有 [PDF](https://www.cmu-agents.com/slides/lecture-18-multi-agent-systems.pdf) | pp.1–44 文本扫描。重点 pp.4–8、11–25、27–32、39–44。官网条目未列录像。 |
| L19 Human-Agent Interaction | 课表列 11 月 10 日及 Valerie Chen。当前无 slides、录像、阅读链接 | 核对[讲师网站](https://valeriechen.github.io/)和课程标题检索。未查到可确认是本讲的公开原材料。 |
| L20 Reranking and Critic Models | 课表列 11 月 12 日。当前无 slides、录像、阅读链接 | 已核对官网与讲次标题检索。只登记课表条目。 |
| L21 Tree Search | 课表列 11 月 17 日及 JY Koh。当前无 slides、录像、阅读链接 | 仅登记未来课程条目。 |
| L22 Guest Lecture | 课表列 11 月 19 日及 Karthik Narasimhan。当前无具体题目或材料链接 | 仅登记未来课程条目。 |
| L23 Guest Lecture | 课表列 11 月 24 日及 Sasha Rush。当前无具体题目或材料链接 | 仅登记未来课程条目。 |

[官方课表](https://www.cmu-agents.com/#/schedule)

PDF 快照 SHA256。

- L17 `e07e9da87a97680981fca5b3ba766a7c620c83e8c1515fb75a798fe5812b79b1`
- L18 `5da3102180a75b0c8ad6c849ef497e6cc165ebcd77b4428ce8a177de6ebdc93c`

## 定向回查的原始引用

| 讲次 | 来源 | 阅读定位 |
|---|---|---|
| L17 | [How Well Does Agent Development Reflect Real-World Work? v1](https://arxiv.org/html/2603.01203v1) | §2.3 的分类映射与覆盖口径、人工验证。只用方法及限制。 |
| L17 | [GDPval v1](https://arxiv.org/html/2510.04374v1) | §1–2 的任务构建、专家评审和适用范围，§5 的数据限制。 |
| L17 | [Measuring What Matters v1](https://arxiv.org/html/2511.04703v1) | §3.4、§3.7–3.8、Discussion，构念效度检查表的相关项。 |
| L17 | [CollabSkill v1](https://arxiv.org/html/2606.09833v1) | 引言、排名相关结果、Limitations，核对同类工具的协作排名和样本边界。 |
| L17 | [Efficient Test-Time Adaptation v1](https://arxiv.org/html/2609.04141v1) | §2.1 人机动作与交替过程。上下文和权重适应结果只看概要。 |
| L18 | [Effective Strategies for Asynchronous Software Engineering Agents v1](https://arxiv.org/html/2603.21489v1) | §2.3–2.5 的 worktree、消息、依赖和验证条件。 |
| L18 | [Scaling Discovery through Test-Time Communication v1](https://arxiv.org/html/2609.21032v1) | §3.4 和 Table 3，Appendix B 的解释范围。 |
| L18 | [One Frozen Simulator Is Not Enough v1](https://arxiv.org/html/2608.12253v1) | §4 的 held-out 曲线和峰值口径，§5 与评估局限。 |
| L18 | [Multi-Agent Computer Use v1](https://arxiv.org/html/2606.01533v1) | 引言与可并行依赖图定义，未核对全部实验表。 |

本批针对以上引用做了内容级核查。L17 和 L18 的其他直接引用以[来源索引](../sources/)的实际状态为准。没有用作者的其他演讲或同主题第三方笔记填补 L19–23 的官方材料缺口。

2026-11-13 增量更新。L20 官网已发布讲义与引用。L19 仍未列材料，L21–L23 尚未开讲。L20 实际页码、图示和来源核查见 [新课件补查](search-2026-11-13.md)。
