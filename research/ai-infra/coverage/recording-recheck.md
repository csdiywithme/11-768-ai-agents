# 官方录像与课件图示补查

核验日期 2026-11-12。官网课表列出 L1–L9 的录像。由讲师 Graham Neubig 发布的[课程 YouTube 播放列表](https://www.youtube.com/playlist?list=PLSN0qpDfUvTM)有 L1–L14 共 14 条。逐项比对官方播放列表。再用 L10–L14 的公开播放页核对讲次、标题、发布者及长度。官网资源 SHA-256 与首次快照一致。课表与录像列表分别记账。

## 实际查看的录像画面

按课程问题选择时间点。使用播放页元数据指向的官方 10 秒间隔 storyboard。以下是实看静帧，可验证演示当时的画面。完整音轨和连续画面仍需区分。

| 原录像 | 实看帧 | 对照结果 |
|---|---|---|
| [L3 长上下文](https://www.youtube.com/watch?v=AiwCCvFW1uE&t=4050s) | 1:07:30、1:08:20 | 先展示压缩后的状态估计，随后出现 anchors、checkpoint 和 recent tail。与 PDF pp.47–48 和 1:07:30 的第三方字幕主题对应。 |
| [L6 编码 Agent](https://www.youtube.com/watch?v=1BWeH1oOM7k&t=3650s) | 1:00:50、1:01:00、1:01:10 | 录像由单一 harness 图逐步加入 fail-to-pass 与 pass-to-pass。前者检验修复目标，后者检验已有行为是否保留。与 PDF p.47 的流程对应。 |
| [L7 Computer Use](https://www.youtube.com/watch?v=jwGluLrrqjQ&t=2060s) | 33:50、34:20 | 对比最终结果和过程的购物任务示例。静帧确认处于 Hybrid / trajectory 段。具体违反下单约束的字样以已核对的高清 PDF p.39 为准。 |
| [L8 SFT](https://www.youtube.com/watch?v=O3HSU0AoILc&t=1640s) | 27:20、27:30 | 从成功 rollout 的不良行为切换到按轨迹形状过滤的表格。验证“结果成功”和“示范过程适合训练”是两个判断。该片段与 p.28 的数据规模图是不同位置。 |
| [L10 Deep Research](https://www.youtube.com/watch?v=nKUBrXFQBUM&t=1500s) | 24:40、24:50、25:00、25:10、25:20 | 动画逐步展示搜索、工具返回和生成进入同一循环。回查课件与检索原文时，分别记录找到的文档、读到的证据及最终主张。 |
| [L11 Advanced RL](https://www.youtube.com/watch?v=jKx7_fOZRzg&t=1440s) | 24:00、24:20 | 从同步与异步 RL 转到同步等待图。短轨迹先结束仍要等批内其它轨迹。图是机制示意。吞吐和 staleness 以原论文对照为准。 |
| [L12 RL Systems](https://www.youtube.com/watch?v=dwR6NP9nI-8&t=1850s) | 30:50 | Weight Sync 画面并列全量广播与 delta sync，并注明推理端分片布局。与 PULSE 原文补查结合，避免把传输字节缩减直接写成墙钟加速。 |
| [L14 OpenHands](https://www.youtube.com/watch?v=VAzZkT68h2Q&t=1800s) | 30:00、30:40、31:10 | 从上下文管理进入专门角色，再展示单 Agent 基线。与 PDF pp.18、22–23 对应。比较分工前先记录单 Agent 的上下文保留和交接成本。 |

本批请求上述八个录像公开播放页内提供的英文字幕地址。地址返回成功状态，但正文均为空。已读逐字稿仍以 [L1–L7](harness-reading.md) 和 [L8–L9](training-reading.md) 中记录的第三方英文选段为准。L13 已核验发布入口，未用录像画面补充其内容。L15–L23 在当前公开播放列表中没有条目。
