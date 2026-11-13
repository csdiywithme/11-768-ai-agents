# L8–L12 与 A3 阅读记录

核验日期为 2026-11-12。页码按 PDF 实际页序计。L8、L9、L11 和 L12 检查了全篇抽取文本。L10 先检查全部页面的主题，再精读选段。图表与公式的视觉核验、录像画面与未列出的课程资料仍有缺口。

| 课程 | 实际范围 | 字幕和原始视频 |
|---|---|---|
| L8 · SFT | [官方 PDF](https://www.cmu-agents.com/slides/lecture-08-sft.pdf) p.1–49 文本。细读 p.11–19、p.21–34、p.38–47 的 mask、数据和目标环境评估。SHA-256 `1598a8bd9784ec044f89b8951af52bb2f80c0236b73aada0ca4cc3038049c4e3` | [第三方英文字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-08/) 中检查 10:35、12:45、13:41、16:08、19:06、27:26、41:23、49:35、51:32、52:52、54:23 附近段落。按官方 PDF 对照。 |
| L9 · RL Foundations | [官方 PDF](https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf) p.1–46 文本。检查 SFT 与 RL、ReST、策略梯度、基线和同组退化。SHA-256 `c8724ca6fbadb2afef38d627bbab2e09f8e8077fb1bb4dc0a4a078fb8669c1ca` | [第三方英文字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-09/) 中检查 07:21–12:39、1:04:28–1:04:51 与 1:13:46–1:17:09 的英文段落。按官方 PDF 对照。 |
| L10 · Deep Research | [官方 PDF](https://www.cmu-agents.com/slides/lecture-10-deep-research-agents.pdf) p.1–124 主题扫描。检查 p.3、6、12、14、18、21、26–27、29、31–32、35、38–39、42–45、47、50、52、57–58、60、63、65、67–68、70–71、73–74、76–77、80、82–85、89–90、93–95、98、100–101、103、105–107、110–111、114、117–119、122–124 的可提取文字及部分图注。SHA-256 `b3f9fafe7d586394efe3880d026c653b8b079151ffc75607d6aa7fab9c9ee259` | 课表快照未列。官方录像由 [播放列表补查](recording-recheck.md)核验。 |
| L11 · Advanced RL | [官方 PDF](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf) p.1–58 文本。检查信用分配、不同策略比率、奖励和蒸馏。SHA-256 `e2f23de9092424cde3a72805e3b343d261a2e219518871843e719c4dbe5cc968` | 课表快照未列。官方录像由 [播放列表补查](recording-recheck.md)核验。 |
| L12 · RL Systems | [官方 PDF](https://www.cmu-agents.com/slides/lecture-12-rl-systems.pdf) p.1–43 文本。检查训练与推理边界、上下文前缀、异步、部署拓扑和诊断。SHA-256 `15a062df1513c0710e4e3bd8cfa247d4b131b966faadc32e5101561c322a8a13` | 课表快照未列。官方录像由 [播放列表补查](recording-recheck.md)核验。 |
| A3 · scottyrl | 本地提交 `88d1b1fa2aea58938d6d251847ce5559c4ac6558`。检查 [讲义](../../../assignment-3/ASSIGNMENT.md) 的任务和交付、README、data/README、固定配置清单、SkyRL 拓扑、训练与拼接函数的契约及相关测试。重点行在专题笔记内。 | 未运行作业。未用课程成本估算替代实测。 |

L8、L9 官方视频网页可返回标题、播放器和自动字幕入口。两条公开 timedtext 请求都返回 HTTP 200 和空响应体，网页抓取工具未取得可读字幕。表内时间来自第三方整理的英文自动字幕，表示实际检查的片段。原录像画面按 [播放列表补查](recording-recheck.md)中的静帧记录。L8 录像问答约 12:23 提到 Meta 的代码执行世界模型，[CWM](https://arxiv.org/abs/2510.02387) 是与描述一致的原始论文候选。尚未用录像画面或官方字幕确认题名。

## 原始资料的实际检查

| 来源 | 课程入口 | 已检查范围 |
|---|---|---|
| [Data Recipes for Agentic Models v1](https://arxiv.org/html/2606.24855v1) | L8 p.25–28、43、47 | §3.2–3.6、§4。重点对照合成改写、同任务多次采样及任务来源混合。 |
| [Agent Data Protocol v1](https://arxiv.org/html/2510.24702v1) | L8 p.31–34 | §3.2–3.3 数据结构与转换，§6 表 3–4。 |
| [DAgger](https://proceedings.mlr.press/v15/ross11a/ross11a.pdf) | L8 p.42 | §1 与 §2 的状态分布定义。 |
| [Dr. GRPO v1](https://arxiv.org/html/2503.20783v1) | L9 p.39–40、A3 讲义 | §3.1–3.2 的响应归一、组 std 和分母。 |
| [BrowseComp-Plus v1](https://arxiv.org/html/2508.06600v1) | L10 p.27–28 | §3.2 固定语料流程，§4.6 与 §4.8 的取证方式及 oracle。 |
| [DeepResearch Bench v1](https://arxiv.org/html/2506.11763v1) | L10 p.47–49 | §3.2 与附录 E 的主张—来源对和计分。 |
| [ResearchQA v2](https://arxiv.org/html/2509.00496v2) | L10 p.43–44 | §3.2 的 rubric 专家复核与局限。 |
| [AgentIR v1](https://arxiv.org/html/2603.04384v1) | L10 p.106–123 | §1、§5.1–5.3 与表 2–3 的输入和训练消融。 |
| [DR Tulu v1](https://arxiv.org/html/2511.19399v1) | L10 p.84–98 | §3.2 的 rubric 管理及 §8 的训练与评估不一致。 |
| [AReaL v1](https://arxiv.org/html/2505.24298v1) | L11 p.19–20、L12 p.25、30 | §3.2、§4–5 的同步、异步和策略修正，§7.2 的运行对照。 |
| [Tinker Tutorial 404](https://tinker-docs.thinkingmachines.ai/tutorials/advanced/sequence-extension/) | L12 p.19、A3 讲义 | 读教程正文的拼接、拆分与 mask。页面标题中的 404 是教程编号。 |
| [Miles TITO](https://www.lmsys.org/blog/2026-05-13-no-token-left-behind/) | L12 p.21、24 | 原理、字符串往返失败模式、比较器、2026-08-18 更新。 |
| [SkyRL Tinker API](https://docs.skyrl.ai/docs/tinker/overview) | A3 讲义和部署 README | 概览的训练与采样边界。课内实际使用固定 fork，不能从当前文档推断该 fork 的完整兼容性。 |

## 待补范围

- L8、L9 的其余字幕段落，L8 讲义公式与 mask 颜色、L9 的动态演示。
- L10 未逐一复核的图像保留范围差异。OpenScholar 正式全文已从公开 JATS 核查，指定章节和表格见 [L10 补查](../sources/retrieval-followup.md)。
- L11、L12 的未精读引用，优先查 CTPO、PULSE、R3 和训练 prefix sharing。它们影响 token 与状态契约和可复现性判断。
- 官方 PDF 的链接由课程索引独立登记。录像问答中的口头引用要与可读字幕和画面复核后再扩展。

## 官方录像播放器补查

尝试在应用内浏览器打开 L8 官方 YouTube 录像 27:26。浏览器工具启动 Node runtime 时失败，未到达播放器。L8 p.28 的官方 PDF 图已独立复核，见 [图示复核](evaluation-operations.md#图示复核)。这个访问结果与 YouTube 字幕响应为空分别记录。
