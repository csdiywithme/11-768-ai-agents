# L1 和 L14 · 项目证据、协作接口与离线改进

### `https://www.anthropic.com/engineering/building-c-compiler`

- 原文：[Anthropic 原始工程报告](https://www.anthropic.com/research/building-c-compiler)
- 范围：课程链接重定向到同一作者 2026-02-05 报告。读长期运行、并行协调、测试环境、差分定位与评估边界。报告时点和仓库状态分别记录。
- 机制：16 个 Agent 用 Git 中的任务文件避免同时认领同一个问题，在独立容器中工作。作者改造验证器、准备测试套件和进度文件，并将快速测试设为每个 Agent 固定的 1% 或 10% 抽样。最初大家在 Linux kernel 的同一个故障上反复冲突。用 GCC 编译大部分文件，仅对局部换候选编译器，才获得可独立分派的故障。后续仍需定位组合故障。
- 改变的动作：当整模型回归只有一个总失败，用参考实现和候选实现逐层替换。按算子、dtype、输入 shape 和设备建立互相独立的故障。先跑短反馈，再用完整 workload 检查组合效应。并行调优前定义每个分支的实验所有权、硬件配额与停止条件。
- 反证：如果失败由多个优化共同触发，逐一替换都通过，就加入组合消融。如果多 Agent 反复打到同一瓶颈，以相同预算对照单 Agent 的有效实验数。作者报告约两周、近 2,000 会话和约 2 万美元 API 成本。测试套件、CI 和参照选择包含明确的人类贡献。Linux demo 的 assembler/linker 和 16 位启动边界以该报告为准。

### `https://github.com/anthropics/claudes-c-compiler`

- 原文：[Claude C Compiler 仓库](https://github.com/anthropics/claudes-c-compiler)
- 范围：`main` README，blob `2038aae6df6b6bda01ca768ce062998f47a64975`，及 `Cargo.toml`，blob `6d29dab9246878c2b19dfff73638bdf174b5ef38`。核对构建目标、编译阶段、fallback、测试格式和已知限制。没有构建或执行。
- 机制：当前 README 说明前端、SSA IR、优化和多架构后端，声称默认内建汇编器及链接器。Cargo manifest 声明 GCC 汇编、链接与 16 位兼容 feature。测试按目标比较程序输出和退出码。README 写明所有优化等级共用流水线、原子类型限定没有完整跟踪。该状态有别于原报告的 demo 当时使用 GCC assembler/linker。README 的测试声称未独立验证。
- 改变的动作：给算子和框架补丁记录真实执行路径，包括 fallback、架构分派、编译器参数及 profiler 区间。用阶段计时区分编译耗时、运行耗时和生成代码速度。把已有功能检查和目标 workload 性能检查配成两组。
- 反证：如果原生代码路径没有实际运行，或者多个优化等级走同一 pipeline，兼容性测试和参数名称不能证明优化效果。要求记录运行标识和原始结果再判定。

### `https://www.swebench.com/`

- 原文：[SWE-bench 官方榜单](https://www.swebench.com/)
- 范围：当前页面的 benchmark 变体、Bash Only 说明、结果字段和认证标记。未抽取动态排名。v1 论文另记。
- 机制：Full 有 2,294 个实例。Lite 有 300 个。Verified 是人工过滤的 500 个实例。页面把固定 mini-SWE-agent 的 Bash Only 当作 Verified 默认视图。Multilingual 与 Multimodal 的任务分布不同。网页区分模型权重开放与官方实际核验。
- 改变的动作：发布两条 AI Infra 结果。一条固定 harness 评价模型，另一条按端到端任务评价完整系统。两者都标出 workload、设备、预算、种子和证据来源。面向开发用的小样本和用于评估泛化的保留集分开。
- 反证：如果固定 harness 排名与完整系统排名相反，先用同一 workload 检查互相作用，再选择项目方向。动态榜单可展示变体和报告方式，无法单独证明性能工程外推。

### `https://xwang.dev/blog/2024/opendevin-codeact-1.0-swebench/`

- 原文：[OpenDevin CodeAct 1.0 作者报告](https://xwang.dev/blog/2024/opendevin-codeact-1.0-swebench/)
- 范围：2024-05-07 全文，工具、交互预算、容器化评估与结果表。CodeAct 论文和后来的 OpenHands SDK 分别登记。
- 机制：Agent 可向人提问，也可执行 Bash 或 Python。加入定向文件浏览和编辑动作、剩余交互计数及简化解析。评估预建对应代码和依赖的容器。发布的 SWE-agent patch 原报告 54/300，此 harness 复跑 52/300，CodeAct 1.0 报 63/300。AutoCodeRover 结果从论文转录。多项改动捆绑，无法归因到单一工具。
- 改变的动作：比较调优 Agent 前，把相同历史 patch 交给旧 runner 和新 runner。逐项记因安装、patch 解析、超时和断言产生的差异。另测专用 profiler query 能否替代长 Bash 组合，并给出剩余 GPU 预算。
- 反证：如果换 runner 就改变多条对照结果，先归因并校准，再报告 Agent 效果。若结构化工具未减少定位错或无效调用，保持更通用的执行接口。

### `arxiv:2308.00352`

- 原文：[MetaGPT v7 全文](https://arxiv.org/html/2308.00352v7)
- 范围：登记未固定版本，读取 arXiv 当前 v7，2024-11-01。核对 §§ 3–4、Tables 1–3、附录示例和局限。
- 机制：SOP 用结构化需求、接口设计和任务分派连接不同角色。Agent 从发布订阅池选择相关文档。工程角色可写和运行测试并按反馈修改，最多重试三次。SoftwareDev 的比较抽七题，Table 1 中 ChatDev、MetaGPT 无反馈、完整 MetaGPT 的可执行性分别为 2.25、3.67、3.75，完整系统也用了更多 token。角色消融同时改变 token 和任务产物。可执行性、任务质量和维修代价各有定义。
- 改变的动作：先固定交接结构。一个调优任务记录目标 workload、约束、因果假设、变更位置和测量条件。让单 Agent 与多个角色共用结构，再观察交接质量和重复测量。测试反馈回到有能力改动对应阶段的角色。
- 反证：若结构化交接由单 Agent 同样完成，而多角色使背景重建和等待增加，角色分工未证明价值。论文主要测从规格生成软件，不能解释现有 CUDA 或推理框架的维护成本。

### `https://code.visualstudio.com/docs/agents/quickstart`

- 原文：[VS Code Agents 官方 Quickstart](https://code.visualstudio.com/docs/agents/quickstart)
- 范围：页面显示 2026-11-11，读本地 Chat/Agents 界面、手动权限、diff、Agent 自测、人工复核与恢复边界。教程版本可能晚于 L14。
- 机制：用户在当前项目启动本地会话，选择 Agent 和权限。Agent 编辑并在浏览器执行用例。用户可从 diff 选变更并反馈，另做持久化检查。Stop 只停后续工作。Restore Checkpoint 恢复受影响的 workspace 文件与对话记录，外部请求、终端动作和部署有自己的状态。
- 改变的动作：设计统一接手点。用户从变更位置跳到触发它的 profiler 观测、运行参数和可重复检查。停止按钮显示已提交 GPU 作业状态，取消作业和恢复文件各有独立结果。
- 反证：若恢复文件后 GPU 作业仍运行，或 diff 不能定位结果对应的编译产物，接手流程未覆盖远端状态。教程示范单 HTML 项目，无 GPU 或大仓库协作数据。

### `https://github.blog/news-insights/product-news/introducing-github-copilot-ai-pair-programmer/`

- 原文：[GitHub Copilot 结对编程技术预览](https://github.blog/news-insights/product-news/introducing-github-copilot-ai-pair-programmer/)
- 范围：2021-06-29 发布、2022-02-23 更新的产品说明，读上下文、行及函数建议、Codex 和语言范围。
- 机制：从正在编辑的代码取上下文，在输入过程中提出可由人接收的行或函数建议。产品说明给出 API 探索和写测试的用例，但没有提供接受率、人工耗时或正确性对照。
- 改变的动作：对于工程师已确认优化方向、只需要写 benchmark 模板或接现有 API 的任务，用补全作为基线。记录修改后代码、直接接受量、审阅时间和数值问题。
- 反证：补全引入重复同步、错误 stride 或错误精度，即使减少键入也可能增加返工。用相同任务比较轻量补全、固定工作流和后台 Agent 的实际产出。

### `https://github.blog/news-insights/product-news/github-copilot-meet-the-new-coding-agent/`

- 原文：[GitHub Copilot Coding Agent 发布公告](https://github.blog/news-insights/product-news/github-copilot-meet-the-new-coding-agent/)
- 范围：2025-05-19 发布、2025-05-23 更新的历史产品公告。读任务分配、环境、检索、draft PR、审阅、权限和会话日志。
- 机制：从 issue 或 VS Code 启动后台任务，使用 Actions 环境、代码检索和仓库指令。Agent 把增量提交写入 draft PR，用户通过会话日志查看过程并通过评论反馈。公告时点的边界包括专属分支、指定审批人及对工作流运行的控制。
- 改变的动作：比较后台模式与即时补全。把可从 issue 明确定义的测试修复、日志整理和结果汇总委托给后台。让探索性瓶颈诊断保留即时人机反馈。记录从发现问题到可审查工件的时间，以及等待和返工成本。
- 反证：如果一次硬件或 workload 歧义让后台 Agent 无人纠偏地反复重跑，短回路可能更经济。公告的产品功能与客户引语没有说明对匹配 GPU 性能任务的净增益。

### `https://www.alphaxiv.org/abs/2610.skillrefiner-offline-skill-refinement`

- 原文：[SkillRefiner 作者摘要及说明](https://www.alphaxiv.org/abs/2610.skillrefiner-offline-skill-refinement)，[作者实现](https://github.com/OpenHands/SkillRefiner/blob/main/skill_refiner/pipeline.py)
- 状态：原件受限
- 范围：alphaXiv 的作者摘要、作者说明及实现 `pipeline.py` 重点路径，blob `fc0e33b252362332ec16ee86ac7181b78c7030e6`。PDF 入口停在加载页。实现、作者所报结果和论文待核细节各自保留。没有用平台 AI 概述补论文结论。
- 机制：按历史评分分开处理成功与失败轨迹。先摘要，再嵌入聚类。每组提出一项修改，负例额外做证据检查，最后写回技能。实现读取带 trace ID 的 JSONL 和独立评分。无输出 span 被忽略。源码还保存被证据检查拒绝的提议。作者报告改写阶段 token 减少 1.4–42 倍，涉及表格、数学和 PR review。该统计不包含原始轨迹采集。
- 改变的动作：先收集调优中真实出现的决策、可追踪 profiler 数据和最终结果。把 GPU 排队、版本错配、数值失败与性能无改善分开。用重复模式提出条件化技能。保留原失败及备选修正。将同样记录用于人工标注、skills 试验或后续 SFT 比较。
- 反证：如果缺失环境字段使多个同类表象被错聚，或者新规则提高历史通过率却在不同 GPU、shape 或精度下误导，就缩小适用条件。失败轨迹能支持问题诊断，修正规则仍需独立验证。

### `https://github.com/adityasoni9998/SkyRL`

- 原文：[课程登记 SkyRL fork](https://github.com/adityasoni9998/SkyRL)
- 范围：GitHub 文件接口读取 fork 的 `main` 根 README 和 `skyrl-agent/README.md`，blob 分别为 `e86fad40ed8520768e21c3019c3b35c92442ce2f`、`ffac6953cf1b47de7dd36836d4e0aaf1a4921616`。与上游根 README 的 blob `e6923eb82eb8bd4920d61c0940f4043640c66d5b` 不同。未追查完整提交差异或执行训练。
- 机制：fork README 提醒代码向统一 `skyrl/` 包迁移。训练、Agent 层和 Gym 环境各有入口。Agent 层列出可插拔工具、异步派发和 OpenAI-compatible、VERL、SkyRL-Train、Tinker 后端。复现旧 SkyRL-v0 结果给出专用提交。
- 改变的动作：若性能任务能可靠重放，将 workload、结果验收和实验环境定义成可在 harness 与训练使用的契约。替换 rollout 后端时，同时测观测匹配、重复执行、轨迹完整、样本成本和训练吞吐。
- 反证：同一任务换后端后，超时或截断使有效轨迹分布改变时，先检查执行语义及数据收集。README 中的接口目标没有验证目标 GPU 训练兼容性。

### `youtube:UwfjzyLnvMg`

- 原文：[L1 官方录像](https://www.youtube.com/watch?v=UwfjzyLnvMg&list=PLSN0qpDfUvTM&index=1)，[第三方英文转写](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-01/)
- 状态：原件受限
- 范围：官方课程与播放列表条目已核验。官方 watch 入口此轮未能提取正文和音频。此前第三方转写读取 38:15、59:32。本轮另读第三方英文约 19:30–26:40、47:00–49:39 和 52:00–53:39。第三方自动转写有识别误差。
- 机制：转写中讲师回答可用 Bash 或 Python 语法表示动作，并提到与 JSON 的取舍。另指出领域观测形状可通过工具设计或训练改善，讲义列出的 harness 职责在口述中落实到工具、状态和错误处理。
- 改变的动作：对 profiler 的结构化数据、视觉时间线和原始文本各留一个例子。先用相同模型观察数据形态是否改变对同步点和瓶颈的判断，再决定是否标注样本做训练。
- 反证：只有原视频能核验语气、屏幕操作和课堂问答完整性。若转写时间和课件对不上，保留冲突，以官方音画作最终依据。

### `youtube:jMyTCXpEz10`

- 原文：[OpenHands Visual Debugging 录像](https://www.youtube.com/watch?v=jMyTCXpEz10)
- 状态：原件受限
- 范围：课程 L1 第 6 页把该链接标为 Graham Neubig 的视觉调试演示。watch 和 embed 均无可读内容。讲义仅核验标题和演示主题。
- 处理：保留视觉调试的来源入口。OpenHands-Versa 的论文另行精读，提供独立的截图与工具实验。该论文的结果不用于补这段录像。

### `youtube:fjHtjT7GO1c`

- 原文：[Devin 发布录像](https://www.youtube.com/watch?v=fjHtjT7GO1c)，[Cognition 同期作者公告](https://cognition.com/blog/introducing-devin)
- 状态：原件受限
- 范围：录像页返回标题和通用页脚，未提供音画或字幕。读同期 2024-03-12 原始产品公告的环境、协作、展示及性能部分，作为替代证据。
- 机制：公告展示在独立环境内组合 shell、编辑器、浏览器和人工反馈。例子覆盖读文档、创建项目、训练准备与修复公开仓库。作者自报在 SWE-bench 随机 25% 子集上解决 13.86%，其他历史模型的上下文条件不同。
- 改变的动作：把读取文档、建立环境、生成候选和提交可审查结果分别计时。用同一任务、代码状态和执行条件比较后台 Agent 与人工路线。
- 反证：跨任务演示无法给出普通故障的成功率。若接手者要重新定位源码或安装环境，即使 demo 完成，工作流节省仍须单独测量。

### `https://github.com/anthropics/claude-code/blob/71cdddec623889d38af14b7a489670a03186f659/demo.gif`

- 原文：[固定提交的 Claude Code 演示 GIF](https://github.com/anthropics/claude-code/blob/71cdddec623889d38af14b7a489670a03186f659/demo.gif)，[同提交 README](https://github.com/anthropics/claude-code/blob/71cdddec623889d38af14b7a489670a03186f659/README.md)
- 状态：原件受限
- 范围：固定提交的 GitHub 文件接口返回 GIF blob `8f9ba5e684d471fc75a18b00ea871f2ad3c358e3`，没有可审阅图像。另读同提交 README，blob `80aa75edf74a5f9e1269811b4e8b5852be91e66e`。README 确认嵌入该 GIF，说明终端、IDE 和 GitHub 入口。
- 处理：保留 README 的入口说明。GIF 的用户动作、工具反馈和画面未核验。若补到图像，再记录时间和交互。

### `https://app.all-hands.dev/canvas`

- 原文：[课件登记的 Canvas 入口](https://app.all-hands.dev/canvas)
- 状态：原件受限
- 范围：L14 第 33 页引用的应用 URL，公开返回页面标题 OpenHands，正文没有可读行。已精读的 Canvas 仓库与 #797 文档分别登记。
- 处理：只确认入口。该应用的会话、状态和体验没有独立核验。技术判断引用源码、文档和相应演示范围。
