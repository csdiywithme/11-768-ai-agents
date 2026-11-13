# L1–L7 与 A1 阅读记录

核验日期为 2026-11-12。课表、PDF 和本地作业是课程一手材料。English auto captions 是第三方整理的录像字幕，只用于定位讲述，并用官方 PDF 复核。以下时间是实际检查的字幕段起点，不能当作连续观看的范围。页面链接以 PDF 实际页序计。可搜索 PDF 文本存在图像页与数学符号抽取缺口。

| 课程 | 实际阅读 | 字幕证据和范围 |
|---|---|---|
| L1 | [PDF](https://www.cmu-agents.com/slides/lecture-01-agents.pdf) 39 页标题扫描，检查第 7–19、26–33 页文字 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-01/) 38:15、59:32。对应权重与 harness、课程目标。原录像网页本次抓取不可用。 |
| L2 | [PDF](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf) 60 页标题扫描，检查第 13–17、19–29、36–59 页文字 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-02/) 31:42、39:00。对应 JSON 结构约束、MCP。 |
| L3 | [PDF](https://www.cmu-agents.com/slides/lecture-03-long-context.pdf) 54 页标题扫描，检查第 1–53 页可抽取文字，部分图页只有标题 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-03/) 20:08、1:04:24、1:07:30。对应线性注意力、前缀缓存、压缩。官方录像页本次只返回外壳。 |
| L4 | [PDF](https://www.cmu-agents.com/slides/lecture-04-memory-and-skills.pdf) 46 页标题扫描，检查第 1–8、10–13、16–20、24–25、29–45 页文字 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-04/) 17:28、51:24。对应渐进加载和 Agent Workflow Memory。官方录像页本次只返回外壳。 |
| L5 | [PDF](https://www.cmu-agents.com/slides/lecture-05-planning.pdf) 48 页标题扫描，检查第 15–27、29–34、40–47 页文字 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-05/) 47:22、50:58。对应反馈重规划和交互预算。 |
| L6 | [PDF](https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf) 82 页标题扫描，检查第 23–30、32–45、47–54、62–72、74–81 页文字 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-06/) 30:34、1:00:59。对应代码评估和正反向回归测试。 |
| L7 | [PDF](https://www.cmu-agents.com/slides/lecture-07-computer-use-agents.pdf) 94 页标题扫描，检查第 2–6、17–19、24–28、34–44、47–50、56–80 页文字 | [第三方字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-07/) 34:29、57:45。对应长任务评估、GUI 训练数据。 |
| A1 | [本地讲义](../../../assignment-1/ASSIGNMENT.md) 的 Part 1–3、评分和交付，在提交 67498d87e36f7b45a2a4ce82d65d1139fb6a51c0 上定位关键实现及测试 | 本轮重点复核讲义 85–378 行、src/assignment/agent/base.py 的 TODO 和 tests/test_agent.py 的相应契约。课程环境未运行。 |

## 向外核查

- [OpenHands Tool System](https://docs.openhands.dev/sdk/arch/tool-system) · L2 工具注册与分派。查阅 Core Responsibilities 和 Key Components。属于当前文档，未核对 L2 讲义中的源码提交。
- [Agent Skills specification](https://agentskills.io/specification) · A1 指向的官方协议。查阅 Directory structure、Frontmatter 和 Progressive disclosure。属于当前规范，A1 是简化实现。
- [SkillsBench v4](https://arxiv.org/html/2602.12670v4) · L4 原始论文。查阅 §4、§5.1 的匹配对照与附录 D.6 的失败机制。结论用于设计实验，未复现原 benchmark。
- [Agentless arXiv v2](https://arxiv.org/html/2407.01489v2) 与[作者仓库](https://github.com/OpenAutoCoder/Agentless) · L5/L6 固定工作流对照，查阅 §3 与 Table 4 的步骤和测试消融。
- [MACU arXiv v1](https://arxiv.org/html/2606.01533v1) · L5/L7 多 Agent 原文。查阅 §4.2 的并行度、规划预算消融和附录 B 的时间与可分解条件。
- [Building effective agents](https://www.anthropic.com/research/building-effective-agents) · L5 第 16 页引用。查阅工作流与 Agent 的定义，以及复杂度与预算建议。属于 2024 年作者经验，文章标注了工具生态的时间变化。

- [PagedAttention v1](https://arxiv.org/html/2309.06180v1) · L3 block table。复核 §4.2–4.3 的块分配与一次 decode 步骤。
- [SGLang v2](https://arxiv.org/html/2312.07104v2#S3) · L3 prefix reuse。查阅 §3、缓存复用实验与多副本索引摘要。现代版本实现待核。
- [DistServe v2](https://arxiv.org/html/2401.09670v2) · L3 服务指标。复核 §2–4 的 goodput、phase 干扰和带宽相关放置。
- [CUA-Gym v2](https://arxiv.org/html/2605.25624v2#S2.SS1) · L7 §2.1 的 setup 与 verifier 分离和最终 rollout 过滤。
- [Gym-Anything v1](https://arxiv.org/html/2604.06126v1#S7.SS1) · L7 §6.3–7.1 的预算、评估器和 Table 3。实测范围限于论文报告。

## 官方播放器和图页复核

本轮从课表指向的 YouTube watch 页尝试跳到三个时间点，也检查了同一视频的公共 embed 入口。带时间参数的 watch 和 embed 在本次网页读取工具中均返回 Cache miss。无时间参数的 L3 watch 只返回页面外壳，L6 和 L7 的 watch 仍返回 Cache miss。未取得可播放画面，此处只记录首轮播放器工具尝试。随后读取了 [官方录像静帧](recording-recheck.md)。以下官方 PDF 图页另用 Poppler 渲染并目视确认。

| 录像尝试点 | 对应字幕证据 | 官方讲义图页实际核对 |
|---|---|---|
| [L3 1:07:30](https://www.youtube.com/watch?v=AiwCCvFW1uE&t=4050s) | 1:07:30.880 起的片段，讨论压缩后未来动作接近原轨迹 | [PDF 第 47–48 页](https://www.cmu-agents.com/slides/lecture-03-long-context.pdf#page=47)。图中把行为保真、预算约束、可恢复性分成三项。下一页分别标出原样保留的 anchors 和 recent tail、编码的 checkpoint、外部证据，以及可丢弃的重复项。 |
| [L6 1:00:59](https://www.youtube.com/watch?v=1BWeH1oOM7k&t=3659s) | 1:00:59.760 起的片段，讨论复现与既有行为测试 | [PDF 第 47 页](https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf#page=47)。SWE-bench 图中，代码库与题目进入模型，测试由独立 harness 使用。harness 应用补丁和测试，再运行和记录结果。 |
| [L7 34:29](https://www.youtube.com/watch?v=jwGluLrrqjQ&t=2069s) | 34:29.599 起的片段，讨论长任务中的状态、过程和稀疏奖励 | [PDF 第 39 页，印刷页码 55](https://www.cmu-agents.com/slides/lecture-07-computer-use-agents.pdf#page=39)。图中目标商品已加入购物车，实际下单却违反约束，因此整体失败。 |

## 待补范围

各讲剩余内容与图像演示尚未逐页确认。原视频未完整播放。字幕阅读按上述区间计入，第三方整理可能合并或误识别术语。PDF 引用由总索引独立登记。后续优先补 L3 的模型表原始版本、L5 的推理与交互对照原文、L7 的未读评估图示。
