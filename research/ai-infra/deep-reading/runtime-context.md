# L03 上下文实现和 L07 模型输入及数据

以下是公开实现和发布材料的阅读记录。工程动作是从材料提出的验证假设，尚未运行。

### `https://github.com/openai/codex`

- 原文：[OpenAI Codex 仓库](https://github.com/openai/codex)
- 范围：用官方文档核对 Responses 压缩项的行为。另读提交 `454bf339f0da9eff8904936b0036ab85e8719d29` 的 `compact.rs`，重点是组装替换历史、恢复当前环境上下文、持久化边界。[OpenAI Docs](https://developers.openai.com/api/docs/guides/compaction) · [固定源码](https://github.com/openai/codex/blob/454bf339f0da9eff8904936b0036ab85e8719d29/codex-rs/core/src/compact.rs)
- 机制与证据：Responses API 返回的压缩项可传到下一窗口，其内部对调用方不透明。源码所读路径从当前环境重建上下文，再按真实用户消息、需保留的输出或压缩项定位插入点，并记录压缩窗口和响应 ID。API 链接和本地历史替换有各自的使用边界。
- 工程动作与反证：对跨窗口的调优任务，保留不可变运行 ID、GPU 配置和用户最后一次修改的约束。在原窗口、跨窗口、进程恢复后分别检查下一步。若恢复后仍能读到摘要却复用旧基线，用独立版本化字段恢复当前配置，并重新检查同一条任务。

### `https://github.com/anomalyco/opencode`

- 原文：[OpenCode 仓库](https://github.com/anomalyco/opencode)
- 范围：读 `dev` 提交 `b645291e9cb091e7d51a9caa489826b1bea34741` 的 core session 压缩逻辑，核对 `usePrecomputed`、`heldIndex`、`due` 与尾部保留。[固定源码](https://github.com/anomalyco/opencode/blob/b645291e9cb091e7d51a9caa489826b1bea34741/packages/core/src/session/compaction.ts)
- 机制与证据：预计算摘要带有会话头 ID、尾部边界、模型变体和初始指令。替换前重新核对这些字段、估算近期尾部长度。overflow 或 native 压缩走不同路径。刚完成压缩时先允许运行器按检查点重建请求，避免即时再次进入自动压缩。注释说明常规摘要请求维持原有前缀以复用 prompt cache。
- 工程动作与反证：对同一条长时 profiling 会话比较前台摘要和后台预计算。记录可复用比例、废弃摘要成本、缓存写入 token 和 P95 暂停时间。若用户更新实验约束后仍命中旧摘要，或预计算总成本高于节约的时间，修正失效条件或关闭该优化。

### `https://github.com/badlogic/pi-mono`

- 原文：[Pi 仓库，课程链接现跳转到 earendil-works/pi](https://github.com/badlogic/pi-mono)
- 范围：读提交 `275dde410065080d291de45fbe2110272d10de23` 的 `prepareCompaction` 与文件操作累计逻辑，对照 Compaction Reference 的持久化和分支摘要。[固定源码](https://github.com/earendil-works/pi/blob/275dde410065080d291de45fbe2110272d10de23/packages/coding-agent/src/core/compaction/compaction.ts) · [实现文档](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md)
- 机制与证据：从规范化的会话投影选切点，保存摘要及 `firstKeptEntryId`。再次压缩从上次保留部分继续，并使用之前的摘要。默认代码累计已读与已改文件。扩展提供的摘要需管理自己的字段，默认代码不会自动累加其文件列表。
- 工程动作与反证：在两个 kernel 备选分支之间切换，并强制两次上下文压缩。核对已改文件、运行记录和分支的父状态是否可恢复。若扩展摘要后混用分支日志或漏掉失败改动，补结构化文件及运行引用再测。

### `https://github.com/NousResearch/hermes-agent`

- 原文：[Hermes Agent 仓库](https://github.com/NousResearch/hermes-agent)
- 范围：读提交 `8cd3f3d3e692e2b9268b1a5cd9a80a69af67c8e2` 的压缩源码，重点是 token 尾部切点和候选无法缩短时的补救，与项目的四阶段说明交叉核对。[固定源码](https://github.com/NousResearch/hermes-agent/blob/8cd3f3d3e692e2b9268b1a5cd9a80a69af67c8e2/agent/context_compressor.py) · [项目文档](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/developer-guide/context-compression-and-caching.md)
- 机制与证据：保护首尾并先裁旧工具输出。尾部切点与工具调用／结果的原子组对齐，保留锚点会超过软上限。补救函数先尝试清理较旧输出与缩短摘要。只有仍含用户消息且估算严格缩小时才返回候选。估计长度与后端真实输入仍需比对。
- 工程动作与反证：准备刚返回大截图或长 profiler 日志、模型尚未处理的测试。压缩后检查调用与返回仍配对，后端接受请求，助手能找到截图对应的 stream 和 event。若压缩成功却漏掉新结果中的因果证据，先保留该工具组并把早期大输出改存为可读工件。

### `https://github.com/All-Hands-AI/OpenHands`

- 原文：[OpenHands 仓库，课程链接现跳转到 OpenHands/OpenHands](https://github.com/All-Hands-AI/OpenHands)
- 范围：读主仓职责说明，以及 SDK 提交 `365f886933cfe70e4c7ae44722699ab1cc8f411e` 的 `LLMSummarizingCondenser`。核对事件窗口、工具原子边界和压缩结果字段。[固定 SDK 源码](https://github.com/OpenHands/software-agent-sdk/blob/365f886933cfe70e4c7ae44722699ab1cc8f411e/openhands-sdk/openhands/sdk/context/condenser/llm_summarizing_condenser.py)
- 机制与证据：当前主仓说明将代理事件与会话 SDK 放在独立仓库。Condenser 根据请求、事件数或 token 阈值选忘记范围，保护开头的系统提示，用 `manipulation_indices` 对齐原子边界，并记录被省略事件的 ID、摘要和插入偏移。精读范围是当前 SDK，不能从中推断旧课件运行路径全部相同。
- 工程动作与反证：保留 benchmark 原始事件流，给模型提供由摘要和近期事件构成的视图。按被移除事件 ID 回放。若工具回传找不到配对调用，或评分脚本无法从事件 ID 重建证据，先修视图生成再比较省下的 token。

### `https://huggingface.co/Qwen/Qwen3.8-27B/blob/main/chat_template.jinja`

- 原文：[Qwen3.8-27B chat_template.jinja](https://huggingface.co/Qwen/Qwen3.8-27B/blob/main/chat_template.jinja)
- 范围：读取原始 174 行模板，文件页显示提交 `3756181`。核对图像、视频、工具结果、历史思考与生成提示的序列化。未启动服务。[原始模板](https://huggingface.co/Qwen/Qwen3.8-27B/raw/main/chat_template.jinja)
- 机制与证据：图像与视频内容写入各自的视觉占位符，system 角色内容不接受这些类型。连续 tool 结果放入同一 user 边界内的多个 `tool_response` 块。模板将最近真正 user 问题与这种工具结果区分，`preserve_thinking` 未定义或 true 时保留历史思考文本，显式 false 时在最近用户问题前省去它。网关是否预设该参数需要实测。
- 工程动作与反证：为多张 profiler 图、并行工具返回和用户改口构造模板测试。检查角色顺序、图像数、调用与返回对应、历史裁剪后的 token 前缀。若框架声称兼容却生成不同模板序列，先对齐输入协议再做服务比较。

### `https://huggingface.co/datasets/likaixin/ScreenSpot-Pro`

- 原文：[ScreenSpot-Pro 数据卡](https://huggingface.co/datasets/likaixin/ScreenSpot-Pro)
- 范围：2026-11-13 读取数据卡 Data Details，核对应用、软件版本、系统、图标与文本样本计数。网页未提供可用数据预览。本条未下载图像。
- 机制与证据：数据卡按专业应用与平台列出目标数量。同为开发工具，VS Code 在 macOS 上有 22 个图标目标与 33 个文本目标，PyCharm 有 38 与 40。只看整体 grounding 分数会掩盖文本、图标和软件版本之间的差异。
- 工程动作与反证：给 Nsight 和指标看板截图建立按图标、文本标签、窗口缩放和软件版本的分组。若模型正确找到控件却仍定位错 kernel，另加 trace 关联任务验证，保持两种指标各自可解释。

### `https://huggingface.co/datasets/osunlp/Multimodal-Mind2Web`

- 原文：[Multimodal-Mind2Web 数据卡](https://huggingface.co/datasets/osunlp/Multimodal-Mind2Web)
- 范围：读 Dataset Summary、Data Splits、Data Fields 与公开预览。按数据卡核对截图、HTML、动作和候选字段。
- 机制与证据：每行动作把截图与动作前 HTML 对齐。测试拆分为同站不同任务、不同站点、不同领域。作者说明三个测试集经人工检查可见性和渲染。清洗后正例候选可能为空，原始目标仍可从 raw HTML 找到。
- 工程动作与反证：保存 profiler 导航的原始截图与结构化事件、过滤后的候选列表及点击目标。按软件和 workload 家族做留出。若真实目标在候选生成时已被去掉，计入候选召回失败，然后用 oracle 目标独立测动作预测。

### `https://huggingface.co/moonshotai/Kimi-K3`

- 原文：[Moonshot Kimi-K3 模型卡](https://huggingface.co/moonshotai/Kimi-K3)
- 范围：2026-11-13 读 Model Overview、Model Architecture、Evaluation Results 与相关脚注。技术报告另有登记。
- 机制与证据：卡片列出 69 层 KDA 和 24 层 Gated MLA、1,048,576 token context、MXFP4 权重与 MXFP8 激活。Kimi-K3 的 OSWorld-Verified 为 84.8，OSWorld 2.0 为 58.3，属于两个任务集。卡片规定 Kimi 评测统一使用 max effort，并在一些视觉任务分开报告有无工具的分数。
- 工程动作与反证：将同一 profiling 任务做成截图、截图加脚本工具、结构化导出三组，匹配 effort、超时和人工检查。若截图组能读图但不能复述指标定义或正确追溯事件，分开评估视觉识别与性能推断。

### `https://huggingface.co/Qwen/Qwen3.6-27B`

- 原文：[Qwen3.6-27B 模型卡](https://huggingface.co/Qwen/Qwen3.6-27B)
- 范围：2026-11-13 读视觉评测、服务配置、Tool Use 和 Preserve Thinking。此处是 3.6-27B，模板条目为另一版本。
- 机制与证据：模型卡列出缺省 262,144 token 上下文，并说明缺省只保留最后一轮用户任务的思考块，可显式开启跨轮保留。服务示例把 reasoning parser、tool-call parser 和 MTP 当作各自配置项。模型卡的 AndroidWorld 70.3 依赖其所用评测设置。
- 工程动作与反证：先在同一后端验证 reasoning 与 tool parser，再固定提示测 MTP 的接受率、P95 ITL 和任务成功率。若 decode 更快但工具调用漏参数或完成率下降，把协议变更从优化实验中剥离。

### `https://github.com/OSU-NLP-Group/Mind2Web`

- 原文：[Mind2Web 作者仓库](https://github.com/OSU-NLP-Group/Mind2Web)
- 范围：读 README 的数据使用、Candidate Generation 与 Action Prediction 部分，核对作者发布的基础预测产物。
- 机制与证据：用候选评分器缩小 DOM 元素集合，再做动作预测。仓库列出其 DeBERTa-v3-base 发布产物 Recall@50 约 85%，动作评测通常给 50 个候选。因此下游成功会受候选覆盖率约束。
- 工程动作与反证：为源码和运行 trace 建两级定位器，先选可能相关的 kernel 或 event，再选动作。比较真实候选、oracle 候选和扩展候选。若 oracle 提升很小，将预算投到解释和实验决策层，避免只调召回。

### `https://github.com/OSU-NLP-Group/Online-Mind2Web`

- 原文：[Online-Mind2Web 作者仓库](https://github.com/OSU-NLP-Group/Online-Mind2Web)
- 范围：读 README 的 WebJudge、与人工一致率表、评估输入要求与发布结果说明。论文条目另有记录。
- 机制与证据：WebJudge 先提取成功条件和关键截图，再按任务、画面与真实动作判结果。仓库要求从规定网站开始，评审输入排除模型自己写的最终答复。表中 WebJudge-7B 的平均人工一致率为 87.0%，实际任务完成情况仍需独立工件支持。
- 工程动作与反证：给性能实验裁判提供同一基线、事实动作和关键 trace 片段。另做注入自报成功叙述的对照。若仅增加自述便显著改变判决，要求验证程序输出与实际工件，再重测裁判可靠性。
