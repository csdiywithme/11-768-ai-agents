# L3 课件、压缩续跑与推理服务条件

核验日期 2026-11-13。共 12 项登记来源。课件报价为 2026-08-30 的历史快照，厂商价格条目记录本次读取的页面和计费条件。官方录像检查静帧，字幕正文来自单列的第三方页面。

### `https://www.cmu-agents.com/slides/lecture-03-long-context.pdf`

- 原文：[L3 官方课件](https://www.cmu-agents.com/slides/lecture-03-long-context.pdf#page=47)
- 范围：54 页 PDF。重点 pp.3–13、18–25、29–36、38–44、46–51。核对 p.47–49 的压缩图及 p.50–51 的漂移和续跑验收。文件 SHA-256 为 `cd08b3463430b78a144a24de611ec9a58befae32c5d2109471d89934fbf2f926`。
- 机制与证据：历史每轮增长时，累计重传输入为二次增长。p.4 的 1,500 个 OpenHands 会话平均 77,922 token，工具结果占 37%，工具调用 20%，系统及工具定义 23%。该样本用于决定先测何处的输入增长。p.47 把压缩验收分为后续动作保真、状态预算、证据可恢复。p.48 原样保留目标约束和最近动作，编码检查点，外置原件。p.49 明确触发、选段、替换及失败恢复。p.50 举出 CUDA 12.4 经数次摘要变成无约束的示例。p.51 直接用续跑行为验收。
- 工程动作与反证：构造带精确 CUDA 版本、基线命令、已排除假设、工单中途修订及原始 trace 路径的回放。固定模型比较全历史、自由摘要、结构化检查点加证据引用。测第 1、3、5 次压缩后的下一实验、错误重复、跨轮指令及可回查状态。若报告更短但丢失一个会改变调优动作的条件，按决策失败计。

### `youtube:AiwCCvFW1uE`

- 原文：[L3 官方录像的相关时点](https://www.youtube.com/watch?v=AiwCCvFW1uE&t=4050s)
- 范围：用官方公开播放页和课表核对录像 ID 与标题。目视检查既有官方 storyboard 中约 1:07:30 与 1:08:20 的静帧。图片 SHA-256 为 `339add9df6c40bba98bd1abf54e60f14b27d8ebbdbd0c6de86894fda1c1c4bc0`。先前公开字幕请求的正文为空；本次未获取连续画面或官方音轨。
- 示例与证据：画面先是 Compaction as state estimation，后出现 What survives compaction，与课件 pp.47–48 顺序一致。第一张是目标和限制的示意，第二张区分 anchors、checkpoint、recent tail 和外部证据。图中没有对实际方案的性能结论。
- 工程动作与反证：把画面和课件定位当作设计问题的来源。实现后单独测压缩前后的动作和成本。若原始日志仍存在但无法用保存的引用找到所需证据，判可恢复性失败。
- 状态：静帧已核

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-03/`

- 原文：[L3 第三方英文字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-03/)
- 范围：扫描 72 个英文合并段的起始内容，从 00:07.120 到最后一段起点 1:14:52.960。逐段精读 01:00–09:32 前、52:06–1:06:09 前、1:06:09–末段。原 HTML SHA-256 为 `fb1a0f78d2a2e73a2cb87d0933b3e48f991aa4b8f29450cc5afa49b55375cb60`。数字与机制对照正式 PDF、论文和厂商文档。
- 机制与证据：1:04:24 讨论写入动态时间戳与模型切换对复用的影响。1:07:30–1:11:53 讲压缩状态、锚点、最近动作和恢复。1:12:22 起讲师报告内部早期压缩方案在当时的单轮评测中总体分数未降，实用中发生重复开 PR 和遗漏对话中段的用户要求。这里保留为讲师叙述，字幕无法提供匹配的实验产物。
- 工程动作与反证：在多轮任务加入用户中途改变约束和已提交的外部动作标识。回放中记录重复提交、重复启动 GPU 任务及旧约束残留。若单轮分数保持但多轮的重复操作上升，改状态保存与幂等判断，再测。
- 状态：字幕文本已读

### `https://platform.claude.com/docs/en/build-with-claude/prompt-caching`

- 原文：[Claude 官方 Prompt caching 文档](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- 范围：2026-11-13 页面。核对自动与显式断点、顺序、20 个位置回看、最小长度、5 分钟与 1 小时 TTL、首个并行请求生效时点、usage、计费和模型例外。
- 机制与条件：缓存基于工具、system、消息顺序形成的匹配前缀。回看检查的是已写入的条目。断点移动 20 个位置可越过上一条目。默认 TTL 5 分钟，文档另支持 1 小时写入，具体最低 token 数因模型和平台而变。并行请求可在首个响应开始前读不到刚写的前缀。用读、写和普通输入 usage 区分计费，5 分钟写入一般为普通输入 1.25 倍，1 小时为 2 倍，读取倍率有模型例外。
- 工程动作与反证：把时间、动态状态和最新日志留在后缀。对代表性任务运行短间隔、超过 TTL、并行首轮、工具 schema 修改等条件。记录缓存写、读、普通输入、TTFT 和总价。若预期稳定的前缀仍频繁重写，按断点回看范围、缓存阈值及失效字段定位。若长 TTL 的写入成本高于实际复用，收窄缓存边界。

### `https://api-docs.deepseek.com/quick_start/pricing/`

- 原文：[DeepSeek 官方 Models & Pricing](https://api-docs.deepseek.com/quick_start/pricing/)
- 范围：2026-11-13 页面。核对模型名和底层版本、每百万 token 价格、峰谷时段、独立 Search／Fetch 计费。
- 机制与条件：本次页面的 `deepseek-flash` 对应 DeepSeek-V4.1-Flash-1017。USD／百万 token 的谷／峰价为输入命中 0.003／0.006、未命中 0.15／0.30、输出 0.60／1.20。课件 p.40 用的是 2026-08-30 的 V4 Flash。文档还说明旧 Flash 名称仍接受，但由新模型服务。Search 和 Fetch 是按成功调用另计的 API。
- 工程动作与反证：给同一 workload 账单记录实际模型版本、UTC 请求时刻和命中 token 数。若跨峰谷、产品或后台模型，分段计算，只有延迟与任务成功也相近时才比较成本。

### `https://docs.z.ai/guides/overview/pricing`

- 原文：[Z.AI 官方 Pricing](https://docs.z.ai/guides/overview/pricing)
- 范围：2026-11-13 页面。核对 Latest Models、Text Models、Cached Input Storage 和 Built-in Tools。
- 机制与条件：GLM-5.2 为 USD／百万 token 普通输入 1.40、缓存输入 0.26、输出 4.40，与课件该行一致。GLM-5.3-Flash 同次页面为 0.15、0.03、0.50。缓存存储列为限时免费。Web Search 单独按用量计费。定价页没有给实测命中率。
- 工程动作与反证：使用指定模型及实际 usage 复算调优助手成本，另记检索次数、缓存存储条件和成功率。如果只改变了模型选型或账单计费，保持该比较的条件，不把差异归给 harness。

### `https://www.kimi.ai/resources/kimi-k3-pricing`

- 原文：[Kimi 官方 K3 价格说明](https://www.kimi.ai/resources/kimi-k3-pricing)
- 范围：2026-11-13 页面。核对 API 价格表、自动上下文缓存、长度和会员／API 的边界。
- 机制与条件：`kimi-k3` 每百万 token 缓存命中输入 0.30 USD，未命中 3.00，输出 15.00。价格说明称支持自动上下文缓存，给出 1,048,576 token 窗口，页面没有给本地性策略、可保证的命中率或任务有效长度。会员权益使用不同口径。
- 工程动作与反证：以同类多轮工单的实际 usage 测冷暖条件。将完整请求 token 与缓存计费 token 各记一列。若输出量主导或短任务缺少共同前缀，缓存单价的折扣不能说明整单节省，按账单重算。

### `https://developers.openai.com/api/docs/models/compare`

- 原文：[OpenAI 官方 Compare models](https://developers.openai.com/api/docs/models/compare)
- 范围：2026-11-13 页面已显示的 GPT-6 Astra、GPT-6.1 Sol、GPT-6 Luna 比较卡。核对普通输入、缓存读取、写入、输出、窗口与端点。
- 机制与条件：此次页面的 GPT-6.1 Sol USD／百万文本 token 为普通输入 2.00、缓存输入 0.10、缓存写入 2.50、输出 10.00。与课件 p.40 的 GPT-5.6 Sol 是不同模型和日期。型号卡的窗口与最大输出是分别列出的上限。
- 工程动作与反证：核对请求模型、服务档位、端点和账单实际读写字段。比较稳定模型和按任务路由时同时算重新写缓存的成本。如果路由节省输出或普通输入价格却导致额外缓存写入和重试，应以整单总成本与通过率比较。

### `https://docs.nvidia.com/megatron-core/developer-guide/latest/user-guide/features/context_parallel.html`

- 原文：[NVIDIA Megatron Core，Context Parallel Overview](https://docs.nvidia.com/megatron-core/developer-guide/latest/user-guide/features/context_parallel.html)
- 范围：2026-11-13 页面。核对 TP2CP2 图、KV 通信、175B 对照、启用条件。未运行训练。
- 机制与条件：CP 按序列切输入和激活。注意力需访问同一序列的 K/V，前向 gather，反向对应梯度 reduce-scatter。文档说明该方案用环形 P2P 实现，可通过 MQA/GQA 降低 KV 通信量。175B 图比较 TP/CP 组合与 TP8CP1 的完整激活重计算。适用性还取决于模型路径、Megatron Core 和 Transformer Engine。
- 工程动作与反证：固定模型、总 GPU、全局 batch token 和精度，比较 CP、更多 TP 或重计算。记录每步时间、峰值显存、暴露通信和当前互连。若仅增加可装入长度但牺牲稳态吞吐，按容量和吞吐分别决策。

### `arxiv:2401.09670`

- 原文：[DistServe，arXiv v3](https://arxiv.org/html/2401.09670v3)
- 范围：§§3–4、6.1–6.4，表 1–2 和图 8–11。核对 SLO 口径、阶段放置、KV 传输和模拟对照。
- 机制与证据：把 prefill 和 decode 放到不同 GPU 资源，分别选 batch 和并行策略，并考虑 KV 搬运。4 节点、32 张 A100、25 Gbps 跨节点带宽的试验，以不少于 90% 请求同时满足 TTFT/TPOT 为条件。ShareGPT 的 OPT 系列相对论文 vLLM 基线可承载 2.0–4.6 倍请求率。图 10 的 OPT-175B 请求中传输占总时延不足 0.1%，该放置方案用同机分阶段安排减少跨节点传输。
- 工程动作与反证：在已有引擎上先做 P/D 共置基线。对相同到达、长度、GPU 数和服务阈值回放，量化排队、执行、handoff 各段。若新拓扑的跨节点 KV 搬运、长短负载偏斜或 idle GPU 吃掉收益，继续调并行配比或保留共置。

### `arxiv:2309.06180`

- 原文：[PagedAttention，arXiv v1](https://arxiv.org/html/2309.06180v1)
- 范围：§§4.1–4.5、6.1–6.5、7.1–7.2，图 12、15、18。核对块表、共享、相对负载和单核开销。
- 机制与证据：固定 token 数的逻辑 KV block 映射到非连续物理块，按需分配，可通过引用计数和写时复制共享。§6.2 的 ShareGPT 近似延迟条件下，服务请求率相对知道输出长度的 Orca Oracle 提升 1.7–2.7 倍。§7.1 的独立 attention kernel 对比高度优化的 FasterTransformer 增加 20–26% 延迟。服务受益来自分配效率和可支持更大 batch，随长度、块和负载而变。
- 工程动作与反证：在固定 checkpoint、精度和到达序列下记录逻辑页、实际占用、回收、抢占、TTFT/TPOT 和并发。测块大小时同时统计最后一块碎片与映射访存。若 batch 容量不再受 KV 限制，先验证 kernel/调度开销再扩大变更范围。

### `arxiv:1706.03762`

- 原文：[Attention Is All You Need，arXiv v7](https://arxiv.org/html/1706.03762v7)
- 范围：§§3.1–3.5、4、5.1–5.2、6.1，图 1–2、表 1–2。核对注意力模式、路径长度与硬件。
- 机制与证据：以缩放点积形成注意力，分别处理 encoder self-attention、带因果掩码的 decoder self-attention 和交叉注意力。表 1 区分每层复杂度、最少串行步骤和跨位置路径。全 self-attention 每层 O(n²d)，限制邻域 r 后为 O(rnd)，位置之间的最长路径由 O(1) 到 O(n/r)。原论文实测条件为 WMT 翻译等任务。
- 工程动作与反证：对一个长输入请求分开估算 prefill 两两计算、单步 decode 的 KV 读和数据传输，再用 profiler 实测。若实测瓶颈是 HBM 或调度间隙，应把 FLOP 估算用作候选筛查，按实际阶段优化。
