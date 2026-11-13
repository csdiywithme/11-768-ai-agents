# L03 长上下文模型实例与 L10 检索版本补读

### `https://qwen.ai/blog?id=qwen3.8-flash-next`

- 原文：[Qwen3.8-Flash-Next 课件链接](https://qwen.ai/blog?id=qwen3.8-flash-next)
- 状态：替代版本精读
- 范围：登记页可打开，浏览器工具没有提取正文。精读阿里云官方同标题发布文章的 Attention、Gated Residual、Embedding、Optimization 及评测部分，交叉核对 Qwen 官方模型卡。两个发布渠道的文本一致性未逐段比较。[阿里云发布正文](https://www.alibabacloud.com/blog/qwen3-8-flash-next-a-new-architecture-towardsultimate-cost-efficiency_603501) · [Qwen 模型卡](https://huggingface.co/Qwen/Qwen3.8-Flash-Next)
- 机制与证据：每四个注意力层中有三层用 GDN 保存定长历史状态，另一层用 QSA 按 micro-block 筛选历史。QSA 在各层独立压缩索引。N-gram embedding 另有 51B 参数，可卸载到主机并尝试异步预取。作者在 1M token 报告 QSA kernel 的 prefill 和 decode 最高分别加速 7.6 倍、4.9 倍。对比 Qwen3.7-Plus 的 prefill 吞吐 8.6 倍对应 1M token、90% 前缀命中和完整模型比较。
- 工程动作与反证：在同模型上把压缩索引、稀疏 attention 和主机预取分别计时。前缀命中率按 0%、50%、90% 分组，并控制输入长度与并发。给主机预取标出 PCIe 拷贝和 GPU 等待区间。若低命中率下 TTFT 上升，或 memcpy 仍落在关键路径，分别调整调度和预取假设。

### `arxiv:2602.15763`

- 原文：[GLM-5 技术报告，arXiv v2](https://arxiv.org/html/2602.15763v2)
- 范围：§2.1–2.4、§3 中的 DSA RL insights、§6 对应评测条件。重点复核表 4–5 的 GLM-9B 消融与作者对 GLM-5 的最终设计说明。
- 机制与证据：GLM-5 最终采用 DSA。另用 GLM-9B 的 1:1 全注意力与 SWA 设计比较层安排。表 5 中 RULER@128K 的全注意力为 75.28，固定交替为 44.93，搜索层安排为 69.59。SimpleGDN 的 HELMET-ICL@128K 为 81.84，高于全注意力的 77.36，但 RepoQA@128K 从 65.83 降到 58.50。作者还观察到，训练与 rollout 的 DSA top-k 索引选择不一致会影响 RL，确定性实现有速度成本。
- 工程动作与反证：给同一 checkpoint 的候选 top-k kernel 保存固定输入和候选相近的边界样本，核对多次执行和不同后端的索引，再测任务完成率、TTFT 与稳定性。若对齐候选索引后输出偏差仍在，继续查 logits、精度和采样状态。训练的收益需要用训练或 rollout 指标另测。

### `https://huggingface.co/zai-org/GLM-5.3-Flash`

- 原文：[Z.ai GLM-5.3-Flash 模型卡](https://huggingface.co/zai-org/GLM-5.3-Flash)
- 范围：2026-11-13 可见版本。阅读 Introduction、Serve Locally、Note 和评测脚注。架构字段另见配置条目。
- 机制与证据：作者列出 320B 总参数、18B 激活参数，采用稀疏和线性注意力的混合架构与 mHC。卡片指定 `reasoning_effort` 可选 low、high、max，缺省 max。`clear_thinking` 在模板里缺省为 false。脚注中 NL2Repo 给 1M context，DeepSWE 给 400K context，并使用指定的 harness 和超时。对比时这些条件会改变可用输出预算和任务流程。
- 工程动作与反证：用同一组多轮调优任务做 2×2 实验，分别控制 effort 和是否清除上一轮思考。记录实际输入输出 token、连续回合前缀命中率、定位正确率及 P95 完成时间。若清除历史后误判增多，查失败回合依赖哪条已验证信息，再决定是否用显式证据字段保留它。

### `https://huggingface.co/zai-org/GLM-5.3-Flash/blob/main/config.json`

- 原文：[GLM-5.3-Flash config.json](https://huggingface.co/zai-org/GLM-5.3-Flash/blob/main/config.json)
- 范围：2026-11-13 读取公开 `main` 的原始 JSON，并解析 `text_config`。模型 API 同次返回 HEAD `46a050bec13002ad29ee795c16e61cf4fa3c4a9c`。未运行对应引擎核对分配。[原始 JSON](https://huggingface.co/zai-org/GLM-5.3-Flash/raw/main/config.json)
- 机制与证据：配置有 45 个 text layers，`layer_types` 中 34 个 `linear_attention` 与 11 个 `deepseek_sparse_attention`。`max_position_embeddings` 为 1,048,576，`index_kpool=4`、`index_topk=2048`、`kv_lora_rank=512`。前几层的类型呈三层线性加一层稀疏。配置给出结构线索，实际分配粒度、状态精度和可服务并发由后端及服务配置共同决定。
- 工程动作与反证：在容量表中分别计算稀疏层的 KV 与 index 状态、线性层的 recurrence 状态，再用引擎计数器核对单请求长度递增时的显存增量。若观察斜率比估计大，先定位实际 cache dtype、页对齐、共享前缀复制与临时工作区。

### `https://thinkingmachines.ai/news/introducing-inkling/`

- 原文：[Thinking Machines Lab Inkling 发布文](https://thinkingmachines.ai/news/introducing-inkling/)
- 范围：2026-07-15 发布。重点阅读 Controllable thinking effort、Architecture、Training、RL at scale 及 Benchmarking 脚注。
- 机制与证据：Inkling 是 975B 总参数、41B 激活参数的 MoE。作者在注意力层中按 5:1 交错滑窗和全局层，使用 8 个 KV heads，报告 relative positional embedding 在该模型上优于其 RoPE 尝试，并在若干路径加入短卷积。后训练在样本中调整 effort 系统消息和每 token 成本，让模型学会控制思考用量。部分评测采用内部 harness 或调整提示，需按脚注判断同条件性。
- 工程动作与反证：比较同一任务集的 effort 配置，分别包含证据齐全的算子审查和证据不足的跨模块定位。记录有效定位率、失败重试、实际生成 token 和总耗时。若低 effort 节省生成却增加错误回滚或重复 profiling，保留按证据缺口升级 effort 的对照。

### `arxiv:2606.19348`

- 原文：[DeepSeek-V4 技术报告，arXiv v1](https://arxiv.org/html/2606.19348v1)
- 范围：§2.3、§3 的混合 KV 管理与磁盘缓存、图 1–3。核对注意力 FLOPs 与 KV 体积各自的参照。
- 机制与证据：CSA 先把多 token 的 KV 压为一个条目，再用稀疏索引选条目。HCA 压缩更强，再对压缩条目做密集注意力。局部滑窗补充细粒度信息。报告给的 1M token KV 约为 BF16 GQA8、head dim 128 参照的 2%，同时采用 RoPE 维 BF16、其余维 FP8 的混合存储。磁盘复用在完整压缩块处停止。滑窗状态的全量缓存、周期检查点、零缓存策略会改变 I/O 与尾部重算。
- 工程动作与反证：用相同请求回放分别改变前缀命中长度、最后压缩块剩余 token 和存储速度。分项测 KV 字节、读盘时间、尾部重算与 P95 TTFT。若周期检查点只在高复用下胜出，将缓存策略的适用条件写入结论。若 FLOPs 下降但 GPU kernel 时间没有相应改善，查索引与内存访存。

### `https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash`

- 原文：[DeepSeek-V4-Flash 官方模型卡](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)
- 范围：2026-11-13 可见版本。阅读 Introduction、Evaluation Results、Chat Template、How to Run Locally。这里核对发布契约与使用条件。
- 机制与证据：Flash 为 284B 总参数、13B 激活参数，声明 1M context。卡片比较不同思考模式的 1M MRCR 与 CorpusQA。发布说明提供专用编码、解析脚本及测试样例，Think Max 本地运行建议给至少 384K context。模型、模板、思考模式和上下文设置共同影响可复现的调用。
- 工程动作与反证：给服务升级加一个小型协议回归集，覆盖历史 tool call、tool result 和思考模式切换。先核对编码和 token 数，再测多轮生成。若同模型换模板后性能或任务分数变化，回放相同 token 序列，确认差异是否来自推理引擎。

### `https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-White-Paper.pdf`

- 原文：[NVIDIA Nemotron 3 白皮书，2025-12-24](https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-White-Paper.pdf)
- 范围：13 页 PDF，重点读 §2.1–2.3、图 1–3 和表 1。该日期发布了 Nano，Super 与 Ultra 的完整模型资料计划随后发布。
- 机制与证据：Mamba-2 层保留定长状态，少数注意力层保留全局交互。Nano 在 8K 输入、16K 输出场景的图 2 对照中，相对 Qwen3-30B-A3B-Thinking-2507 报告 3.3 倍每卡输出吞吐。白皮书将少量 token 批次的专家权重读取和大量 token 批次的 all-to-all 分开讨论。LatentMoE 通过减小路由表示维度减少载荷，同时增加专家数量和 top-K，将节省的预算用于能力。表 1 是作者的近似等活跃参数对照。
- 工程动作与反证：对 MoE 调优先做每步 token 数扫描，将权重读取、专家计算和 all-to-all 关键路径时间拆开。若扩大 batch 后通信比例没有上升，先检查专家分布、重叠和实际互联带宽。若估计的显存斜率偏低，把保留的注意力层 KV 加入预算。

### `arxiv:2603.04384v1`

- 原文：[AgentIR，课件链接 arXiv v1](https://arxiv.org/html/2603.04384v1)
- 范围：§3、§4.2–4.4、§5.1–5.3。此前未带版本号的登记项读的是 v3。本条复查 v1 的表 2–3 和替代检索信号。
- 机制与证据：v1 在 BrowseComp-Plus 使用可控语料、对比相同 retriever 骨干的输入和训练。Tongyi-DR 下只给当前查询为 59.40% 准确率，加入全局问题为 63.25%，给当前查询及驱动本次查询的信息为 66.27%。嵌入历史查询、历史推理和文档为 60.00%，受长度限制，通常只保留近约三轮。带 visit 工具时，各种设置的调用数也要计入比较。
- 工程动作与反证：索引同一批仓库与 profiler 文档，对比三种查询输入，即符号名、符号名加全局任务、符号名加当前需排除的瓶颈及版本证据。固定检索和工具预算，测正确证据 span、最终定位及调用次数。若符号名在精确报错检索中更稳，保留按查询类型选择输入的实验。

### `arxiv:2504.12516v1`

- 原文：[BrowseComp，课件链接 arXiv v1](https://arxiv.org/html/2504.12516v1)
- 范围：无版本登记项已读同一 v1 正文。本条复查 §4.2–4.4、表 3 和图 4，专门记录置信度与并行取样。
- 机制与证据：作者对每题生成 64 个含自报置信度的输出，比较多数投票、置信度加权和选择最高置信输出。图 4 的最高置信选择在该封闭答案任务中表现最好，表 3 同时显示该设置的绝对置信度校准有误差。相对排序与绝对可靠性是两个指标。
- 工程动作与反证：在固定预算下比较单分支与两条独立排查路径，随后用原始日志和代码定位裁决。统计高置信错误中两个分支是否共用同一证据。若两路对同一错误假设重复投票，改为让第二路优先查能推翻首个候选的观测。

### `arxiv:2508.06600v1`

- 原文：[BrowseComp-Plus，课件链接 arXiv v1](https://arxiv.org/html/2508.06600v1)
- 范围：无版本登记项已读同一 v1 正文。本条复查 §4.8.4 与表 6–7，记录扩大索引的验证边界。
- 机制与证据：增加 FineWeb-Edu 文档后，作者报告 BM25 的检索分数提高，两个向量检索器的分数下降。在 Qwen3-32B 配 Qwen3-Embedding-8B 的该对照中，最终准确率为 10.36% 和 7.11%。新增文档未被人工标注，部分表面负例可能含有效证据。正文对文档数量和倍数的表述有出入，这里仅采用扩大语料的方向及表 7 数值。
- 工程动作与反证：为同一框架问题比较固定版本的官方索引、加入邻近版本和外部文档后的索引。人工复核新增 top-k 中原先未标注的证据，再判断召回变化。若扩大覆盖提高文档命中却使错误版本的操作建议增加，增加版本过滤后重新比较。

### `arxiv:2509.00496v2`

- 原文：[ResearchQA，课件链接 arXiv v2](https://arxiv.org/html/2509.00496v2)
- 范围：无版本登记项已读同一 v2 正文。本条复查 §5、表 3 的主结果、表 4–5 的篇幅和漏项分析。
- 机制与证据：评估按研究问题的 rubric 项计覆盖，作者把关键作品、局限和比较列为经常漏掉的类别。该实验的表 3 最高 coverage 为 75.29%，即使最高分仍有条目缺失。评分同时受题目、参考规则和回答篇幅影响。
- 工程动作与反证：给技术选型摘要设三个可以独立核对的检查，即是否引用实际版本、是否说明失效条件、是否比较一个可运行基线。给短和长两种答案同样的证据预算，并抽查遗漏类别。若较长报告涨分却没有提供新的核对入口或边界证据，调整评分规则和长度控制。
