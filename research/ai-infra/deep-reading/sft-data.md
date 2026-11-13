# L8 训练数据、工具动作和模板

核验日期 2026-11-13。5 项论文及报告、1 项会议论文、4 项官方文档。四篇官方文档按核验当日页面记录，以对应安装版本的实际渲染和张量为准。

### `arxiv:2505.20192`

- 原文：[BalanceSFT，arXiv v3 正文](https://arxiv.org/html/2505.20192v3)
- 范围：§§3.1–3.3、4.1–4.5、5 和附录 D。核对表 1、2、5。该 ID 的摘要入口显示 FunReason，课程标签和 v3 HTML 正文标题为 BalanceSFT，本条以 v3 正文方法为准。
- 机制与证据：训练数据的推理段平均 350.74 token，调用结果平均 31.07。按总 token 平均时，前者影响更大。SSB 先按段平均，再用可学权重组合。HDR 将格式基本合格但执行答案错误的样本拿来重新生成，按质量评估筛选再训练。表 5 的 Qwen2.5-Coder 对照中，BFCL 多轮准确率为普通 SFT 38.25%，仅 SSB 41.62%，仅 HDR 43.12%，两者 47.00%。Llama-3.2-3B 对应 33.25% 到 36.12%，收益幅度不同。作者指出对推理数据质量和 LLM 评判的依赖。
- 工程动作与反证：分别评价短工具调用中的 shape、dtype、GPU ID、trace 路径、参数缺失时的询问，以及后续动作能否通过实测。对同一初始模型做段权重与错误样本重采样的四格实验，保持训练量和测试接口一致。若格式准确率上升而真实参数、执行或诊断变差，检查评判筛选或把训练目标移到能执行验证的字段。

### `arxiv:2405.14394`

- 原文：[Instruction Tuning With Loss Over Instructions，arXiv v2](https://arxiv.org/html/2405.14394v2)
- 范围：§§3、4.1–4.4、5。核对式 1–4、表 1、图 2–3 和限制。
- 机制与证据：Instruction Modelling 将自然语言指令与回答一起用于 next-token loss，排除模板控制 token。作者在部分小样本或长指令配短输出的设置中观察到提升。图 3 只按回答计算损失，LIMA 训练损失均值由常规方法 1.37 变为 1.45，另一份 Tulu V2 样本的测试损失从 1.32 到 1.17。该比较支持作者关于过拟合的解释，评测收益依数据而变，表 1 的 LIMA 并非每个指标都提高。
- 工程动作与反证：若手头只有少量长 profiler 背景配短结构化决策，比较仅回答和经过筛选的指令加回答。固定任务和初始 checkpoint，在独立仓库、别的 GPU 及不同描述措辞测试工具动作和正确性。若模型开始复述日志、降低工具执行率或在换项目后失去收益，缩小可监督的输入内容。

### `arxiv:2507.07817`

- 原文：[On the Effect of Instruction Tuning Loss on Generalization，arXiv v2](https://arxiv.org/html/2507.07817v2)
- 范围：§§2–5.2。核对式 2、表 1–2、图 2 和图 4。
- 机制与证据：WIT 给 prompt 与 response 的 token 各用独立权重，分母为非零权重的 token 数。改变权重还会改变有效损失尺度。作者在 5 个 1B–8B 模型、3 种训练数据、5 个通用评测上搜索权重，报告各配置选优后比仅回答 loss 平均相对增加约 6.55%。最优点随模型、数据和指标变化。约 81% 的细分比较中最优 prompt 权重在 0 至 0.6，这个范围包含 0。评测组合含客观题和 LLM 判分，结论应在新任务上复核。
- 工程动作与反证：固定样本和训练预算，验证仅 assistant loss、少量 prompt 权重、全序列三组。记录有效监督 token、损失归一化、学习率与计算量。验证集选参数，单独用保留项目测试提示改写、执行成功、下一实验是否有效。如果优势只出现在验证集，或按真实回放选择的指标变差，维持原损失。

### `https://proceedings.mlr.press/v15/ross11a.html`

- 原文：[DAgger，AISTATS 2011 官方 PDF](https://proceedings.mlr.press/v15/ross11a/ross11a.pdf)
- 范围：印刷页 627–635，重点 §§2–5。核对算法 3.1、定理 4.1、图 2、4、5。
- 机制与证据：用当前策略产生实际会遇到的状态，让专家给这些状态标注动作，再合并此前数据。早期可混合专家策略。理论保证依赖替代损失、在线学习后悔界和专家等条件。作者在 Super Tux Kart 单赛道的设定下报告第 15 轮获得零掉落策略。Mario 例子显示只从专家示范学到的模型会卡在专家本来就避开的障碍旁，迭代采样能补这种恢复状态。
- 工程动作与反证：在离线环境让现有助手提出诊断动作，挑选错误动作随后实际出现的状态，例如编译失败、抓错 kernel 或 GPU 时间不足。请工程师根据当时可见信息标下一步，保留原错误和恢复过程供验收。与同等标注预算的正常示范比较首次恢复、重复错误、无效实验和整单完成率。如果标注员需凭不可见答案才能纠错，补观察接口或停止该状态的标签。

### `arxiv:2510.24702`

- 原文：[Agent Data Protocol，arXiv v2](https://arxiv.org/html/2510.24702v2)
- 范围：§§3–6、附录 B、C、E。核对动作、观察、三步转换、数据混合和跨 harness 结果。
- 机制与证据：ADP 用任务信息、动作和观察记录轨迹。API 动作、代码动作、用户消息与环境观察有明确类型。流程先转公共格式，再转各 harness 训练格式，末端验证对话和工具调用结构。作者统一 13 个数据集。在表 3 的同一 SWE-Bench Verified 评测，OpenHands 配 Qwen2.5-Coder-7B-Instruct 的底座为 2.8%，SWE-Gym 数据为 10.6%，ADP 混合数据为 20.4%。数值比较不同训练数据与转换流程，无法归因给格式这一项。附录 E 在 OpenHands 与 Qwen3-8B 中按约 3 万条样本对照，SWE-smith 上采样为 11.0%，ADP 为 16.6%。该对照匹配样本数，未给出相同监督 token 数。
- 工程动作与反证：如 profiler 工单、repo patch、serving 轨迹要供多个 harness 读取，记录同一任务定义、可重放的工具动作、原观察引用、环境版本和验收判定。用两种导出检查工具参数、单位、退出状态、时间窗和产物路径往返保留。若第二种 harness 仍需为每类输入另写大量例外，或丢失证据语义，先收窄公共格式再扩数据源。

### `arxiv:2507.20534`

- 原文：[Kimi K2，arXiv v2](https://arxiv.org/html/2507.20534v2)
- 范围：§§3.1–3.1.1、3.2.1、5、附录 B。重点核对图 8、合成工具、模拟回执、真实执行、模型限制及工具 token 模板。
- 机制与证据：作者组合 3,000 多个真实 MCP 工具定义和超过 20,000 个合成定义，按工具配置生成任务与判定标准，用状态化模拟器生成多轮交互、部分失败和异常。对代码任务另用真实沙箱和测试提供可执行证据。报告把 SFT、RL 和多种训练数据用于同一模型，§3.1.1 的描述未给 GPU 调优上的合成比例对照。§5 说明困难任务和模糊工具定义可能造成输出过长和调用截断。
- 工程动作与反证：使用真实的只读 profiler 接口与历史报告做一组回放。合成无匹配 kernel、过期工具版本、计数器缺失或超时等分支。算子性能标签来自匹配基线和实测，同一硬件与模型下保留非改善结果。固定训练量比较真实轨迹、纯模拟和混合轨迹在保留仓库上的工具及最终判定。若模拟训练只提高脚本顺序吻合率或掩盖真实工具失败，减少模拟样本或修正其行为。

### `https://huggingface.co/docs/transformers/en/chat_templating`

- 原文：[Transformers Chat templates 官方文档](https://huggingface.co/docs/transformers/en/chat_templating)
- 范围：角色控制 token、`apply_chat_template`、`add_generation_prompt`、`continue_final_message`、Model training 的示例。已核对文档示例，未本地运行模型。
- 机制与条件：对话角色列表需经模型对应模板转换。生成下一条 assistant 消息和续写末条消息使用不同参数，这两个参数不能同时用。文档训练示例关闭 `add_generation_prompt`。若先渲染文本再 tokenize，应检查第二步是否重复添加 BOS/EOS。
- 工程动作与反证：固定模型与 tokenizer revision，对同一轨迹保存训练文本、token IDs 和实际推理边界。检查截断、工具调用前缀和结束 token。若角色错位或 BOS/EOS 重复，先修渲染路径，再比较模型能力。

### `https://huggingface.co/docs/transformers/en/chat_templating_writing`

- 原文：[Transformers Writing a chat template 官方文档](https://huggingface.co/docs/transformers/en/chat_templating_writing)
- 范围：模板空白控制、存取顺序、跨 Jinja 实现注意点、工具定义、调用、响应和官方示例。已核对文档示例，未本地运行模型。
- 机制与条件：模板用 Jinja 把工具定义、assistant 的 `tool_calls` 列表、`tool` 消息分别渲染成模型要求的表示。工具支持多个调用与否由模型接口决定。空白控制、JSON 序列化和加载优先级都会改变模型收到的 token。
- 工程动作与反证：为同一个 profiler API 保存无调用、单次、并行、部分失败和工具返回的模板样本。比较训练前处理、Python 推理与 serving 渲染的实际 token 和 JSON 参数，检查带空格的路径及空列表。若任一端二次编码参数或错配调用回执，先修复接口并重跑对照。

### `https://huggingface.co/docs/trl/en/sft_trainer`

- 原文：[TRL SFT Trainer 官方文档](https://huggingface.co/docs/trl/en/sft_trainer)
- 范围：数据格式、Packing、assistant-only loss、completion-only loss、EOS 对齐和 Tool Calling with SFT。已核对配置示例，未实际启动训练。
- 机制与条件：prompt-completion 数据默认只监督 completion。多轮对话可设置 `assistant_only_loss`，需要模板标记能产出 assistant mask。工具调用样本包括工具 schema、调用、回执。示例中训练 EOS 须与模板对齐，packing 将多条样本放到一个输入序列。
- 工程动作与反证：查看 dataloader 真实 batch 的 `input_ids`、labels 与掩码。确认回答、调用和结束标记覆盖符合实验假设，工具返回、环境观察和 padding 按实验处理。长日志还要确认没有截掉最后动作。比较 packing 时分别记录有效监督 token/s 和任务正确性。若 loss 降低但主要监督了回执或重复了 EOS，修复数据流水线后重建基线。

### `https://huggingface.co/docs/trl/en/chat_templates`

- 原文：[TRL Chat Templates 官方文档](https://huggingface.co/docs/trl/en/chat_templates)
- 范围：支持的模型族、训练模板自动选择、DeepSeek-V3、Qwen3/Qwen3.8 的模板差异和 helper。已核对文档示例，未实际运行模板。
- 机制与条件：SFT assistant-only 的生成标记决定哪些 token 被监督。工具交互 GRPO 另要求追加 tool 消息时已生成部分保持原渲染。文档中的 DeepSeek-V3 补丁用 `tojson` 处理字典参数，Qwen3 相关补丁可保留之前的思考段以满足前缀稳定。已识别的基础模板才触发相应替换。
- 工程动作与反证：取一份已知任务轨迹，分别比较加回执前后的公共前缀、assistant mask 和 EOS。将训练模板与推理 tokenizer 模板作为两个可复核版本保存。若追加回执重写前缀，或 assistant mask 漏掉调用结束标记，先修当前模板，再解释损失和训练结果。
