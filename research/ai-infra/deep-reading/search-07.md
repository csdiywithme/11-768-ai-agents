# L06、L20 深读 · 编辑工具的契约与实验执行成本

本批细读七项登记来源。六项是课程指定版本的代码，另一项是 Polar 论文。下列 AI Infra 应用为待验证的假设。

### `https://github.com/django/django/blob/2eed554c3fd75dae1beade79f357ffd18d3c4fdf/django/db/models/query.py`

- 原文：[Django QuerySet.bulk_update 固定版本](https://github.com/django/django/blob/2eed554c3fd75dae1beade79f357ffd18d3c4fdf/django/db/models/query.py#L673)
- 范围：读取指定 commit 中的 L610–L724，重点核对登记的 L673 与批量更新过程。文件 blob SHA 为 `f8d848e92623d9cffe0a308000eca599504b56e6`。对照作业给出的原始症状和可能修改位置。
- 机制：批量更新会按字段与记录构造 CASE 表达式，必要时做类型转换，并在数据库事务内执行。给定旧版本把不继承指定基类的属性当普通值包装。作业记录的 F 表达式因此进入字面值路径，受影响行为可以从生成 SQL 和读取的行值观察。
- 改变的动作：看到框架在一个 dispatch 分支上行为错误时，先用最小输入确认「对象进了哪个转换分支」和「结果实际是什么」。对 GPU runtime 检查 dispatch 条件和底层实现。若有两种修改候选，各用邻近数据类型和正常路径验证接口契约。
- 推翻条件：若更宽的判断接受了不支持的类型，或只修复一个样例并改变其他路径，重新检查相关调用和类型约束。此处是作业中的软件语义修复，性能测量应另设基线。

### `https://github.com/OpenHands/software-agent-sdk/blob/df2ea8fa5542d5d2a543e108bc8b2d4fbbab34b1/openhands-tools/openhands/tools/file_editor/definition.py`

- 原文：[OpenHands FileEditorTool 固定版本](https://github.com/OpenHands/software-agent-sdk/blob/df2ea8fa5542d5d2a543e108bc8b2d4fbbab34b1/openhands-tools/openhands/tools/file_editor/definition.py)
- 范围：读取文件 L1–L272。细读请求、返回、声明的路径资源、唯一匹配说明和 executor 装配。blob SHA 为 `82ec8e6a0886b079fa90fbe355f198bf5cf84722`。未查看执行器完整实现。
- 机制：工具为查看、新建、替换、插入和撤销定义字段及结构化观察。替换说明要求原字符串在文件中精确且唯一。资源声明把所有操作映射到解析后的路径，注释说明同路径读写串行，不同路径可并行。输出支持差异、错误和截断状态。
- 改变的动作：性能调优中，让代码编辑与后续读取共享明确的目标版本。对单个补丁要求旧上下文能唯一匹配，对有依赖的多个文件记录验证顺序。把截断作为需要再取观测的状态。
- 推翻条件：比较重读、同路径排队和整工作区串行的正确性与耗时。若资源声明未覆盖并发修改者，读取版本或哈希确认。文件定义只说明 SDK 在该层声明的契约。

### `https://github.com/anomalyco/opencode/blob/57ef3828431790c53f8f333c7ffbfe88770a1812/packages/opencode/src/tool/edit.ts`

- 原文：[OpenCode edit.ts 固定版本](https://github.com/anomalyco/opencode/blob/57ef3828431790c53f8f333c7ffbfe88770a1812/packages/opencode/src/tool/edit.ts)
- 范围：细读指定 commit 的编辑执行入口、路径锁、换行和 BOM、`replace`、diff 及 LSP 诊断。blob SHA 为 `a92e4720c0fccbc162ae0ac0a33e9b121fa9fd5a`。未运行工具。
- 机制：先用精确替换，随后尝试忽略行首尾空白、锚点或其他匹配方法。单处修改要求找到可用的唯一片段，并对异常大的候选跨度设置拦截。工具写入前准备 diff，格式化后重算差异，并返回 LSP 诊断。
- 改变的动作：对高度重复的 Triton 和 CUDA 分支，先试严格匹配。如果候选依赖格式容错，把实际选中的函数、跨度和格式化后的 diff 返回给审阅者。补丁与编译或 profiling 的证据分别保存。
- 推翻条件：用正确但换行不同、同名多个、旧版本代码三类案例计误匹配。如果更宽的匹配增加语义错误或审阅时间，保留严格方式并提示重读。LSP 诊断对数值正确性、寄存器压力和调度收益另无验证作用。

### `https://github.com/badlogic/pi-mono/blob/c1d4c801114545f47c440921d8b3e04aeb1e565d/packages/agent/src/harness/tools/edit.ts`

- 原文：[Pi edit.ts 课程链接](https://github.com/badlogic/pi-mono/blob/c1d4c801114545f47c440921d8b3e04aeb1e565d/packages/agent/src/harness/tools/edit.ts)，[GitHub 跳转后的同 commit](https://github.com/earendil-works/pi/blob/c1d4c801114545f47c440921d8b3e04aeb1e565d/packages/agent/src/harness/tools/edit.ts)
- 范围：按 GitHub 重定向在同 commit、同路径读取 L1–L145，blob SHA 为 `97fa680442c1a871744685fc59203bde2e787217`。细读批量输入兼容、修改队列和取消检查。未核查依赖模块完整实现。
- 机制：同一文件的多个替换以原文件为参照，契约要求唯一且互不重叠。旧的单项输入可归一到数组。工具在文件修改队列内读、应用和写入，保留原始换行样式与 BOM。取消检查也出现在写入之后，所以只看最终取消状态无法判断写入结果。
- 改变的动作：当多个改动共同对应一个假设时，可在同一原始版本上生成独立变更。修改取消后，先核对磁盘差异及实验状态，再决定重试。对两个候选实验，隔离工作区或用已标记的原始版本分别生成。
- 推翻条件：比较单次合并与逐步修改的意图保持、冲突率和回退时间。若多项修改互相依赖，拆成独立假设或用一个更明确的上下文块。这里的文件内修改契约不能替代多文件事务。

### `https://github.com/OpenHands/software-agent-sdk/blob/df2ea8fa5542d5d2a543e108bc8b2d4fbbab34b1/openhands-tools/openhands/tools/preset/gpt5.py`

- 原文：[OpenHands GPT-5 工具 preset 固定版本](https://github.com/OpenHands/software-agent-sdk/blob/df2ea8fa5542d5d2a543e108bc8b2d4fbbab34b1/openhands-tools/openhands/tools/preset/gpt5.py)
- 范围：细读 L1–L75 的工具列表、CLI 差异和压缩配置。blob SHA 为 `dd3e6275f45b35ffc2d836708415cc4267cab7d0`。未复验该模型。
- 机制：该 preset 使用 ApplyPatchTool、终端和任务跟踪，CLI 关闭浏览器。历史压缩由带独立使用标识的 LLM 实例承担，参数设置 `max_size=80`、`keep_first=4`。更换 preset 会同时改变工具、提示参数和压缩。
- 改变的动作：比较优化助手的编辑格式时固定其余工具、提示和压缩。单独记录模型生成与历史整理的成本。运行 ID、基线配置和未处理的验证项通过外部状态可复查。
- 推翻条件：若工具能力提升在只改变编辑接口时消失，则检查浏览器、任务跟踪或压缩的贡献。如果压缩使前后实验条件丢失，调整事件保留方式。代码配置不能证明某模型在目标任务上的优劣。

### `https://github.com/anomalyco/opencode/blob/57ef3828431790c53f8f333c7ffbfe88770a1812/packages/opencode/src/tool/registry.ts`

- 原文：[OpenCode registry.ts 固定版本](https://github.com/anomalyco/opencode/blob/57ef3828431790c53f8f333c7ffbfe88770a1812/packages/opencode/src/tool/registry.ts)
- 范围：细读插件工具适配、工具输出截断、模型工具筛选和调用追踪。blob SHA 为 `9167cb3ea6bc5c8dd075f0f8271adbdec6074b12`。
- 机制：注册表按模型标识字符串筛选 patch 或 edit/write。插件在注册边界提供 schema、执行结果和可选附件。输出截断时保留元数据和可用的输出路径。调用追踪附带工具、会话、消息与可能的调用标识。
- 改变的动作：比较不同模型与调优工具时保留实际可见工具列表、参数 schema 和模型 ID。长 profiler 输出可保留摘要及可复查的完整文件。观察下一轮是否请求正确片段，判断截断是否改变决定。
- 推翻条件：若移植时工具路由根据名称猜错，改用可验证的兼容性配置。若输出摘要漏掉关键 trace 行，改为有来源的分段读取。注册表筛选规则只能说明此版本的工具可见性。

### `arxiv:2605.24220`

- 原文：[Polar: Agentic RL on Any Harness at Scale](https://arxiv.org/html/2605.24220v1)
- 范围：arXiv v1。细读 §§3.1–3.5、§§4.1–4.2 与 Appendix A.1，核对 Table 1 和 Figure 5 的口径。
- 机制：通过 API 代理记录真实采样 token 和 log probability。对可检查的追加历史做前缀合并，遇到压缩或分支则分开保存。准备、执行与结束处理使用独立池和有界等待区，超时仍可保存已有轨迹。在同配置的三个训练步中，合并将 1,185 个请求级更新变成 218 个轨迹级更新，耗时从 189.5 分钟到 35.2 分钟。作者给每个请求传播全局结果的对照出现奖励问题。
- 改变的动作：先为正常工具记录实验会话、分支和阶段成本，保留失败和取消。如果要做 SFT 或 RL，再选择保留的 token 与动作。若准备 GPU 环境与收集验证频繁互相阻塞，用有界预热及独立检查对照。记录模型调用的代理是否改变流式反馈。
- 推翻条件：若合并后失去哪条证据触发动作的对应关系，保留原请求记录。若预热增加空闲资源而未减少结果等待，保留串行流程。论文使用合成下游流式响应，系统数字限于报告的训练设置。
