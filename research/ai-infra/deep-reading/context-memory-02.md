# L3 长上下文机制与 L10 课件证据

核验日期 2026-11-13。L3 阅读 8 篇固定版本论文和 1 个工具文档。L10 两项档案资料用于验证课件案例中的事实关系。

### `https://digitalcollections.library.cmu.edu/cmu-collection/herbert-simon`

- 原文：[Carnegie Mellon University Archives，Herbert Simon](https://digitalcollections.library.cmu.edu/cmu-collection/herbert-simon)
- 范围：官方馆藏页 Biography、About the Collection、Finding Aid。读到的官方 HTML SHA-256 为 `3789b3017d544f042203d3c853a553b14d92da8f12d391a343381f17554c5ade`。未逐件读档案。
- 示例与证据：Simon 和 Allen Newell 是同事和长期研究伙伴，共著 1972 年的《Human Problem Solving》，共同获得 1975 年图灵奖。档案正文另阐述有限理性、满意解和以子目标推进决策。课件通过这些事实构造可回查的多步题。馆藏说明部分材料仅可现场查阅。
- 工程动作与反证：给调优工作规定终止条件，依据当前瓶颈和可承受的试验成本逐个排除。用同一问题检验分支式排查是否更早找到可复核原因。如果提前停止频繁漏掉对最终收益有决定性的限制，应调整阈值或新增分支。

### `https://www.nobelprize.org/prizes/economic-sciences/1978/summary/`

- 原文：[NobelPrize.org，1978 年经济学奖官方摘要](https://www.nobelprize.org/prizes/economic-sciences/1978/summary/)
- 范围：官方摘要中的获奖人、年份、奖项份额、获奖理由，以及相关发布入口。下载的官方 HTML SHA-256 为 `4ed831fec75898d5d2188322f605c197575a4302252e3713fbd450c93853bb4c`。
- 示例与证据：官方摘要确认 Herbert A. Simon 独得该奖，授予理由是他对经济组织决策过程的开创性研究。与 CMU 馆藏合用可区分获经济学奖的 Simon 和同获图灵奖的 Newell。课件用二者关系生成一个有确定答案的多跳问题。
- 工程动作与反证：把每条框架事实连到提供相应证据的版本或仓库。分别核性能比较、功能合入和发布日期。若上游博客确认了某能力，但 release 与实现日期未证实，保留该范围的未决字段。
- 状态：示例已核

### `arxiv:2004.05150`

- 原文：[Longformer，arXiv v2](https://arxiv.org/html/2004.05150v2)
- 范围：§§3.1–3.2、4.1、6、7，图 1–3、表 10–11。
- 机制与证据：在固定宽度窗口内做局部注意力，也可使用 dilation 扩展感受野。少量预选全局 token 与所有 token 双向相连，且用独立投影。局部项计算量约为 O(nw)。图 1 的向量化实现和 CUDA 实现随长度有不同表现。表 11 的 arXiv 摘要中，LED-large 的 4,096 和 16,384 版本 ROUGE-L 分别为 39.76、41.83。LED 的稀疏模式用于 encoder，decoder 仍看完整 encoder 输出。
- 工程动作与反证：以相同序列、模型条件记录 attention kernel 的 IO、峰值显存和端到端吞吐。测试把跨长距的代码与日志证据放在不同窗口。若局部窗口节省计算但错过目标关系，先测全局锚点或不同窗口，不据 encoder 结果推断 decode KV 收益。

### `arxiv:2006.16236`

- 原文：[Transformers are RNNs，arXiv v3](https://arxiv.org/html/2006.16236v3)
- 范围：§§3.2–3.4、4.1–4.3、图 1、表 1–3。重点读取因果递推和梯度实现。
- 机制与证据：将相似度写为特征映射内积，维护前缀矩阵累加和向量归一化状态。给定特征维度，decode 每步读取和更新固定大小状态。反向用累积计算避免存每个时刻的整张状态矩阵，输入与输出序列仍占内存。表 1 的逐像素 MNIST 生成由 softmax 的 0.45 到线性注意力的 142.8 images/s，作者利用更大并发，且 bits/dim 从 0.621 到 0.644。该吞吐只适用于其模型和基准。
- 工程动作与反证：对等精度下比较递推实现和分块实现的状态、prefill 时延、decode TPOT 与工作集。固定 batch 做第一组，再比较两种模型最大可用 batch。若吞吐提高完全依赖更大 batch，或质量测试的长距离约束失败，按相应条件限定收益。

### `arxiv:2102.11174`

- 原文：[Linear Transformers Are Secretly Fast Weight Programmers，arXiv v3](https://arxiv.org/html/2102.11174v3)
- 范围：§§3.1–3.2、4.1–4.2、6.1–6.3，图 2–3、表 2。核对状态容量与更新公式。
- 机制与证据：线性注意力可视为把键和值外积写进固定大小矩阵。键互相重叠会干扰查询。delta 规则先读旧值，以新旧差和可学习的写入强度更新。图 2 在 64 维键特征的合成试验中约从 60 个关联开始出现误差；图 3 固定 20 个可重复键，检查能否取回最近值。WikiText-103 的中型设置测试困惑度，sum 33.0、delta 31.5、标准 Transformer 29.6。
- 工程动作与反证：验证 recurrent 模型时覆盖很多相似 key、更新同一字段、重新读取较早字段、不同距离下的保真。记录状态容量与 dtype，再对照可用显存。若减少状态后更容易把旧值误当成新值，在外部保留原始证据或重测混合结构。

### `arxiv:2405.04434`

- 原文：[DeepSeek-V2，arXiv v5](https://arxiv.org/html/2405.04434v5)
- 范围：§§2.1.1–2.1.4、2.2、3.2.3、附录 C。核对式 9–19、表 1 和实际部署条件。
- 机制与证据：MLA 缓存联合压缩的 KV latent 和解耦的共享 RoPE key。计算中可以吸收内容路径的上投影，避免为历史 token 重建全部 K/V。表 1 每 token 元素数为 `(d_c+d_rope)×层数`，以 DeepSeek-V2 配置约等于 2.25 组 GQA。§3.2.3 在 8×H800 的整服务中用 FP8 权重、平均 6 bit KV、原 DeepSeek 67B 的长度分布，报告 5.76 倍最大生成吞吐。模型架构、量化和 batch 都发生变化。
- 工程动作与反证：先核当前后端实际存 latent 还是展开后 KV，记录 TP 复制、padding 和块分配。保持权重、模型、dtype、batch、长度和输入一致测试 fused MLA kernel，另做调大 batch 的服务实验。若 latent 减少字节但投影计算、通信或对齐消耗抹平 TPOT 收益，先测瓶颈转移再选下一项。

### `arxiv:2104.09864`

- 原文：[RoFormer，arXiv v5](https://arxiv.org/html/2104.09864v5)
- 范围：§§2.2–2.3、3.1–3.4、4.1–4.4、4.5.5，图 1–3、表 1–2。
- 机制与证据：按位置旋转 Q、K 成对维度，位置 m、n 的点积只出现相对位移的旋转。论文还给出与一类线性注意力结合的公式，归一化分母保留未旋转形式。WMT14 En-De 的 BLEU 为 27.3 到 27.5，同设置的 Transformer-base 对照。作者对较快收敛原因的解释仍有限，原文也没有给现代长窗口推理延迟结论。
- 工程动作与反证：验证 fused RoPE 和 paged KV 时固定 checkpoint 的旋转配对、频率、位置偏移和已旋转 K 的存储格式。检查短序列、长 offset、分块 prefill 接 decode。若整段运行通过而分块或 prefix reuse 出现 logits 差异，先追位置或 cache 写入格式。

### `arxiv:2306.15595`

- 原文：[Extending Context Window via Position Interpolation，arXiv v2](https://arxiv.org/html/2306.15595v2)
- 范围：§§2.1–2.3、3.1–3.5，表 3–6 和 passkey 的有效窗口定义。
- 机制与证据：将目标窗口内的位置缩放回原训练的范围，再做 RoPE，减少直接外推造成的异常注意力值。表 3 的 7B／16,384 在 PG19 上从零训练步骤的困惑度 112.13 到 200 步的 7.05。表 4 的有效窗口是各采样距离的成功率都至少为 20% 时达到的最远距离；7B、32K 配置在 200 步时到达该标准，600 步时为 18,432。
- 工程动作与反证：分别测后端允许的长度、任务实际可用长度和服务成本。长度与证据位置二维扫描，加入精确数字、跨段关系和引用检查。若能收 32K 且 passkey 偶尔成功，却达不到任务所需成功率或短任务回归，则回到较短窗口或用证据分块。

### `arxiv:2309.00071`

- 原文：[YaRN，arXiv v3](https://arxiv.org/html/2309.00071v3)
- 范围：§§3.1–3.4、4.1–4.5、附录 B.1、B.5，图 2–3、表 1–4。
- 机制与证据：PI 一起缩放各频率。YaRN 基于波长分段，保留更短波长，插值更长波长，中间过渡，并加 attention 温度修正。表 1 在 10 份 Proof-pile 文档、滑窗步长 256 上，Llama 2 7B 的 64K 模型在 128K 困惑度大于 10。基于 64K 模型再训练 200 步且将扩展因子改到 32 后，该值为 2.37。两者配置与训练均变化。
- 工程动作与反证：先核 serving 的实际 rope 配置，并按原长度、预期长度和最大长度测质量。若使用动态配置，比较一把 prefill 与逐段 prefill 共享前缀时 logits 和缓存。若 128K 困惑度改善但短输入行为变差或不同路径的 KV 不一致，限制配置和重用条件再验证。

### `arxiv:2305.19466`

- 原文：[NoPE Length Generalization，arXiv v2](https://arxiv.org/html/2305.19466v2)
- 范围：§§3–6、Limitations、附录 F.1–F.3，图 1–4、F.2。
- 机制与证据：同一 decoder-only 设置比较绝对位置、ALiBi、Rotary、T5 相对偏置和无显式位置。主要实验从头训练 10 类合成任务，每组用三种 seed，NoPE 与 T5 相对偏置在长度外推中靠前。论文分析 causal mask 与 BOS 允许模型形成隐式位置信息。附录 F 的 1.3B StarCoder 子集初步实验，NoPE 与 ALiBi 在 1,024 训练长度之外扩到约 1,800，ALiBi 在更长区域相对稳定。主实验和附录的指标分别是任务正确率与困惑度。
- 工程动作与反证：对已有 checkpoint 先按其训练的架构测量；验证位置方法时用同配方训练的模型。把长度、证据距末尾、位置冲突和真实调优任务纳入对照。若合成结果提升而实际长代码与日志上的决定错误，保留结论为训练条件下的现象。

### `https://github.com/gkamradt/LLMTest_NeedleInAHaystack`

- 原文：[Needle In A Haystack，课程登记的仓库入口](https://github.com/gkamradt/LLMTest_NeedleInAHaystack)
- 范围：登记入口重定向至作者仓库。读 v2 README 的 sweep、任务、结果重建和历史运行说明，核对 UUID chain 代码入口。未固定 commit，未运行示例。
- 机制与证据：按上下文长度和事实插入深度组合跑，任务有单事实、多事实和 UUID 链。每条 JSONL 记录目标和实际深度、种子、回答及重建所需信息。README 说明 v1 的多事实深度会被前面的插入内容挤偏，v2 修订了计数。历史 2023 结果格式与 v2 不同。
- 工程动作与反证：固定源文档、tokenizer、插入规则和请求版本，用每个格子的实际 token 深度验收。把单个精确字段和必须跨工具关联的多个字段分开评分。若只在均匀合成干扰中稳定，改用实际日志分布并保留失败样本。
- 状态：示例已核
