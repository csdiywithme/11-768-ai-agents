# 外部原文 · 候选优化、运行状态与历史版本

### `https://engineering.fb.com/2026/04/02/developer-tools/kernelevolve-how-metas-ranking-engineer-agent-optimizes-ai-infrastructure/`

- 原文：[Meta Engineering · KernelEvolve](https://engineering.fb.com/2026/04/02/developer-tools/kernelevolve-how-metas-ranking-engineer-agent-optimizes-ai-infrastructure/)
- 范围：图 1、候选合成、检索、搜索、自动验证、生产与开发结果。
- 方法与证据：运行图安排编译、数值检查、性能计时和按需 profiler。搜索反馈包含占用、带宽等定位信号。成功探索可形成按硬件和任务检索的经验。文章报告特定 MTIA 排序训练吞吐增加 25% 以上，另一个 NVIDIA 推理模型增加 60% 以上。这些是作者案例，无法由文章分离出每种工具的独立贡献。
- 工程用法：B300 一轮先用现有候选和判别式 probe 建人工基线。固定设备、shape、dtype、baseline、容差和计时，再比较仅给运行时间与额外给 profiler 两种生成反馈。记得到第一个可复现有效候选的时间、GPU-hours 和失败分布。若 profiler 不改变下一批候选，减少其开销。

### `https://github.com/ScalingIntelligence/KernelBench`

- 原文：[KernelBench 官方仓库](https://github.com/ScalingIntelligence/KernelBench)
- 范围：README 的数据等级、评估、基准、fast_p、setup 与多轮生成链接。2026-11-13。
- 方法与限制：fast_p 用任务总数作分母，计同时通过正确性且比指定 PyTorch 参考快过阈值的任务。硬件、编译后端、数据与参考实现都影响结果。README 提供运行工具，不能证明通过的实现覆盖工作负载外条件。
- 工程用法：选择实际算子输入时保留小、常见和尾部 shape，固定高质量参考，给编译失败、错误输出和慢候选保留分母。对验收项单独报告匹配时延与适用范围，避免小样本成功后将所有形状放行。

### `https://github.com/OpenAutoCoder/Agentless`

- 原文：[Agentless 官方仓库](https://github.com/OpenAutoCoder/Agentless)
- 范围：README 的 localization/repair/validation、版本对照、可下载的产物与问题分类。
- 方法与限制：按文件、符号、位置收缩问题，生成多个差异，再用回归测试选择。固定阶段仍使用模型。仓库报告分数对应指定模型、框架版本和公开题集。
- 工程用法：框架调优先定义固定的定位→候选→独立验证流程作为强基线。动态流程只在实际反馈要求改顺序时获得额外机会。对照相同检索和 profiler，并按源码范围、最终修复、人工纠正分别计分。

### `https://openai.com/index/harness-engineering/`

- 原文：[OpenAI · Harness engineering](https://openai.com/index/harness-engineering/)
- 范围：仓库可读性、文档、可执行约束、环境、审阅及 drift。
- 方法与证据：团队以短入口链接版本化知识，用可执行规则、浏览器或可观察服务行为给 Agent 反馈。已存模式会被重复，需持续收敛重复或过时做法。作者给开发速度估计，但单一团队项目缺匹配的人工实验。
- 工程用法：从 vLLM、CS336、DeepInfer 的真实命令抽出最小可复现入口与事实定位，再比较简单 README、检查脚本与可分支 harness。测首次找到真实失败位置、重入以及人工修正。少量稳定契约优先机械检查，项目限制仍从原 repo 读取。

### `https://www.anthropic.com/research/effective-context-engineering-for-ai-agents`

- 原文：[Anthropic · Effective context engineering](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)
- 范围：上下文预算、工具设计、just-in-time retrieval、compaction、notes 和 subagents。
- 方法与限制：保留轻量句柄并按需打开原材料，长时任务可用压缩和笔记，独立分支可隔离上下文。文章的做法和案例取决于任务如何恢复证据以及分支是否独立。
- 工程用法：对同一 GPU 回放前缀比较完整日志、原件索引、压缩摘要和固定状态字段。用执行前状态恢复、下次 profiler 选择、重复操作和总 token 衡量。只在更少上下文维持这些行为时采用更激进裁剪。

### `https://proceedings.mlr.press/v15/ross11a/ross11a.pdf`

- 原文：[DAgger · PMLR 2011 正式 PDF](https://proceedings.mlr.press/v15/ross11a/ross11a.pdf)
- 范围：§§2–4、Algorithm 3.1、§5 两个模仿任务和序列标注。
- 方法与证据：从当前策略及可混入专家的策略访问状态，由专家为这些状态标动作，聚合后再训练。在给定损失、策略集合和 no-regret 条件下约束诱导分布上的误差。真专家、可访问状态和采样成本是前提。
- 工程用法：收集在错误地选过 profiler、工具失败或看到反例之后的真实前缀，由工程师标哪些动作可继续区分。保留能合理绕过失败的多种动作，避免只模仿成功长轨迹。把该数据与普通示范用同预算比较。若没有可复查状态或标签分歧太大，先修记录。

### `arxiv:2608.06503v2`

- 原文：[Towards Reliable Context Compression v2](https://arxiv.org/html/2608.06503v2)
- 范围：§§3.1–3.3、4.1–4.2、5.2–5.4 和附录 A，核对 matched boundaries。
- 方法与证据：固定 Agent 和环境，从同状态比较压缩后的继续执行。AppWorld 的 test-normal 中无压缩平均 85.7，Prompting-O 71.4，优化后的 TRACE 77.1。两次都成功为 77.4、59.5、67.3。中等预算下简单 FIFO 有时与摘要接近，重压缩时结果分开。
- 工程用法：在收到可否定假设的新 trace 后构造压缩边界，重放同状态。比较短摘要、原始近期回执与持久 run 状态。检查终止识别、重复查旧证据、重复 GPU 提交和适时换下一实验。结果按独立任务和重跑次数分别计。

### `arxiv:2608.06503`

- 原文：[无版本登记入口](https://arxiv.org/abs/2608.06503) · [实际阅读 v2](https://arxiv.org/html/2608.06503v2)
- 范围：同一 v2，另检查 §3.2 的终止例和附录 A 的冻结条件。
- 方法与限制：同一终止决策处，2K 摘要、同预算 FIFO 与完整前缀的正确停止率不同。摘要可使 Agent 不知道任务已经完成，即使原始状态可重新查询。
- 工程用法：验证器加入已经生成全部文件、等待远端运行和确实需重试三个几乎同形的状态。只换上下文表示，检验恢复是否依据真实任务阶段。与 v2 是同一份证据。

### `arxiv:2401.09670v2`

- 原文：[DistServe v2](https://arxiv.org/html/2401.09670v2)
- 范围：§§3.1–3.3、4.2–4.3、6.2–6.4、表 2，对照本研究已读的 v3。
- 方法与证据：低跨节点带宽的放置要求同 stage 在一台机器用 NVLink 传 KV。作者在该对照上报告 95% 以上传输小于 30 ms。另比真实与模拟 SLO 达成率。数据来自其模型与旧对照，不能作为新架构带宽假定。
- 工程用法：先测实际拓扑与 KV 传输，再计算 prefill queue、prefill、传输、decode queue、decode 五段。模拟须用至少两档实际 load 校准。若路由分布、context 或网络更改，再决定用分离或共置。

### `arxiv:2505.24298v1`

- 原文：[AReaL v1](https://arxiv.org/html/2505.24298v1)
- 范围：§§4–5、7.2–7.5、表 1 和附录 D，与 v5 的指标范围对照。
- 方法与证据：v1 的 2.57 倍指对照 VeRL 吞吐的最高项。表 1 还分别给指定训练步、Synchronous AReaL 与 AReaL 的时间，以及部分根据较新实现吞吐估计的 VeRL 时间。计时与最高可达到的质量需要同时看。
- 工程用法：内部 benchmark 报并列原始速度、同质量时间和计算估计来源。使用相同 seed、配置和硬件复测，不混写未实跑的基线。版本变化时先回读本次要比较的表。

### `arxiv:2602.00482v1`

- 原文：[AReaL-DTA v1](https://arxiv.org/html/2602.00482v1)
- 范围：§§3–5、表 1、图 2–3，与 v2 的算法标注和消融对照。
- 方法与证据：v1 §5.1 写 PPO，v2 对应段写 GRPO。v1 端到端设置把 rollout 与 trainer 配比由 2:6 改为 5:3。DFS 拆分、负载均衡和显存重算均影响结果。
- 工程用法：验证共享优化须先用相同 rollout 算法、卡比和数据，再单独改变资源。写清哪个数据是单 GPU 计算，哪个是训练 step，哪个是同等质量。复现前将 paper 版本加入实验合同。

### `arxiv:2602.03839v1`

- 原文：[PULSE v1](https://arxiv.org/html/2602.03839v1)
- 范围：§§3.2–3.4、4.1–4.3、5、图 6，与 v2 区分。
- 方法与证据：v1 的 sparse value patch 用实际 BF16 值覆盖连续 checkpoint 中变化的项。低学习率的特定 7B 部署中 108 MB 对照 14 GB，传输通过 SHA-256 校验。v2 的 PULSELoCo 训练侧伪梯度是不同操作。
- 工程用法：按可见精度测当前优化步及不同同步间隔的有效变化。编码、传输、解码和丢版本恢复分开测。若带宽没占关键路径，继续调查 rollout 等待，不把压缩倍率写成训练速度。

### `arxiv:2411.15114v1`

- 原文：[RE-Bench v1](https://arxiv.org/html/2411.15114v1)
- 范围：§§3–5、Fig. 2、核优化任务示例，检查时间预算和 best-of-k。
- 方法与证据：七个手工研发环境，61 位专家留下 71 次尝试。在作者 2 小时对照，代理得分高于专家组；8 小时时专家超过所测代理。代理提交更频繁但大部分候选没有超过参考。长预算部分通过独立短尝试的 best-of-k 表达。
- 工程用法：记录第一个合格候选、最终候选以及 GPU/人力时间。区分一个可重规划的长轨迹与并行短轨迹。按时间画收益，不用单次冠军对专家平均。结合正负任务看何时应接管。

### `arxiv:2502.13138v1`

- 原文：[AIDE v1](https://arxiv.org/html/2502.13138v1)
- 范围：§§3–4 的解空间、扩展策略和 MLE-Bench/RE-Bench 结果，图 4 与局限。
- 方法与证据：将候选程序作为节点，运行和评分写入边，生成新起点、修错或从有效父节点改进。贪心倾向可快速增加试验，但也可在难题早收敛。RE-Bench 图来自被引用实验，不能当独立第二次复现。
- 工程用法：对相同 CUDA 核给固定编译和 bench 预算，比较贪心优化、周期性换机制与人工。按共享父节点划分数据，确认看似多样的候选是否只是语法变化。若最佳得分停滞而瓶颈假设未换，单独测试一个不同瓶颈解释。

### `arxiv:2503.20783v1`

- 原文：[Understanding R1-Zero-Like Training v1](https://arxiv.org/html/2503.20783v1)
- 范围：§§2、3.1–3.3、Fig. 6、附录 A，核对长度、组标准差、模板与数学对照。
- 方法与证据：作者观察 base prompt 改变初始探索；从 GRPO 移除按响应长度和组标准差的除法以减小两类偏差。指定 Qwen2.5 数学实验中错误答复长度较受控。论文区分公式和实际 loss reduction 的影响。
- 工程用法：如果长工具轨迹与成功相关，先看长度来自更多有效动作、重复读取、环境等待还是 loss 加权。固定长度分布跑 mask 与 loss 检查，再按任务难度比较质量、生成 token 和 GPU 时间。不要把更长回答本身记为进步。

### `arxiv:2510.24702v1`

- 原文：[Agent Data Protocol v1](https://arxiv.org/html/2510.24702v1)
- 范围：§§3–6，数据转换、保留字段、实验和适配成本，§7 的未覆盖内容。
- 方法与证据：中间格式让每个数据源先转换一次，再按 harness 转到 SFT。作者估算接口数由 D×A 变为 D+A，实际成本仍含各数据集语义映射、清理和校验。该版本主要覆盖文本。
- 工程用法：先把两份真实工程轨迹转成含决策输入、工具 ID、状态与回执的共同格式。再渲染到两个 harness 比较生成 token、工具名和标签对齐。若关键 profiler 信息只能以无结构文本保留，先补评估字段和原件引用再扩大数据。

### `arxiv:2602.05892v1`

- 原文：[ContextBench v1](https://arxiv.org/html/2602.05892v1)
- 范围：§§2.1–2.4、3.1–3.5 与表 2。核对文件、块、行级与最终解决率。
- 方法与证据：评估对真实改动相关的参考上下文，分层记录精度和召回。相同模型下各 scaffold 的检索排名与 issue 解决排名不同。参考位置本身不覆盖全部可行解。
- 工程用法：为 vLLM 调度例标注可替换的充分证据组合，例如 scheduler 配额与 runner 回执，或等价的定向回放。分别判断定位、读到关键证据、诊断和候选验收。如果参考行提高但选择动作不变，直接测筛选或判断。

### `arxiv:2603.01203v1`

- 原文：[How Well Does Agent Development Reflect Real-World Work? v1](https://arxiv.org/html/2603.01203v1)
- 范围：§§2–4、附录 A，O*NET 活动、职业与 BLS 分母。
- 方法与限制：将 Agent 论文和实验按职业与活动覆盖映射，再用就业和工资统计比较研究覆盖。分类、发表样本、地区和统计权重会影响解释。可定位研究空白，不能推出单个岗位或任务的收益。
- 工程用法：以实际每周工作记录按选题、定位、测量、调优、验收和交付分层。没有覆盖到的高人工成本任务应新增回放。先看真实时间和工作频次，再判断公开基准是否代表需求。

### `arxiv:2606.09833v1`

- 原文：[CollabSkill v1](https://arxiv.org/html/2606.09833v1)
- 范围：§§3.1–3.3、4 与附录 G，核对评分工具、用户动作及人工复查。
- 方法与证据：用户实际协作后提交产物。按 rubric 给两个独立模型评分，人工核对 27 个 GDPval 任务的 rubric 与 135 个任务/产物评分样本。后者作者报告 Pearson r=0.72。相关并不保证个别输出或性能实验的排序正确。
- 工程用法：比较不同协作策略时保留用户主动输入、检查和返工时间，再由固定的运行验证器检查实际产物。对审阅信息不足的结果记录可复查原件。人评和自动 rubric 各按自己的误差范围使用。

### `arxiv:2607.05147v1`

- 原文：[DSpark v1](https://arxiv.org/html/2607.05147v1)
- 范围：§§3–4、5.2–5.4，位置接受率、target verify 选择与线上 MTP 对照。
- 方法与证据：混合并行及半自回归草稿，按置信度调验证长度。匹配总吞吐的作者实测中，V4-Flash 每用户生成速度提高 60%–85%。极端 SLA 下历史基线逼近容量下限。生产方案同时改变草稿和调度。
- 工程用法：固定 target 逐项比较 drafter、draft length 和 verify policy。记录有效输出、实际 verify 长度、草稿开销、队列及 SLA goodput。若只缩短 verify，需检查固定草稿成本是否抵消收益。

### `arxiv:2609.21058v1`

- 原文：[How Much of a Real Workload Can LLM-Generated GPU Kernels Actually Reach? v1](https://arxiv.org/html/2609.21058v1)
- 范围：§§2–6、真实模型可触及部分和局限，核对分母及独立计时。
- 方法与证据：作者从 60 个单算子任务排除 4 个正确性校验无信息的题。指定 A100/bf16 设置下，一个模型生成 22/56 个独立复核的快实现，中位加速 1.235 倍。比较取多个基线中最快。相对误差、写覆盖、算子不变量和隔离计时分别核验。
- 工程用法：候选生成前先估目标工作负载里可替换且位于关键路径的份额。验收必须有 matched baseline、完整输出与真实 shape。若一组模型更擅长不占热路径的题，则不按题集胜率决定 GPU 搜索投入。

### `arxiv:2610.17471v1`

- 原文：[Scaling Inference-Time Compute with Teams of Coding Agents v1](https://arxiv.org/html/2610.17471v1)
- 范围：§2、§5、附录 C.2–C.5 与 Table 3，分工、消息、模型和预算。
- 方法与证据：把团队设计看成同预算优化，分别委派计划、实现和验证。ProgramBench 某些 Sol 设置只在规划阶段委派即可接近全阶段平均结果。开放消息后各模型表现不同，单题波动较大。
- 工程用法：先找可独立验证的候选生成工作。比较单代理、并行独立搜索、只委派规划和共享状态团队，同样算模型与 GPU 总预算。若分支只重复读取或引入冲突，把交接合同简化再测。

### `arxiv:2610.26827v1`

- 原文：[KEEP-Bench v1](https://arxiv.org/html/2610.26827v1)
- 范围：§§3.2–3.3、4.1、附录 A，平行、条件和少数证据。
- 方法与限制：构造每条证据都必要的样本，让摘要单独接受证据保留判定。关键句虽出现少也应保留。文本任务与人为构造的证据条件不能直接代表长程工程动作。
- 工程用法：在 NCU 与 DeepInfer 记录置入“仅预打包”“真实 workload 不同”“未观察到 overlap”等会翻转动作的少数证据。测试摘要有没有保留这些条件，并将选错候选与遗漏句子对应。再补真实日志上抽查。

### `arxiv:2610.41177v1`

- 原文：[TraceSpec v1](https://arxiv.org/html/2610.41177v1)
- 范围：§§3.1–3.2.4、后续 adaptation 实验，核对 replay 输入和接受量定义。
- 方法与证据：对相同已保存决策输入比较 drafters。Qwen3-8B、T=0 下 DSpark 从 <2K 到 16–32K，Web research 5.242→3.442，软件工程 3.073→4.321 tokens/round。此口径可含 correction/bonus，不能直接用作 accepted draft fraction。
- 工程用法：对 DeepInfer 先固定模型、输入内容、状态和温度，再按任务阶段与长度分桶。分别报每次 proposed、accepted、extra、总输出及墙钟。若接受趋势相反，选择分任务阈值，并保留同等请求到达下的服务对照。
