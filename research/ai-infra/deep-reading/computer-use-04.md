# L07 厂商的计算机操作接口与评测

以下记录核对课件 PDF pp.57、79、85 所列的厂商资料。API 文档反映 2026-11-13 可见版本，发布文章保留其历史口径。工程验证尚未执行。

### `https://ai.google.dev/gemini-api/docs/computer-use`

- 原文：[Gemini Computer Use 文档，课程链接](https://ai.google.dev/gemini-api/docs/computer-use)
- 范围：原链接现重定向 Google AI Studio。读动作执行、状态回传、循环及 browser／mobile／desktop 动作表。课件用 Gemini 3.8 Flash 举例，当前页示例用 `gemini-4-hydrogen`，接口按当前页核对。[现行文档](https://aistudio.google.com/docs/computer-use)
- 机制与证据：模型返回带 `intent` 的函数调用。点击字段范围为整数 0–999，客户端按画面尺寸将其映射到实际像素。示例用 `call_id` 关联结果，带当前 URL 和 PNG，再用 `previous_interaction_id` 继续。JavaScript 例在一组操作后取图，组内中间状态可能只留在客户端日志。
- 工程动作与反证：对同一份 profiler 导出页回放两个视口，记录归一化坐标、图像尺寸、实际落点与已打开的 trace ID。若点击命中随缩放而变，先做坐标和视口回归。若组内页面变化使后续动作点错，在该转换点重新观察。

### `https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/`

- 原文：[Meta Muse Spark 1.1 发布文](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/)
- 范围：2026-07-09 发布，读 Agents、Computer Use 和 Coding。对照配套评估报告 §5.2.1 的 OSWorld-Verified 和 OSWorld 2.0 实验条件。[Meta 评估报告](https://ai.meta.com/static-resource/muse-spark-1-1-evaluation-report)
- 机制与证据：发布文描述模型可切换脚本、单个 GUI 动作或动作批次。评估报告的 OSWorld-Verified 是 GUI-only，361 个任务，1920×1080 截图、每题最多 200 步。OSWorld 2.0 同为 GUI-only，最多 500 步，区分完全通过率和部分得分。报告按基准说明部分对比读数来自别家自报或排行榜。
- 工程动作与反证：固定模型和任务，将“只用 GUI”与“允许结构化指标导出及少量 GUI 检查”分组，分别记步数、实际耗时和最终状态。若批量脚本只在导出畅通时提速，保留导出失败的回退样本和阈值。

### `https://developers.openai.com/api/docs/guides/tools-computer-use`

- 原文：[OpenAI Docs Computer use](https://developers.openai.com/api/docs/guides/tools-computer-use)
- 范围：读 SDK driver 职责、Use the API directly、Use the computer tool 与 Run safely。核对结构化操作和代码执行两条官方路径，未调用 API。
- 机制与证据：SDK 可运行模型工具循环，应用负责环境驱动、会话和清理。直接 API 中，应用将动作执行后的截图与原 `call_id` 绑定。模型会话的延续与真实执行环境各有状态，继续响应不会恢复 UI、登录或运行变量。官方示例要求在应用中核查最终结果。
- 工程动作与反证：以只读 benchmark 看板测断线重连。回放同一任务，在继续会话前检查环境 ID、当前页面和工作负载过滤器。若旧会话续上但 UI 已重置，恢复观测并废弃从旧画面产生的操作。

### `https://openai.com/index/introducing-operator/`

- 原文：[OpenAI Operator 历史发布文](https://openai.com/index/introducing-operator/)
- 范围：2025-01-23 发布，网页注明 2025-07-17 整合更新。读 How Operator works、How to use、Safety and privacy、Limitations。
- 机制与证据：研究预览以截图观察远程浏览器，使用键鼠动作。产品设计允许用户接管。预览文描述敏感输入时接管、重要动作确认以及部分站点监督。它也列出复杂界面表现的局限。此处证据限当时公布的能力与产品设计。
- 工程动作与反证：在只读排查演练中把人工接管设为显式状态。用户切换 workload 或修正环境后，重新确认活跃 run 和已完成检查。若恢复后重复执行旧任务，检查交接中的任务 ID 与最新观测。

### `https://openai.com/index/codex-for-almost-everything/`

- 原文：[OpenAI Codex for (almost) everything，2026-04-16 发布文](https://openai.com/index/codex-for-almost-everything/)
- 范围：读 Extending Codex beyond coding、开发流程与 Availability，并核对现行 Computer Use 文档对 macOS、Windows 的区别。[OpenAI Docs](https://learn.chatgpt.com/docs/computer-use)
- 机制与证据：发布文以 macOS 后台键鼠控制为例，同时介绍应用内浏览器、终端、预览和产物跟踪。现行文档称支持区域内 macOS 可后台执行，Windows 使用活动桌面前台。应用访问许可与 shell 或文件操作的执行范围分别管理。
- 工程动作与反证：用同一 UI 性能回归任务分别记录任务时长、工程师可并行工作的时间和干预次数。若前台操作占用工程师设备，评估专用会话的成本。若两条并行诊断共用窗口导致串扰，按进程与运行 ID 隔离产物后复测。

### `https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool`

- 原文：[Claude Platform Computer use tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool)
- 范围：读 `computer_toolset_20260801`、动作定义、截图尺寸、历史裁剪、错误处理和旧版迁移。未发请求。
- 机制与证据：通过 `toolset_name` 与 member 名称识别动作。坐标在交给模型的整屏截图像素空间，放大图不改坐标基准。截图须先符合模型尺寸约束，客户端保留缩放比例以逆映射。文档说明截图逐轮删除会改写缓存前缀，部分模型还会使后续思考块失效。
- 工程动作与反证：在高 DPI 的 Nsight 图上测整屏、按需放大与历史裁剪。记录点击误差、视觉 token、前缀命中和下一轮定位正确率。若删图后输入变短却降低定位正确率，保留关键事件图和可回查引用。

### `https://www.anthropic.com/news/3-5-models-and-computer-use`

- 原文：[Anthropic Claude 3.5 与计算机操作发布文](https://www.anthropic.com/news/3-5-models-and-computer-use)
- 范围：读发布文 Teaching Claude to navigate computers 部分，并核对同页对 3.5 Sonnet 与 3.5 Haiku 的描述。此处使用历史评测。
- 机制与证据：作者报告 Claude 3.5 Sonnet 在当时 OSWorld 截图输入条件下为 14.9%，增加可执行步数后为 22.0%。产品文没有给足把这些比例换算为相同时间和费用预算的细节。报告还说明当时滚动、拖动和缩放存在困难。
- 工程动作与反证：给同一批 UI 定位任务设不同步数预算，记录首次找到正确事件时的步骤与实际时长。若成功率的增加主要来自重复尝试且耗时过高，把脚本化检查、固定步骤数和人工检查作为可比较的基线。

### `https://www.anthropic.com/news/developing-computer-use`

- 原文：[Anthropic Developing a computer use model](https://www.anthropic.com/news/developing-computer-use)
- 范围：2024-10-22 研发博文，读 The research process、Making computer use safe、What’s next 的历史限制。
- 机制与证据：作者先在计算器和文本编辑器等少量应用训练，强调屏幕定位的像素判断与遇障自纠。离散截图像逐帧翻页，可能漏过短时通知。文中是早期研发观察，跨应用能否迁移要测目标软件。
- 工程动作与反证：准备同一导出操作的快、慢和异步完成状态。比较固定间隔截屏、按事件取样以及读取实际导出文件。若漏判集中在瞬态通知，使用可查询的结果状态确认，重新测试稳定性。

### `https://x.ai/news/introducing-grok-bot`

- 原文：[Introducing Grok Bot，2026-08-11 发布文](https://x.ai/news/introducing-grok-bot)
- 范围：读 A computer of its own、Work with many Bots、Show a Bot、Trust your Bots 和示例。未试用产品。
- 机制与证据：发布文描述云端计算机承载跨应用任务，多个 bot 用线程交接，观察人工示范后记录可复用流程。给出的生产故事为厂商自述，未提供可核对的同条件成功率、恢复成本或隔离实现。
- 工程动作与反证：把长时 benchmark 的观测、结果复核和后续分析拆给不同执行者，并只传共享运行 ID 与任务状态。测一次 worker 中止后的重复执行、交接遗漏和人工耗时。若新增角色后重复启动次数增加，先明确唯一任务所有者与完成凭证。

### `https://yutori.com/blog/introducing-n2`

- 原文：[Yutori Navigator n2 发布文](https://yutori.com/blog/introducing-n2)
- 范围：2026-08-26 发布，读 Computer-native computer use、Training n2、图 1 和比较表脚注。数字按发布时实验口径。
- 机制与证据：27B 模型可在 GUI、CLI 和工具间切换。作者用 rollout 确认任务可执行，再通过程序或 rubric 检查终态，并回收误判样本改进验证器。图 1 的 OSWorld 2.0 为 65.2% 部分得分，每题 API 成本 1.46 美元。比较表一部分结果引用其他报告，图中成本不包含本地运行资源和人工审查。
- 工程动作与反证：为开源框架调优构造“拿到正确 workload 并写出能复跑的运行报告”任务，用实际文件、配置和运行状态验证。分别记录部分得分、严格完成、GPU 分钟和审查时间。若新流程部分得分涨而完整实验完成率不动，检查验证器与最终工作目标是否一致。
