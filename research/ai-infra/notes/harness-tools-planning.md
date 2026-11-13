# 工具接口、执行计划与可复验的代码变更

本专题对应 L1、L2、L5、L6、L7 和 A1。页码按 PDF 文件的顺序，从 1 开始。 [阅读记录](../coverage/harness-reading.md) · [上下文、技能与记忆](harness-context-skills.md)

## 模型提出动作，运行时执行动作

L1 把模型、运行时、沙箱、推理服务、训练和观测放在同一系统图里。遇到工具选错、进度中断、执行不可靠或费用过高，可以分别定位模型、接口、环境与服务。改变权重可以改变行为。改工具、权限与状态可以改变模型观察和执行的条件。两种改动都需要用相同任务重测。 [L1 第 9–19、26–31 页](https://www.cmu-agents.com/slides/lecture-01-agents.pdf#page=9)

L2 把输入 schema、模型序列化、解析、校验、执行、回传 ID 分成不同环节。受限解码可以约束 JSON 结构与类型。执行端仍要验证参数含义、访问范围和当前状态。并行任务独立时，共享一次生成和等待。结果乱序时要按 ID 归属，依赖前一步的操作要串行。重复尝试时，要区分暂时性工具失败与模型错误。 [L2 第 19–29、47–56 页](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf#page=19)

A1 要求每个 tool call 得到一个 observation，坏 JSON 和未知工具返回可恢复的错误。步骤上限与结束动作由控制器管理。CodeAgent 在 /testbed 生成补丁，评分把补丁应用到干净环境后重跑回归与原有套件。课堂的调用 ID 图强调回传关联，A1 进一步限定了同一动作内的关联，供应商可以跨回合重复使用同一个 ID。 [A1 Part 1](../../../assignment-1/ASSIGNMENT.md#part-1-build-the-coding-agent-and-repair-the-chess-app) · [A1 工具与轨迹测试](../../../assignment-1/tests/test_agent.py)

OpenHands SDK 提供一个可比较的实现。它用 Pydantic 对 action 和 observation 定义类型，从定义生成工具 schema，并把工具注册、运行时解析和执行器分开。这样可以减少格式校验和分派的重复实现。GPU 型号、commit 是否匹配、此次测量是否适用等领域条件仍由调用方定义。只做本地只读调用时，可以先比较普通 Python 函数与 SDK 工具接口。需要跨进程发现或独立凭据时再比较 REST/MCP 的配置与维护成本。 [OpenHands Tool System，Core Responsibilities 与 Key Components](https://docs.openhands.dev/sdk/arch/tool-system) · [L2 第 36–46 页](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf#page=36)

**Infra 迁移假设。** 先让只读工具返回结构化的运行配置、实际指标、失败类型和证据位置。若模型常把 NCU replay 当作 GPU event 计时，工具可显式返回采集方式与会话标识。用两份口径不同、字段相似的记录检查模型是否仍混淆。选错工具、丢失状态和调用顺序错误是评估运行时的信号。若工具结果已经清楚而结论仍错，保留该轨迹作为评估样本。

A1 的棋局实验把三种改动分开。第一种在 observation 中提供合法动作。第二种增加不改变真实棋盘的模拟工具。第三种让沙箱中的 Python 组合模拟与一次真实动作。对两个模型分别记录合法走法列表有无时的调用数、拒绝率和是否达到终局。实际状态可能已被脚本改变，所以执行后重新读取。性能调查可以用同样结构区分“给模型更好的读数”“提供廉价模拟或回放”和“允许操作真实环境”。先用可撤销的状态回放测试输出，再用已授权的真实运行核验性能。 [A1 讲义 Part 3](../../../assignment-1/ASSIGNMENT.md#part-3-build-a-chessagent) · [棋局 dispatcher](../../../assignment-1/src/assignment/agent/chess_agent.py)

## 工作流、单 Agent 与多 Agent

L5 第 16 页按决策发生的时间区分固定工作流、执行前制定的计划、执行时修订的计划。固定工作流适合已知步骤与可列举异常。环境反馈会改变后续调查时，动态计划可以减少沿错误方向重复工作。第 24–27 页讨论推理和交互的预算分配。发现缺少能区分两种解释的证据时，一次范围明确的读取或测试可能比延长推理更有用。这个判断取决于观察可靠性、执行成本和副作用。 [L5 第 15–27 页](https://www.cmu-agents.com/slides/lecture-05-planning.pdf#page=15)

第 21 页指出固定角色可能阻止验证者返回定位步骤，交接摘要也可能遗漏必要证据。第 34 页用旅行规划说明全局约束跨子任务。第 40–44 页的多 Agent 实验对应特定模型和任务子集。把同一 commit、workload、计时口径作为各子任务共同输入，再按数据依赖决定是否并行。用相同案例比较单人或单 Agent 的工作流与并行方案，记录交接造成的返工和实际墙钟时间。 [L5 第 21、32–46 页](https://www.cmu-agents.com/slides/lecture-05-planning.pdf#page=21)

MACU 作者的两组结果可以校准并行预期。Odysseys 45 个简单任务中，把最大并行 worker 从 1 改成 4 时，中位墙钟从 25.4 分钟变成 7.9 分钟。作者排除了波动的推理开销。Online-Mind2Web 的主要实验里，MACU 成功率从 52.2% 到 55.6%，中位墙钟从 18.5 到 33.6 分钟。前者是并行度消融，后者是完整方案比较，任务集和计时边界都要保留。可以将独立源码检索和数据读取作为并行候选。基线构建、测量和判定之间的依赖仍然存在。 [MACU v1 §4.2 与附录 B](https://arxiv.org/html/2606.01533v1)

**Infra 迁移假设。** 如果 workload 调优每次都经历固定的配置抽取、基线检查、正确性验证和出报告，先比较现有手工步骤与固定工作流。如果 trace 表明原假设不成立，且下一条观测要由新证据决定，可以在这段引入可修订计划。独立证据可以并行检索。变更同一个环境或读取依赖上一步输出的动作要按顺序完成。

## 验证代码与性能结论

L6 的定位、编辑、验证循环把假设和证据联系起来。先在指定初态复现，再检查改动及未涉及行为。执行验证允许不同实现达到相同行为，但有限测试只能覆盖声明的条件。FAIL_TO_PASS 验证原问题得到修复，PASS_TO_PASS 验证选定的既有行为保留。性能优化还需要定义输出一致判据和计时指标。第 37 页给了 Agentless 固定工作流。第 74–79 页把预测执行结果与实际执行区分开，并要求比较预测错误、任务质量、工具成本和降级策略。 [L6 第 23–27、32–48、74–79 页](https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf#page=23) · [Agentless 作者仓库](https://github.com/OpenAutoCoder/Agentless)

Agentless v2 将定位分为文件、函数或类、具体编辑点。它生成候选补丁，并用复现测试与回归测试筛选。论文 Table 4 在同一 SWE-bench Lite 口径下，从投票 77 个修复，到加回归测试 81 个，再到加复现测试 96 个。该表比较选择机制，也报告了额外成本。值得迁移的是逐层定位和分离验收，不能用这组数字估计当前框架或 Infra 场景的成功率。 [Agentless v2 §3 与 Table 4](https://arxiv.org/html/2407.01489v2)

L7 第 17–19、25–27、35–39 页区分单步动作、最后状态、操作过程。WebArena 例子中的简化 verifier 检查目标 URL 的仓库名，遗漏了题目要求的空仓库。生成了一份性能报告也可能遗漏配置一致性。可以检查输出中的数值与源记录，再人工复核指标定义、排除样本和结论边界。长任务用阶段证据定位错因，最后核对完整目标。 [L7 第 17–27、35–43 页](https://www.cmu-agents.com/slides/lecture-07-computer-use-agents.pdf#page=17)

L7 把 GUI grounding、示范模仿和依靠验证结果优化行为放在不同阶段。CUA-Gym 作者把任务、可复现初态与奖励函数做成配套样本。生成初态和正确终态的角色与写奖励的角色分离，再做实际 rollout 检查可解性和判据。Gym-Anything 则用真实软件环境、可审查的检查表和筛出的示范评估 SFT。论文在 CUA-World-Test 的平均检查表分数从 12.7 到 22.5，全项通过率从 1.6% 到 4.4%，对应其小模型、VLM 评估器和预算。对于 Infra，先在无计费的历史输入上把初态、应保留行为、测量与验收分离。若检查结果只覆盖文件存在或数值格式，先补错误样本和证据条件，避免用有偏验收扩大示范。 [L7 第 59–74 页](https://www.cmu-agents.com/slides/lecture-07-computer-use-agents.pdf#page=59) · [CUA-Gym v2 §2.1](https://arxiv.org/html/2605.25624v2#S2.SS1) · [Gym-Anything v1 §4.1、§6–7，Table 3](https://arxiv.org/html/2604.06126v1#S7.SS1)

**Infra 迁移假设。** vLLM 案例可以用同一条不变量检查前后状态。把离线教学回放、上游 pytest 与真实服务分开标注。对于性能结论，设计一个数值和表格都正确但 baseline 来自不同 workload 的反例。如果检查工具放过反例，先修评估与配置记录，再比较工作流或模型。对于只有 GUI 的 profiler 界面，可以把 CUA 作为候选。先验证从文件或 API 取得结构化结果的覆盖率与时间成本。 [L7 第 56–57、75–80 页](https://www.cmu-agents.com/slides/lecture-07-computer-use-agents.pdf#page=56)
