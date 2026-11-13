# L13 · 事件证据、运行边界与防御对照

### `arxiv:2309.15817`

- 原文：[ToolEmu 无版本入口](https://arxiv.org/html/2309.15817) · [核对的 v2](https://arxiv.org/html/2309.15817v2)
- 范围：两入口指向同一 v2，核对 §3.2 helpfulness、§4.2 表 3 及 §7。
- 方法与限制：人评对模拟环境轨迹的确认揭示模拟和评分两层误差。真实失败频率不可由富集压力测试直接推得。
- 工程用法：用小样本真人复核低成本故障模拟的分类，再在不同后端使用确定性夹具。与 v2 只有一份证据。 [原记录](defensive-evaluation.md:3)

### `arxiv:2406.13352`

- 原文：[AgentDojo 无版本入口](https://arxiv.org/html/2406.13352) · [核对的 v3](https://arxiv.org/html/2406.13352v3)
- 范围：两入口指向同一 v3，核对 §§3.4、4.3 的正常任务效用与防御分母。
- 方法与限制：按成功完成任务和执行不合规动作分别度量，限制工具可阻止一部分操作，也可能减少需要新工具的合理完成。
- 工程用法：同一个 bench 任务用只读、隔离写入和需要升级权限的对照，报告完成率、拦截与等待。与 v3 只有一份证据。 [原记录](defensive-evaluation.md)

### `arxiv:2310.08419`

- 原文：[PAIR v4](https://arxiv.org/html/2310.08419v4)
- 范围：§4.3 防御对照与表 5、§5 局限。阅读静态防御效果和分母。
- 方法与证据：对固定测试材料比较两个防御和无防御的结果，效果随模型而变。作者明确静态实验没有测随防御变化的行为。筛选器的原有结果不能外推到真实有权限的工具链。
- 工程用法：为合法 workload 保留输入误拒与阻止非预期动作的双对照。运行时校验真实 API 或文件作用域。文本判分和能否实际改变系统状态分别测量。

### `arxiv:2507.20526`

- 原文：[Security Challenges in AI Agent Deployment v1](https://arxiv.org/html/2507.20526v1)
- 范围：§§2.1–2.3 的任务、评判与规模，§5 和附录 B.1 的判据与误判。阅读防御评价。
- 方法与限制：作者在 44 个场景和 22 个系统上做竞赛，环境有不同政策、工具与状态。筛过的挑战失败样本可发现边界，但无法估真实工作量中自然失败发生率。
- 工程用法：在正常轨迹上收集代表样本，另用更高风险的无副作用 fixture 作回归。标出候选在什么模型、工具与环境权限上观察。安全判断需要真实状态变化而非只看文字。

### `https://app.grayswan.ai/arena/challenge/agent-red-teaming/leaderboard`

- 原文：[课程登记的排行榜](https://app.grayswan.ai/arena/challenge/agent-red-teaming/leaderboard) · [组织者正式报告](https://arxiv.org/html/2507.20526v1)
- 范围：排行榜返回 403。只读课件 p.14 的来历与组织者 §§2.1–2.3 和附录 B.1。
- 状态：原件受限
- 工程用法：引用静态报告的实验设置时保留版本。动态排行榜未取得，不能声称核对名次或当前系统。

### `https://msrc.microsoft.com/update-guide/vulnerability/CVE-2025-32711`

- 原文：[MSRC 公告](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2025-32711) · [Microsoft CNA 提交的 CVE 记录](https://raw.githubusercontent.com/CVEProject/cvelistV5/main/cves/2025/32xxx/CVE-2025-32711.json)
- 范围：公告网页只返回 JavaScript 壳。核对官方 CNA JSON 的描述、受影响产品、引用与日期。2026-11-13。
- 状态：替代版本精读
- 方法与限制：登记说明适用 Microsoft 365 Copilot，属数据机密性问题。CVE 记录给事件级事实，无法用来估任意研究助手的概率。公告 UI 仍未取得。
- 工程用法：外部资料和内部仓库读取应有明确来源和数据边界。用同一读取任务检查无授权目的地的输出与工具调用是否被隔离。只需要本地判断时保持资料留在原环境。

### `https://thenewstack.io/ai-agents-credential-crisis`

- 原文：[课程登记报道](https://thenewstack.io/ai-agents-credential-crisis) · [Railway 平台复盘](https://blog.railway.com/p/your-ai-wants-to-nuke-your-database)
- 范围：登记新闻返回 403。核对课件 p.4 与平台复盘的恢复、延迟删除、备份隔离和权限。
- 状态：原件受限
- 方法与限制：平台报告客户生产数据库恢复，API 与 UI 的保护曾不一致。复盘解释平台动作和恢复；登记媒体未核全文。
- 工程用法：回收试验产物时用实际环境标识和明确允许范围，区别清理临时资源与共享状态。用可恢复的测试资源验证快照可独立找回。生产和 staging 的凭据范围应通过可观察状态核验。

### `https://gambit.security/blog-posts/autonomous-ai-agents-online-retailers-25-a-company`

- 原文：[Gambit Threat Intelligence 调查报告](https://gambit.security/blog-posts/autonomous-ai-agents-online-retailers-25-a-company)
- 范围：调查来源、观察范围、影响与防御结论。阅读证据边界和运维措施。
- 方法与限制：报告根据取得的调查材料归纳一场持续事件。报告范围内的计数和成本不代表可重复成功率或所有受影响组织。自动化可以缩短异常进展与响应之间的时间。
- 工程用法：GPU 工作流的批量运行需保留预算与资源范围。用可信运行事件发现异常重试和快速扩张，结合最小权限和独立的停止控制。只读问答不能直接调用外部变更。

### `https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Super-Technical-Report.pdf`

- 原文：[NVIDIA Nemotron 3 Super Technical Report，2026-04-03](https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Super-Technical-Report.pdf)
- 范围：Fig. 1、§2.1 的 NVFP4/LatentMoE/MTP、Fig. 5、p.22 的防御性 SFT 数据和质量检查。
- 方法与证据：把任务类别、应答策略与生成示范分步，并用分类检查质量。图 1 约 2.2 倍与 7.5 倍来自 B200、8K 输入/64K 输出、各模型在 vLLM/TRT-LLM 取较优的组合。图 5 是 B300 TP1 的 MTP 对照。这两项环境和分母不同。
- 工程用法：如果需要示范数据，先按任务与允许动作写标签，再比较回复与真实工具行为一致性。模型服务则要在同 dtype、engine 与 workload 内测 MTP，另报每用户和总吞吐。跨模型结果保留框架选择条件。

### `youtube:Mbonnq-PUGM`

- 原文：[L13 官方录像](https://www.youtube.com/watch?v=Mbonnq-PUGM)
- 范围：官方播放列表与缓存页元数据核对。未获得完整音视频或官方字幕。
- 状态：原件受限
- 工程用法：隔离与防御评估以同课 PDF 和所引原文为依据。此条支持找到原录像入口。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-13/`

- 原文：[L13 第三方课程页](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-13/)
- 范围：页面显示 0 项内容，课程笔记和字幕未提供，仅有 PDF 引用。
- 状态：原件受限
- 工程用法：用于回查课程链接，无第三方字幕证据。 [课件阅读](isolation-and-traces.md)
