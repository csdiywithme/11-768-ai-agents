# L06 深读 · 从代码模型到完整工程任务

本批逐项核查 L06 的 18 条登记来源。官方录像只核对了入口和少量官方静帧。来自第三方转写的内容单独注明。下列工程应用为待验证的假设。

### `https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf`

- 原文：[L06 Agents for Coding and Software Development](https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf)
- 范围：通读 82 页的文字和图表文字。核对 p.47 原图及 p.12、16–17、27、43、49、54、62–65、69、72、74–79 的定义与选表。缓存两份 PDF 相同，SHA-256 为 `9f764df75b573499d090ed71d19a7b8bbfa01ad7059a440ca1af00111b480b51`。部分纯图页通过引用原文补足。
- 机制：讲次把单步补全、跨文件修复和完整软件开发当作不同任务。正文从数据、分词、填充目标进入定位、编辑、验证，再到合成数据、多 harness、维护和 CI。p.47 将测试交给评估 harness，p.49 指出终点奖励无法定位早期错误。p.74–79 讨论预测代替部分执行的风险和收益。
- 改变的动作：对一次调优分别记录首个错误判断、跨文件追踪、可执行补丁、测量与回归、发布与复核。以一个可复查的算子改动和一个推理服务配置变化比较固定流程、动态流程和预测筛选。预测只有降低真实执行或人工成本且保持有效决策时才保留。
- 推翻条件：若模型选择提高离线标签或代码通过率，但真实环境运行、回归、可审查性或维护时间下降，分阶段定位失败。课件讲授的分数对应各自原论文，不能合并为同口径能力排名。

### `youtube:1BWeH1oOM7k`

