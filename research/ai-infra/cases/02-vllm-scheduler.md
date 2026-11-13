# vLLM 调度：找到首次状态分歧

## 输入与证据范围

上游 issue #49097 报告一次 PRIORITY 调度漏访问。报告人在 `e6d1310` 上使用真实 Scheduler、KVCacheManager 和 Request，并用合成 ModelRunnerOutput 推进轮次。这里区分 issue 作者实测、本地课程的纯 Python 回放、指定提交上的历史 pytest，以及真实服务验证。合入 PR #49206 的提交为 `4d341ca`，父提交为 `a07086e`。本地教程在 `7d4c347`，回放按 issue 的 R4 合成状态。 [上游 issue 与 R4 输出](https://github.com/vllm-project/vllm/issues/49097) · [合入差异和测试](https://github.com/vllm-project/vllm/commit/4d341ca829d7fbad351b3a5a17d1405e63dc5bf2) · [本地回放说明](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:15)

## 实际调查过程

1. R1–R3 依次进入 A、B、C。R4 前，5 个可用 KV 块中 A 占 3 个，B 和 C 各占 1 个。每块 16 token。R4 开启 prefill throttling，长 prompt 的 A 被延后，仍在 `running`，但没有进入本轮已分配集合。 [上游复现描述](https://github.com/vllm-project/vllm/issues/49097) · [本地状态定义](/Users/simida/my/projects/infra_framework/vllm/pr/replay_priority_preemption.py:24)
2. B 从 16 个已计算 token 增到 17 个时需要新块。调度器抢占优先级最低的 A，`running` 由 `[A,B,C]` 变成 `[B,C]`。旧版只在 victim 属于已分配集合时修正游标。A 已访问却未分配，删除后游标仍为 1。B 领到配额后游标增为 2，循环结束，C 本轮仍为 RUNNING 但没有配额。报告中的 R5 又给 C 配额。 [原始 issue 输出](https://github.com/vllm-project/vllm/issues/49097) · [教学回放分支](/Users/simida/my/projects/infra_framework/vllm/pr/replay_priority_preemption.py:39)
3. 合入代码在删除前记录 victim 的位置，仅当该位置早于游标时将游标减 1，原有已调度集合仍负责回收预算。上游测试断言 R4 的 `num_scheduled_tokens` 含 C 且 C 保持 RUNNING。本地回放只模拟列表、块和单 token，断言 C 保持运行且有正配额。它给出机制和预期红绿关系。 [合入代码和新增断言](https://github.com/vllm-project/vllm/commit/4d341ca829d7fbad351b3a5a17d1405e63dc5bf2) · [本地断言](/Users/simida/my/projects/infra_framework/vllm/pr/replay_priority_preemption.py:58)
4. 本地教程保存了用同一测试文件检查父提交与合入提交的办法。旧版须命中 C 漏调度的断言，新版须出现 `1 passed`。依赖、导入、模型配置读取或导错 checkout 都会使这项验证失效。本次只读核对代码和保存资料，本地历史 pytest 仍待核验，真实服务影响也缺少测量。 [版本表](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:25) · [历史 pytest 验收](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:75)

## 瓶颈与待检验解释

- **已有调查与返工。** issue 作者还原了逐轮请求状态并验证 C 下一轮恢复。合入变更将游标调整条件与预算回收条件分开。课程材料追加了本地回放和双提交检查协议。个人用时、用户在上游修复中的参与、历史 pytest 的本地执行结果未知。 [issue 活动](https://github.com/vllm-project/vllm/issues/49097) · [本地验收](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:100)
- **状态解释的竞争。** C 可能因预算不足、请求状态变化或循环从未访问而无配额。R4 有空余块、C 仍 RUNNING、B 后游标等于列表长度，支持最后一种。对一个新案例应分别记录访问信号、剩余预算、状态变化与实际 token 配额。 [上游实际和预期输出](https://github.com/vllm-project/vllm/issues/49097)
- **验证瓶颈。** 定位依赖两个集合与一个游标的相对关系。复现还要固定源码版本、依赖环境、测试文件和失败原因。测试输出有 exit code 1 时，仍需检查是否命中机制相关断言。 [历史 pytest 验收](/Users/simida/my/projects/infra_framework/vllm/pr/49206-priority-preemption.md:75)

## 可验证的产物

第一项候选是短定位模板和证据检查器。输入 issue、目标提交、回放与测试日志。输出首次状态分歧、同一断言的前后结果、证据层级和未知项。用导入错误、导错 checkout、运行态 C 被误写为 PREEMPTED、把教学回放写成真实测试等反例检查。预留不同调度策略或不同版本的案例，区分模板学习和机制迁移。

若静态证据齐全时模型仍找不到首次状态分歧，考虑做带状态快照的人工示范。训练样本与评估样本按 PR、调度机制和版本分组。

## 反向检索问题

1. 怎样让 Agent 把多轮日志保留成可回放状态，并定位两个版本的首次分歧？
2. 怎样设计同一条确定性断言，使修复前因正确原因失败、修复后通过？
3. 怎样在工具和状态记录中区分环境故障、未运行、模型回放、真实组件测试和服务实测？
4. 对陌生 PR，固定定位模板和自由代码搜索各减少多少证据遗漏与人工返工？
