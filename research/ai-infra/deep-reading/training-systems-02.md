# L12 · 异步训练、运行时对齐与重复计算

### `arxiv:2312.07104`

- 原文：[SGLang 论文 v2](https://arxiv.org/html/2312.07104v2)
- 范围：§§2–4、6.2–6.3，附录 A 和 D.2 的调度与前缀共享。
- 方法与证据：RadixAttention 把可复用的前缀 KV 挂在 radix tree，跟踪引用，并在有显存压力时淘汰空闲叶。队列按共同前缀重排时可减少重复 prefill。附录的最优情形以已知请求、给定缓存条件为前提。正文包含多个任务和旧版基线，峰值吞吐倍率也包含约束解码等其他改变。
- 工程用法：用相同到达序列比较 FIFO、cache-aware 顺序和只改前缀模板三种方案。保留命中 token、缓存淘汰、TTFT p95 和长尾请求年龄。若命中升高但晚到请求因重排超出 SLO，引入年龄阈值并重新测有效吞吐。

### `arxiv:2505.24298`

- 原文：[AReaL v5](https://arxiv.org/html/2505.24298v5)
- 范围：§§3–5、7.2、7.4–7.5，附录 C.3–C.4、E。
- 方法与证据：将 rollout 与训练分离，允许生成在权重刷新时暂停再继续。以样本版本落后约束数据供给。decoupled PPO 把行为策略重要性权重与用于稳定更新的近端比例分开。作者在固定 GPU 数的数学与代码任务中报告最高 2.77 倍，对照包含架构、调度和算法差别。附录给小配置和不同算法的 lag 权衡。
- 工程用法：同一轨迹中先标出每段 token 用到的权重和 logprob，再统计入队到训练的等待、拒收比例、有效样本量和同质量墙钟。给异步候选保留同步强基线。若 GPU 利用率改善却需要更多样本或质量不稳，用目标奖励所需的 GPU-hours 决定是否继续。A3 的组快照和跨版本 token 是两种数据合同。

### `arxiv:2510.11370`

- 原文：[Routing Replay for MoE RL v3](https://arxiv.org/html/2510.11370v3)
- 范围：§§3–5、表 4、附录 E，核对专家选择、logprob 残差和计时分母。
- 方法与证据：rollout 保存每层每 token 的 top-k expert ID。trainer 用原 ID 路由，并用训练侧 logits 算门控权重，保留梯度。Qwen3-30B-A3B 指定对照的估计 KL 约从 0.0015 到 0.00075。表 4 的 42,321→40,910 token/s 对应 3.33% 吞吐下降，同量 token 时间约增加 3.45%。10 步实际墙钟从 4,820 s 到 4,967 s。
- 工程用法：同 checkpoint 若 logprob 仍不同，先比较 token 串、有效 mask、各层 expert ID、gate 权重与 kernel 数值。先量化分歧最早出现的层，再决定锁路由、切对照后端还是改精度。若锁路由后残差仍来自分词或上游隐藏状态，继续查该层，避免调 PPO 超参数来覆盖观测错误。

### `arxiv:2602.00482`

- 原文：[AReaL-DTA v2](https://arxiv.org/html/2602.00482v2)
- 范围：§§3–5.3、附录 B.1–B.2，分开读取单卡计算、固定卡比与重分配整条流水线。
- 方法与证据：共享前缀形成树。训练沿深度优先次序处理分支，保留当前路径所需状态，对部分分块重算。作者的 8.31 倍是相对 Dense+CKPT 的单卡梯度消融，未含 optimizer；相对 Sparse+CKPT 为 1.52–1.70 倍。8B 端到端 2.28 倍同时把 rollout:trainer 从 2:6 改为 5:3，固定 2:6 为 22.6→27.5 K tokens/s。弱共享分布存在变慢项。
- 工程用法：先从轨迹算有效共享 token、分支数和最长活跃路径。用逐轨迹实现校验每段 logprob、loss、梯度和 optimizer 更新，再固定卡比比整步，最后开放重新分配。低共享率时同时记录索引构建、mask、同步和重算时间。可用共享率分桶决定启用条件。

### `https://www.snowflake.com/en/blog/engineering/zorro-enterprise-rl-training/`

- 原文：[Snowflake 工程报告 · ZoRRo](https://www.snowflake.com/en/blog/engineering/zorro-enterprise-rl-training/)
- 范围：训练和推理复用机制、图 1–3、脚注 1。对照作者的 workload、阶段和基线。
- 方法与证据：相同 prompt 的多个响应共享训练前缀，避免重复算同一上下文。约 6 倍衡量 Arctic-Text2SQL-R2 的 actor update，最大 prompt 16K、响应 4K、每题 16 个 rollout。32 H200 上约 3.5 倍衡量整步加一组优化后的对照。图 1 的折算 FLOP 含基线中已被省掉的重复工作。
- 工程用法：在多假设 profiler 或工程检索的长上下文里先测共享前缀比例和实际执行量。把推理缓存、训练共享及资源重分配做成三项干预。若瓶颈只占训练小段，或输入多样性让共享率低，应按整步回报降低该候选的优先度。

### `https://huggingface.co/papers/2602.03839`

- 原文：[Hugging Face 论文入口](https://huggingface.co/papers/2602.03839) · [作者正文 PULSE v2](https://arxiv.org/html/2602.03839v2)
- 范围：入口与作者版本，正文 §§3–5、附录 F.3、J。核对 weight patch 与伪梯度通信。
- 方法与证据：PULSESync 比较连续 checkpoint 在 BF16 可见表示下的变化并发送覆盖值，接收端校验 hash。作者 7B 实验平均 108 MB，对照完整 BF16 的 14 GB。PULSELoCo 在 trainer 间传输带误差反馈的稀疏 FP32 伪梯度，指定设置从 30.5 到 1.77 GB，早期若干 checkpoint 改善较慢。两项的传输对象、频率和质量约束各异。
- 工程用法：把线上的等待拆为编码、发送、加载、同步和恢复，先确认链路在关键路径。补丁做相同版本、重复到达、跳版本和全量重建对照。若同步已经被覆盖，压缩字节未必节省墙钟。训练更新的可接受误差应由同质量对照决定。

### `https://www.lmsys.org/blog/2026-05-13-no-token-left-behind/`

- 原文：[LMSYS · No Token Left Behind](https://www.lmsys.org/blog/2026-05-13-no-token-left-behind/)
- 范围：三类 round-trip 失败、inference session server、增量模板渲染、训练样本抽取与验证。
- 方法与限制：session 保留原始 token 序列，下一轮只追加增量并核对前缀。分词合并、模板裁剪早期 reasoning、解析后重排 JSON 都可改变 token IDs。文本相同只能说明可见表面接近。不同模型需要适配模板和角色边界。
- 工程用法：为 A3 轨迹保留 sampled token IDs、logprobs、动作区间、版本和环境观测边界。先用多轮带工具结果的最小 fixture 比较 rollout 与 trainer 的逐 token 位置。若第一处分歧出在历史工具消息，修复记录和渲染后再解释算法的 KL 或奖励变化。

### `https://cursor.com/blog/tab-rl`

- 原文：[Cursor · Improving Cursor Tab with online RL](https://cursor.com/blog/tab-rl)
- 范围：建议展示阈值的例子、reward、on-policy 采样周期和前代模型对比。
- 方法与证据：保留可选择不展示的动作。示例目标接受概率为 25%，奖励分别是接受 +0.75、拒绝 −0.25、略过 0。文中报告上线新 checkpoint 至下一批数据为 1.5–2 小时，新模型少显示 21%，显示后的接受率相对高 28%。这两个指标不直接衡量正确性、开发效率或遗漏的有效建议。
- 工程用法：在性能助手评估中让“信息不足，先补测”和“沿用当前方案”成为可评分动作。分别统计候选覆盖率、有用建议的总量、错误打断、正确性和人力时间。接受率提高但错过根因较多时，应调候选门槛和反馈设计，再考虑在线更新。

### `https://github.com/areal-project/AReaL/blob/main/docs/paper/AReaL2.0_report.pdf`

- 原文：[AReaL2.0 登记入口](https://github.com/areal-project/AReaL/blob/main/docs/paper/AReaL2.0_report.pdf) · [同一官方仓库 PDF](https://raw.githubusercontent.com/areal-project/AReaL/main/docs/paper/AReaL2.0_report.pdf)
- 范围：2026-11-13，§§3–6、图 1、Hermes 示例和实现范围。PDF SHA-256 为 `99f90d5b79060d098718a9783e368fbc48a1971b115eb14d1315cf6395dda418`。
- 方法与限制：作者提出带可见观测、模型和工具版本、迟到反馈及数据资格的逐步轨迹协议，配合收集代理和干预控制面。原型以 gateway、router、data proxy 和 worker 接入现有 Hermes 应用，覆盖收集到策略权重更新的支路。图 1 不是所有经验写入、harness 修改和权重更新都已有对照效果的证据。
- 工程用法：从本地已完成的数次运行先提取当时证据、动作、结果和后验核验。一个事件按 ID 可找回可见前缀和后续反馈。先验证记录是否支持同任务回放，之后才能比较直接修规则、检索经验或训练。运行可观测不等于该轨迹足以训练。

### `https://tinker-docs.thinkingmachines.ai/`

- 原文：[Tinker 官方文档](https://tinker-docs.thinkingmachines.ai/) · [Tinker SDK 入门](https://tinker-docs.thinkingmachines.ai/tinker/)
- 范围：根入口 SDK 和 Cookbook 导航，SDK Cheatsheet 的训练数据、采样、Forward-Backward、Optimizer Step 和版本保存契约。查阅日期 2026-11-13。
- 方法与限制：训练客户端负责更新，采样客户端绑定可用 checkpoint。并发 Future 表示请求可并行等待，实际执行顺序仍受版本、依赖和 SDK 契约约束。教程演示 API 用法。
- 工程用法：A3 每一组固定采样来源，并记录 trainer 提交结果和生成 sampler 的版本。对并发修改用可观察的作业 ID 与快照核验。若没有快照归属，先修元数据再比较学习质量。 [A3 实际读到的 API](a3-operations.md)

### `https://tinker-docs.thinkingmachines.ai/tutorials/advanced/sequence-extension/`

- 原文：[Tinker Tutorial 404 · Sequence Extension in Multi-Turn RL](https://tinker-docs.thinkingmachines.ai/tutorials/advanced/sequence-extension/)
- 范围：完整教程的前缀拼接、prompt/action mask、无法延展的拆分和环境终止。404 是教程编号，页面有正文。
- 方法与限制：后续观察精确扩展上一轮观察与模型输出的 token 前缀时可拼成一个 datum，训练 mask 只标动作区间。若 tokenizer、模板或上下文清理改写前缀，库拆成多个 datum。教程用 fake token IDs 示范，实际模板是否满足仍需测。
- 工程用法：用一条含工具观测的训练夹具比逐轮与拼接路径的 action token、logprob、mask 及聚合 loss。分别加入模板重渲染、上下文压缩和保留完全相同前缀的对照。验证 loss 等价后再测前向量与显存。 [Miles TITO 的分词反例](https://www.lmsys.org/blog/2026-05-13-no-token-left-behind/)