- 原文：[L06 官方录像及核对起点](https://www.youtube.com/watch?v=1BWeH1oOM7k&t=3650s)
- 状态：原件受限
- 范围：核对官方播放页缓存和官方 10 秒 storyboard，实看 1:00:50、1:01:00、1:01:10 附近画面。官方英文字幕缓存为空，未取得连续音轨或画面。另读第三方英文转写 26:00–28:08、1:00:59 附近与后段 CI 选段。
- 机制：官方帧逐步显示 fail-to-pass 与 pass-to-pass，对应课件 p.47。第三方转写另提及整套测试的时间代价和先用少量相关测试，及利用 CI 的失败和通过版本。其时间和词句应以原录像复核。
- 改变的动作：先用廉价且可区别原因的测量过滤，提交前再用不同 workload 或更完整的回归检查。实验卡里分别写明目标改进与要保留的行为。
- 推翻条件：若便宜子集漏报真实回归，调整它的适用范围与追加触发条件。视频观察只支持这一段，第三方转写独立保留来源。 [第三方英文转写](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-06/)

### `arxiv:2402.00838`

- 原文：[OLMo: Accelerating the Science of Language Models](https://arxiv.org/html/2402.00838v4)
- 范围：arXiv v4。细读 §2.1–2.4 的模型、Dolma、训练中和离线评价，核对 Table 2 和 §3 的数据与复现说明。
- 机制：Dolma 分来源保留文档，清洗包括语言、质量、内容过滤、去重、配比和分词。作者用每 1,000 步的训练中评估指导架构与数据选择，用固定的离线流程对不同 checkpoint 对照。模型另将嵌入表补齐至 128 的倍数以利吞吐。
- 改变的动作：即便产物是小规模 SFT 或提示样例，也保存来源、任务族、版本和分组。运行中指标可选实验，最终按固定留出集比较。若对算子做宽度对齐，分别测有效计算和实际多算的部分。
- 推翻条件：若训练中偏好的方案在固定留出集消失，复查是否反复用同组样例选配置。若填充在特定形状只增加访存或计算，保留未填充基线。

### `arxiv:2305.06161`

- 原文：[StarCoder: may the source be with you!](https://arxiv.org/html/2305.06161v2)
- 范围：arXiv v2。细读 §§3、4 的相关清洗、去重和治理段落，§§5.1–5.5 的格式、去污染和 tokenizer。核对 Table 10。
- 机制：代码、issue、Jupyter 和 commit 的结构保留方式各异。提交可记录旧代码、消息与新代码，大多数例子只保留修改周围的窗口。论文对代码与 Jupyter 做近重复处理，但没有对 commits 或 issues 做相同处理。tokenizer 使用 byte-level BPE、digit splitter 和控制标记。
- 改变的动作：性能训练样例保留旧实现、工作假设、差异、测试与测量。先去掉同一 PR、fork 和近重复实现之间的训练验证泄漏。比较窄窗口和带调用方的上下文，明确哪些事实需要全文。
- 推翻条件：若仅按文件去重后，基准仍被同一提交或项目的近重复覆盖，改按项目和演进关系划分。旧版本模型的训练规模与数据配比只提供方法线索。

### `arxiv:2107.03374`

- 原文：[Evaluating Large Language Models Trained on Code](https://arxiv.org/abs/2107.03374)，[解析到的 v2 全文](https://arxiv.org/html/2107.03374v2)
- 范围：本条为 L06 无版本链接，解析到前一批读过的 v2。本次新增核对 §3.2 代码词法和空白压缩，避免重复统计同一版的 pass@k 证据。
- 机制：论文以 GPT-3 文本 tokenizer 为基底，再加入不同长度的空白序列 token。作者报告在其代码数据上 token 数量少约 30%。这是分词表示的长度对照。
- 改变的动作：比较分析同一段 Python、Triton、PTX、profiler 日志时，用目标模型的实际 tokenizer 计长度。记录内容字节、token 与首 token 时延。保留 Python 的缩进和字符常量。
- 推翻条件：若新模型或语料的 token 节省很小，或格式归一化使代码与日志丢失含义，保留原始格式。文中比例不能直接当作成本或推理吞吐变化。

### `https://huggingface.co/bigcode/starcoder2-3b/blob/main/tokenizer.json`

- 原文：[StarCoder2-3B tokenizer 文件页](https://huggingface.co/bigcode/starcoder2-3b/blob/main/tokenizer.json)，[同文件原始 JSON](https://huggingface.co/bigcode/starcoder2-3b/raw/main/tokenizer.json)
- 范围：文件页所示上传短提交 `b75eb95`。细读原始 JSON 的 added tokens、ByteLevel、Digits、BPE 词表和部分空白合并。未核查全部 98,409 行。
- 机制：三种 FIM 控制 token 各有独立 ID。预分词先分数字，再做 byte-level 处理。词表把换行与随后三个空格合成一个条目。词表、分词前处理和合并规则共同决定实际切分。
- 改变的动作：保存模型及 tokenizer 版本，再比较 FIM、代码窗口和日志的 token 预算。实验请求记录具体输入 token，而不是用字符数估算。测试含制表符、CRLF、不同缩进和长数字的输入。
- 推翻条件：若版本变更使相同请求的 token IDs 不同，重算预算与前缀命中。词表存在某个条目不能证明任意相近字串都会选中该条目。

### `arxiv:2308.12950`

- 原文：[Code Llama: Open Foundation Models for Code](https://arxiv.org/html/2308.12950v3)
- 范围：arXiv v3。细读 §§2.1–2.4、§3.1 的变体比较、§§3.2–3.3 的 FIM 和长上下文。核对 Table 5 的对照。
- 机制：先从 Llama 2 继续学习代码，再分化基础、Python、指令等版本。FIM 在部分尺寸上加入，长上下文专用训练把序列从 4,096 提到 16,384。FIM 对照中，7B 和 13B 在指定 HumanEval、MBPP 指标的平均差分别为 −0.6、−1.1 个百分点。
- 改变的动作：根据补代码、写评估、跨仓库定位三种任务挑候选模型。锁定权重、FIM 格式和调用方式，对比相同工程输入。较长上下文需要额外验证性能数据是否真的进入有效观察范围。
- 推翻条件：若长上下文增加成本却不能减少重复搜索，减少注入。若领域专用版本只提高 Python 片段补全，继续比较跨 CUDA/C++ 与框架的实测结果。

### `arxiv:2409.12186`

- 原文：[Qwen2.5-Coder Technical Report](https://arxiv.org/html/2409.12186v3)
- 范围：arXiv v3。细读 §§2–3.2、§4.1 的评分与执行检查、§5 和 §6 的相关评价。核对 Table 3。
- 机制：文件层训练后加入 32,768 token 的仓库层样例，再用 YaRN 支持 128K 推理上下文。数据对照中，纯代码到 70/20/10 的代码、文本、数学组合，Common code 从 49.8 到 48.3，MATH 从 10.3 到 33.2，MMLU 从 42.8 到 62.9。课件 p.12 把纯代码行的 GSM8K 数字 23.8 放到了 MMLU 列，笔记以原报告为准。
- 改变的动作：用任务族比较代码语义、性能推理、从文档取证与调用多个工具。为跨文件问题挑相关文件和路径关系。若考虑混合训练，按同 token 预算比较真实任务。
- 推翻条件：若增长来自记忆同源样例，或长上下文只提高检索量，补独立仓库和版本。单表的配比对照无法确定面向 GPU 工程的最佳训练混合。

### `arxiv:2204.05999`

- 原文：[InCoder: A Generative Model for Code Infilling and Synthesis](https://arxiv.org/html/2204.05999v3)
- 范围：arXiv v3。细读 §§2–4.1、§5 训练目标对照，核对 Table 1 和 Figure 2。
- 机制：用哨兵把挖去的片段放到可自回归的位置，让模型在生成时看见代码前后文。HumanEval 派生的单行任务，一次左到右、十候选重排和 causal-masked infilling 的执行通过率分别是 48.2%、54.9%、69.0%。多行是 24.9%、28.2%、38.6%。分数对候选结束策略也敏感。
- 改变的动作：对函数体内的 GPU 改动，同时提供前置数据布局与后续 epilogue 或同步约束。对照补洞、给当前函数的指令编辑和生成小 patch。按目标行为、非目标改动和实际 GPU 验收评分。
- 推翻条件：若后缀使改动更保守却漏掉跨文件或跨函数优化，放开改动范围并扩大回归测试。与参考文本一致只表明形式相似。

### `arxiv:2207.14255`

- 原文：[Efficient Training of Language Models to Fill in the Middle](https://arxiv.org/html/2207.14255v1)
- 范围：arXiv v1。细读 §§2–4.5、§5、§8.1，核对 Figures 4–8 对训练率与打包顺序的对照。
- 机制：将文本前缀、后缀、缺失片段按 PSM 或 SPM 序列训练。50B token 对照里，FIM 比例到 90% 时普通自回归 test loss 无明显退化，100% 时出现退化。更高 FIM 比例的执行通过率改善比 perplexity 差异明显。context-level 转换有助于降低截断后不完整样例的影响。
- 改变的动作：若用训练数据提升局部编辑，保留跨文件分析和普通补全作对照。按实际工具的截断、打包和停止规则测复现。小范围优先跑具行为验收的编辑任务。
- 推翻条件：若训练 loss 下降却不改变正确替换与性能结果，缩减训练投入。检查样例截断、任务族重合与工具格式是否决定了改善。

### `arxiv:2308.07124`

- 原文：[OctoPack: Instruction Tuning Code Large Language Models](https://arxiv.org/html/2308.07124v2)
- 范围：arXiv v2。细读 §§2–4.1、§3 的 HumanEvalPack 与其限制，核对编辑、解释、生成三种任务。
- 机制：CommitPack 以提交消息和修改前后代码形成指令样例。HumanEvalPack 在六种语言中分别评估修复、解释和生成。解释任务先生成解释，再用同模型根据解释重建代码，以执行结果判定。只看重建分数会混合解释和实现能力。
- 改变的动作：构造性能任务时区分「解释 profiler 现象」「执行正确修改」「仅依据解释能否复做」。提交样例补齐 benchmark 口径与最终结果，排除不能从说明推断修改的例子。
- 推翻条件：若解释能促成重建，却不能让另一位工程师选对实验或复核因果关系，改用真实交接任务。若同一提交前后版本泄漏，按仓库和提交族隔离。

### `https://www.together.ai/blog/deepcoder`

- 原文：[Together AI DeepCoder 技术报告](https://www.together.ai/blog/deepcoder)
- 范围：读训练数据、稀疏奖励、GRPO+、响应长度和 LiveCodeBench 结果。核对图和表是不同预算口径。
- 机制：对生成代码执行测试，要求抽取的一组测试全部通过。作者用输入字符串长度选较难样例，最长样本的截断在训练损失中单独处理。报告的同模型评测表在 16K、32K、64K 的 LiveCodeBench 分别为 45.6、57.9、60.6。文章的 54→58 概述涉及不同训练阶段。
- 改变的动作：把超时、编译错、执行错误、测量超限分开登记。若廉价测试只跑一部分，按边界形状和常见错误挑样例，不用单一输入长度替代难度。分别比较推理预算的有效答案与资源开销。
- 推翻条件：若更长预算只减少截断，恢复到同完成条件后再比较。若少数测试让候选规避真实验收，增加独立测试，不用稀疏分数充当过程诊断。

### `arxiv:2412.21139`

- 原文：[Training Software Engineering Agents and Verifiers with SWE-Gym](https://arxiv.org/abs/2412.21139)，[解析到的 v2 全文](https://arxiv.org/html/2412.21139v2)
- 范围：课程无版本链接解析到已另有精读记录的 v2。本轮补读 §§3.1–3.3、§4.3 和 §§5.1–5.2，重点核对环境构造和训练饱和。
- 机制：可执行问题需要预先确认依赖与测试。MoatlessTools 的 32B 在第二轮采样训练后，空 patch 从 18.3% 降到 9.7%，但解决率保持 19.7%。作者讨论受限动作空间和拒绝采样。学习式选择与采样数的效果也受任务、训练数据和工具影响。
- 改变的动作：区分工具提交率、有效完成率和独立确认的性能收益。如果训练使提交格式变好但成功率持平，查看是否缺关键工具或证据。用完整轨迹中的失败类别定位原因。
- 推翻条件：若增加训练数据只降低空提交，先查观察、工具动作和验收质量。无版本链接与 v2 是同一版证据，勿重复计算实验样本。

### `arxiv:2604.12374`

- 原文：[Nemotron 3 Super 技术报告](https://arxiv.org/html/2604.12374v1)
- 范围：arXiv v1。细读 §§2.1.1–2.1.3 的 LatentMoE、MTP 和混合结构，§3.2 的分阶段 SWE-RL，以及 §4 的推理精度。核对 Figure 5 和 Table 2。
- 机制：使用 Mamba 状态、稀疏 MoE 和少量全局注意力层。MTP 能并行草拟多 token。Figure 5 比较 B300、TRT-LLM、TP1 的 NVFP4 checkpoint，MTP off 与深度 1、3，在固定输出长度的 SPEED-Bench 子集上观察总吞吐与用户中位延迟。作者将较慢的软件工程 RL 轨迹单列训练阶段。
- 改变的动作：把草拟接受长度、验证成本、吞吐和用户尾延迟都纳入 speculative decoding 对照。对不同请求长度、并发及内容分别跑。比较多阶段 RL 时计入最长轨迹拖延和批处理开销。
- 推翻条件：若接受长度提升但验证、队列或缓存开销抵消了端到端收益，缩小启用范围。Figure 5 的结果限于指定模型、引擎、硬件及 workload。

### `arxiv:2606.15007`

- 原文：[Nemotron 3 Ultra 技术报告](https://arxiv.org/html/2606.15007v1)
- 范围：arXiv v1。细读 §2.7 稳定性、§3.1.1 的 CUDA 与软件工程数据、§3.2 和 §3.3.2 的多 harness 与专家教师。
- 机制：作者报告约 10 万条 CUDA 生成、修复与优化样例。先过编译和数值验证，再用测得运行时间筛候选。修复轨迹包含错误及改正，优化轨迹包含慢核与 Nsight Compute 日志。预训练一次发散经回退与恢复 FP32 梯度归约缓解，另一次发散的根因仍未确定。
- 改变的动作：如果积累训练样例，把实际硬件、shape、dtype、精度要求和基线，并保存配对结果，记录 profiler 提示过的机制及真实结果。既保存无收益的变体，也保留可移植性边界。先比较小规模人工精选数据对现场工作的影响。
- 推翻条件：若只保留单硬件最快候选造成跨 shape 失效，扩充实测并保留多解。若 trace 缺少工具原始结果或无法复现，用它作参考资料。报告的内部验证环境不能自动当成可复查的本地对照。

### `https://github.com/NVIDIA-NeMo/ProRL-Agent-Server/tree/6a1ead6bfac054fce6c1e62d1a77b330d96c58db/src/polar/agent`

- 原文：[Polar agent 目录固定版本](https://github.com/NVIDIA-NeMo/ProRL-Agent-Server/tree/6a1ead6bfac054fce6c1e62d1a77b330d96c58db/src/polar/agent)，[同提交 README](https://github.com/NVIDIA-NeMo/ProRL-Agent-Server/blob/6a1ead6bfac054fce6c1e62d1a77b330d96c58db/src/polar/agent/README.md)
- 范围：通过 GitHub 读取登记 commit 下的 README、`models.py`、`base.py` 和导出入口。重点核对 launch 模式、任务结束状态和产物收集 hook。未执行或验证所有 preset。
- 机制：harness 提供运行准备与有序命令，外层负责 runtime、代理、记录和评估。配置可用内建 preset、shell 或自定义类。结束记录区分 completed、failed、timeout。定义允许失败或超时后收集已有产物。README 的验证版本来自 calculator 示例。
- 改变的动作：将现有 benchmark 工具通过薄适配器接入，记录 workload、运行身份、阶段结果和产物链接。用同一运行接口测试不同工作流。若考虑把数据用于训练，分别保留实际调用事件和最终标签。
- 推翻条件：如果适配为收集事件而改写原工具语义，复验兼容性。若用户当前只需少量本地运行，比较直接脚本与服务方式的净成本。示例通过不能推断实际 GPU 工作负载兼容。

### `https://vibench.ai/`

- 原文：[ViBench 官方网站](https://vibench.ai/)，[任务定义](https://vibench.ai/tasks)，[方法](https://vibench.ai/method)，[发布者仓库](https://github.com/ViBench/vibench-public)
- 状态：替代版本精读
- 范围：官方站的直接抓取无正文，读同站搜索索引中的项目总览、任务和方法，以及课件 p.65。补读作者仓库 v1 材料定位与 v2 README 的计数、浏览器评估及重试，分别保留版本。
- 机制：课件所引 v1 包含从空项目创建、扩展参考实现、扩展模型旧实现三种场景。作者 v2 报告 17 项 app、241 个功能阶段和 544 份测试计划。模型逐项获得需求，浏览器评估观察实际行为，失败评分可重复核验。不同版本的任务数和成功判定有差别。
- 改变的动作：把从零写小算子、在维护者参考实现上优化、在代理自己的旧改动上继续优化纳入对照。检查连续改动中的回归、重跑和排查成本。由独立证据确认失败，保留两类评分错误。
- 推翻条件：若阶段评分只改善当下功能、不能预测后续修改成本，把维护单独测试。若评分不稳定，固定可重复的检查并保留日志。Vibe coding 任务无法衡量 CUDA 内核的数值和吞吐。

### `https://huggingface.co/datasets/JetBrains-Research/lca-ci-builds-repair/blob/ebf12dad7a97c3c0cdc38705403d8bc8ddde47dc/README.md`

- 原文：[LCA CI Builds Repair 课程指定 README](https://huggingface.co/datasets/JetBrains-Research/lca-ci-builds-repair/blob/ebf12dad7a97c3c0cdc38705403d8bc8ddde47dc/README.md)，[同提交原文件](https://huggingface.co/datasets/JetBrains-Research/lca-ci-builds-repair/resolve/ebf12dad7a97c3c0cdc38705403d8bc8ddde47dc/README.md)
- 范围：使用同提交 resolve 链接取得完整 README，细读配置、数据字段、示例和评估。另查作者论文 §2.2 的基线和评估。未下载任务数据或启动 GitHub workflow。
- 机制：每个样例含失败与通过的 commit SHA、workflow、按 step 的日志和变更信息。该数据集不包含仓库副本。论文把真实 CI 成功作为结果，基线有参考补丁定位提示，并只取首个 error 附近的简短日志。README 同时保存 default 和 old 配置。
- 改变的动作：为 GPU 构建失败保留仓库版本、编译器与驱动、CI 配置、失败步骤和实际重跑结果。比较首条诊断、分步日志和只读搜索。若修复只更新配置，要重跑目标流程和原有功能测试。
- 推翻条件：若构建在原版也偶尔通过，先区分环境波动和代码问题。若看过参考定位才成功，另评无需提示的定位效率。真实 CI 通过不能覆盖数值误差或性能回归。 [Long Code Arena 论文](https://arxiv.org/html/2406.11612)
