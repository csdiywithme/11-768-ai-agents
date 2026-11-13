# L07、L08 深读 · 工具观测和训练数据

本批核查十四条课程登记。来源包括 L08 课件、录像静帧与训练报告，也包括 L07 的环境评估。所列 AI Infra 应用是待验证的假设。

### `https://www.cmu-agents.com/slides/lecture-08-sft.pdf`

- 原文：[L08 Supervised Fine-Tuning for Agents](https://www.cmu-agents.com/slides/lecture-08-sft.pdf)
- 范围：通读 49 页文字与表格文字。重点为 pp.11–18、21–29、31–45、47 和附页。缓存 PDF 的 SHA-256 为 `1598a8bd9784ec044f89b8951af52bb2f80c0236b73aada0ca4cc3038049c4e3`。个别图的数值由引用的原文复核。
- 机制：训练数据包含用户任务、助手动作、工具回传和最终提交。训练用目标 token 和只作输入的 token 由模板及 mask 区分。课件比较按样例数与 token 数混合、保持成功与保留优良过程、轨迹标准化与面向目标 harness 的渲染。多来源、跨工具和离线监督还需要在真实执行中检查。
- 改变的动作：先存原始实验事件和执行结果。渲染给某模型训练时检验 tool result 是否只进入上下文、停止 token 是否符合真实生成、空白是否改变边界。训练前后测同一批独立工单，并保存最小工作流基线。
- 推翻条件：若少量样本都无法拟合，先检查格式、标签和终止。若训练指标改善但下一项测量、错误恢复或跨框架复用无变化，重新检查数据选择。大模型报告中的批量和长度只描述其训练条件。

### `youtube:O3HSU0AoILc`

- 原文：[L08 官方录像及静帧核对起点](https://www.youtube.com/watch?v=O3HSU0AoILc&t=1640s)
- 状态：原件受限
- 范围：实看官方 storyboard 27:20、27:30 附近的两页切换，官方字幕缓存为空。另读第三方英文转写 27:26.159、28:47.840 附近。未取得连续画面和音轨。
- 机制：画面从成功轨迹中的坏习惯切换到按轨迹形状过滤。第三方转写指出，小于五轮可能是清楚的一次成功，也可能是异常提前停止，轮数只是一个有误差的筛选信号。
- 改变的动作：将短但干净的实验、有效的错误恢复和重复无进展分开标注。对工具和训练样例保存最终状态及发生过的修正动作。
- 推翻条件：若轮数阈值主要丢掉干净演示，就按可验证行为筛选。录像观察仅覆盖列明的静帧。 [第三方英文转写](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-08/)

### `arxiv:2607.24653`

- 原文：[Kimi K3: Open Frontier Intelligence](https://arxiv.org/html/2607.24653v2)
- 范围：arXiv v2。细读 §§4.1.1–4.1.4 的训练阶段及预算控制，Appendix F 的模板、工具和请求选项。
- 机制：SFT 用专门模型产生轨迹，经多阶段核验和人工标注。后续分别用 RL 培养专门能力，再合并。XTML 把边界和终止用保留 token 表达，工具调用有索引匹配结果。一次性选项放在输入消息之后，使不同请求选项可保留历史 KV 前缀。部分任务的训练预算含工具参数。
- 改变的动作：对训练和服务分别记录模板版本、消息边界、工具 schema 和停止条件。验证只改本轮预算时历史 token 是否保持相同，并按实际生成、排队和 cache 情况度量。
- 推翻条件：若适配后工具索引、停止 token 或内容角色不一致，先修模板和运行记录。若更大预算只使记录变长，保留较低预算。此报告的格式和低精度训练不能直接移到其他模型。

### `https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Ultra-Technical-Report.pdf`

- 原文：[NVIDIA Nemotron 3 Ultra 技术报告 PDF](https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Ultra-Technical-Report.pdf)
- 范围：NVIDIA 65 页 PDF，封面 2026-06-09。细读 §3.1、§3.1.1、§3.1.2 的训练、数据与打包。与另一个 arXiv 登记身份为同工作，未核对逐字等同。
- 机制：两阶段 packed SFT 的长度上限为 294,912 和 515,000 token。软件修复数据由两个不同类型的教师在多种工具环境中取得，按轨迹检查有效提交、无进展循环、工具调用格式、调试残留和运行测试。通过任务的轨迹也可能触发过滤。
- 改变的动作：同一优化任务保存实际执行与最后补丁，标出进展、无效迭代、核查不足、异常退出。用独立正确性、性能和可维护性筛选训练样例。读同一问题的多 harness 轨迹时保留各自实际工具能力。
- 推翻条件：若规则过滤错误排除合理探查或成功恢复，先复核样例再定训练策略。若跨工具的收益来自重复任务，按问题族划分。报告的内部数据和基础设施不能视为可复现的本地结果。

### `https://microsoft.ai/pdf/mai-thinking-1.pdf`

- 原文：[MAI-Thinking-1: Building a Hill-Climbing Machine](https://microsoft.ai/pdf/mai-thinking-1.pdf)
- 范围：109 页官方 PDF。细读 §§3.1.4、3.3、3.5 选段，核对 Table 10、self-distillation 的批量和长度。
- 机制：作者从不同 checkpoint 采样，按任务能力使用不同过滤，再将专门教师轨迹合并。合并 SFT 中 STEM 与代码为 56% 样例、89% token。通用帮助与安全为 33% 样例、2% token。作者报告其设计下后者能力未因此下降。普通自蒸馏 SFT 和最终合并的设置各自报告。
- 改变的动作：按样例、完整 token 和承担 loss 的 token 分别统计任务族。检查短动作、工具参数、退出和失败恢复是否被长解释压过。用相同运行条件比较按样例与按 token 的两种训练混合。
- 推翻条件：若目标任务改善同时诊断通用性或工具成功率下降，复查权重。若不同 checkpoint 只生成重复轨迹，保留能够支持不同动作的例子。报告中的能力相关性限于其数据与评估。

### `https://huggingface.co/IFM/K2-Horizon-375B-A23B`

- 原文：[K2-Horizon-375B-A23B 官方模型卡](https://huggingface.co/IFM/K2-Horizon-375B-A23B)
- 范围：细读模型卡阶段表、Release Artifacts、checkpoint inventory。产物表自报更新日期 2026-09-28。未下载 checkpoint。
- 机制：中训练先后在 32K、128K、512K 条件下扩上下文，再将混合向 agentic 与推理调整。卡片写明五个专门模型经过 RL 后合并，之后有 81B、201B、50B token 的三阶段 SFT。最后阶段采用高质量子集和学习率衰减。RL 步数和 token 数未列。
- 改变的动作：给每版训练产物保存输入数据、版本、模板、长度、训练预算与评估记录。比较同一模型各阶段在工具调用、任务跨度和简短回答上的差异。文档有预告时核对实际制品再依赖。
- 推翻条件：若只有最终权重能取得，只评最终结果，不做阶段归因。若超长上下文在目标服务造成成本且无真实任务增益，降低投入。

### `arxiv:2606.24855`

- 原文：[Data Recipes for Agentic Models](https://arxiv.org/html/2606.24855v1)
- 范围：arXiv v1。细读 §§3.1–3.6、§4、Appendix A.3，核对 Figure 3 和 Table 2。
- 机制：对照任务来源、教师、任务改写和轨迹过滤。去掉少于五轮的轨迹，在匹配 token 预算时仍有收益。测试表现最高的教师并未产生最好学生。数据扩展时，多次采样原任务在 31.6K 到 100K 趋平。另一分支改写了少数来源的既有题，增加表述覆盖并继续改善。底层问题数与表述数要分开计量。
- 改变的动作：若收集标注，比较不同工程任务、同一任务换表述、同一任务增加轨迹。采用训练前后的真实任务、同 token 预算和独立硬件。按样例质量查教师，而非只按教师得分。
- 推翻条件：若模板改写只提升同义题，或短轨迹的成功被无谓过滤，换用真实工单及过程标准。不能直接搬用五轮阈值、来源排序或题量。

### `https://huggingface.co/docs/transformers/en/main_classes/tokenizer`

- 原文：[Transformers Tokenizer](https://huggingface.co/docs/transformers/en/main_classes/tokenizer)
- 范围：截至 2026-11-13 核对当前文档的 `apply_chat_template`、助手 token mask、生成提示、继续末条消息及截断。
- 机制：模板将带角色的消息和可选工具等信息转成模型的 token 序列。返回助手 mask 需要模板有 `{% generation %}` 标记。添加新助手头与继续最后一条消息是不同模式。截断可能改变实际可见输入或目标。
- 改变的动作：保存一条最短的工具样例及一条长轨迹，分别渲染训练输入和推理前缀，检查 token、边界、观测和停止位置。比较升级前后实际输出。
- 推翻条件：若相同轨迹在两端产生不同 token 或工具观察被作为目标，先修渲染。模板文档描述接口能力，实际行为由安装版本和具体模板决定。

### `https://docs.axolotl.ai/docs/dataset-formats/conversation.html`

- 原文：[Axolotl Conversation 数据格式](https://docs.axolotl.ai/docs/dataset-formats/conversation.html)
- 范围：截至 2026-11-13，细读角色映射、EOS/EOT、逐消息和逐片段 mask 以及 BPE 空白示例。
- 机制：`roles_to_train` 决定默认目标角色，可用消息与内容片段字段控制局部。EOS 与 EOT 可分别配置，某些 EOT 要求 tokenizer 中为单 token。空格与后一个词合并时，跨分段的 token 可被 mask。
- 改变的动作：同一条 GPU 分析轨迹保留阶段信息，区分模型作出的选择和 profiler 结果。转入训练格式后检查每个 token 的 label，包含带缩进的代码、换行、工具参数与 EOT。
- 推翻条件：若训练配置上设置保留内容，但关键 token 被截断或 mask 掉，调整分段和模板后重算。不同模型的推理块和停止协议需要分别验证。

### `https://github.com/hiyouga/LLaMA-Factory`

- 原文：[LlamaFactory 仓库](https://github.com/hiyouga/LLaMA-Factory)，[数据格式](https://github.com/hiyouga/LlamaFactory/blob/main/data/README.md)
- 范围：GitHub 跳转到同项目的新命名。读 README 功能、数据准备、LoRA 训练和推理入口，以及数据 README 的 ShareGPT SFT、角色映射和工具例子。未运行训练。
- 机制：配置映射对话、工具、系统消息和角色。ShareGPT 示例将人和 observation 与助手和 function 轮次区分。工具调用目标与回传需要遵守预期角色顺序，数据来源可为本地和远程。
- 改变的动作：若做小规模 SFT，先将原始事件映射到数据适配器，用最小样本查原始记录、导入角色、目标 token 与推理输出。LoRA、全量或提示基线用相同独立任务验证。
- 推翻条件：若导入把工具结果变成训练目标，先修角色映射。如果训练不改善任务完成而增加维护成本，保留可比较的提示和原始轨迹。仓库支持列表会随版本变化。

### `arxiv:1704.04368`

- 原文：[登记 arXiv 链接 Get To The Point](https://arxiv.org/html/1704.04368v2)，[课件所指 MiniWoB 出版论文](https://proceedings.mlr.press/v70/shi17a.html)
- 范围：核对登记 arXiv v2 标题、§§2.1–2.3 与 §6。课程日程用 MiniWoB 标注该链接，但实际指向摘要论文。正确的 World of Bits 另有登记，本批单独记录。
- 机制：摘要模型为每一步分配生成和从输入复制的概率。coverage 汇总过去的注意力，减轻重复。作者在新闻摘要验证，复制能力可以保留罕见数字或词，但不能保证事实关系正确。
- 改变的动作：在 profiler 摘要中对运行 ID、单位、原始数值使用能追溯到输入的提取。把重复警报合并后，保留不同 GPU 或时间区间的出现记录。
- 推翻条件：如果抽取正确数字却配错设备、时间段或指标，先校验字段关联。该来源不支持对 MiniWoB 或网页工具成功率的判断。

### `https://proceedings.mlr.press/v70/shi17a.html`

- 原文：[World of Bits 出版记录](https://proceedings.mlr.press/v70/shi17a.html)，[PMLR 正文](https://proceedings.mlr.press/v70/shi17a/shi17a.pdf)
- 范围：ICML 2017 论文。细读 §§2.1–2.3、§§4.1–4.2，核对 MiniWoB 的环境与 FormWoB 的重放边界。
- 机制：MiniWoB 有 100 个小型浏览器环境，视觉输入和鼠标键盘操作得到可控制的反馈。FormWoB 缓存人示范中的 HTTP 请求与响应来离线重放，遇到未见请求结束并记零。离线动作覆盖因此比真实环境窄。
- 改变的动作：用离线 trace 或日志夹具验证工具选择和状态解释。对新请求返回超出夹具覆盖，单独记录这个失败原因，再决定是否做一次真实只读观察。
- 推翻条件：若固定重放将真实可行的动作判失败，报告覆盖并增加可验证样例。合成界面任务成功无法证明在实际工作台上稳定。

### `arxiv:2207.01206`

- 原文：[WebShop: Towards Scalable Real-World Web Interaction with Grounded Language Agents](https://arxiv.org/html/2207.01206v4)
- 范围：arXiv v4。细读 §§3.1–3.3、§4、§§5.2–5.4，核对 Table 4 和 oracle 的假设。
- 机制：用模拟商店、真实商品与人类写的要求检查属性、价格和商品选项。Choice oracle 拿到隐藏信息并穷举选项，规则搜索的成功率由 9.6% 到 85.4%。这一上界的每回合可能需要数百或数千步。RL 训练后，轨迹变短，但选项得分下降。
- 改变的动作：把生成或找候选，与根据实际约束选择候选分开测。对 GPU 优化先用一组已知候选检验选择器是否能识别正确实现，给每项验证实际成本。若快速策略过早选出方案，增加对必需约束的观察。
- 推翻条件：若候选池上限高但实际选择低，优先改选择。若选项筛选良好却找不到有潜力机制，回到生成和诊断。使用 oracle 的上界不代表可部署结果。

### `arxiv:2401.13919`

- 原文：[WebVoyager: Building an End-to-End Web Agent with Large Multimodal Models](https://arxiv.org/html/2401.13919v4)
- 范围：arXiv v4。细读 §§3.2–3.4、§4、§§5.1–5.4、Limitations，核对 Tables 2、4。
- 机制：从当前浏览器状态给交互元素标记，并让模型选择操作。评估比较尾部截图和全部截图。Table 2 的人机一致率从只读末一帧的 75.3% 到全轨迹的 85.3%，后者 kappa 0.70。错误分析区分卡住、定位偏差、貌似完成与提示遵循。
- 改变的动作：诊断助手验收同时拿到最后结果和必要的前因，例如 workload 选取、基线、采集状态及结果来源。用程序或人工复核相似名称、单位与指标范围。比较短轨迹和完整轨迹可否减少误判。
- 推翻条件：若整段信息增加费用却不能减少错误验收，改用结构化证据。若模型忽略测量条件，先用确定检查约束。论文中的网页评分一致率不是 GPU 结果评分的准确率。
