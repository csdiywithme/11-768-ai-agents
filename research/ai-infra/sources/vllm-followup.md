# vLLM 调度扩展核查 · 回归断言与服务指标

2026-11-12 只读核对。历史修复的父提交为 `a07086e`，合入提交为 `4d341ca`。相邻测试及指标取课程固定的 `v0.33.0`，本地标签与工作树均指向 `4ba787e`。本轮未运行回放、上游 pytest 或服务。 [案例](../cases/02-vllm-scheduler.md) · [上游历史合入差异](https://github.com/vllm-project/vllm/commit/4d341ca829d7fbad351b3a5a17d1405e63dc5bf2) · [固定版本说明](/Users/simida/my/projects/infra_framework/START-HERE.md:21)

## 相邻回归能证明什么

| 证据 | 实际断言与边界 |
|---|---|
| #49206 历史用例 | 用真实 Scheduler 和合成模型输出推进到 R4。断言 C 在 `num_scheduled_tokens` 中且保持 RUNNING。合入代码把「删除位置在游标前」和「已分配请求的预算归还」分开。教学回放还要求 C 的配额大于零。历史同一测试的本地双提交红绿结果待验证。[合入差异和用例](https://github.com/vllm-project/vllm/commit/4d341ca829d7fbad351b3a5a17d1405e63dc5bf2) · [回放断言](/Users/simida/my/projects/infra_framework/vllm/pr/replay_priority_preemption.py:58) |
| Prefill throttling | 单独覆盖合法延后。中途 prefill 的请求本轮没有配额，另一请求继续 decode。下一轮恢复时前者重新得到配额。这能防止把「RUNNING 且本轮无配额」统一判为漏调度。[固定版测试](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_scheduler.py#L1323-L1374) |
| Priority + KV 压力 | `test_priority_scheduling_preemption` 的结尾断言低优先级请求被抢占，高优先级请求仍在 running。名为 `test_priority_scheduling_preemption_victim_selection` 的另一个用例实际只断言等待队列顺序和最高优先级入选。仅凭名字会夸大验证覆盖。[抢占状态断言](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_scheduler.py#L4557-L4569) · [等待队列断言](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_scheduler.py#L4625-L4664) |
| 抢占后恢复与随机回归 | 恢复用例检查 V1/V2 runner 输出保存的历史 token。随机用例覆盖 prefix caching、speculative tokens 与两种压力，检查输出集合和 token 计数的一致性。如果漏掉一个请求且它同时缺席各输出集合，这类一致性断言仍可成立。对 R4 保留有资格的 C 必须获得配额的专项断言。[恢复路径](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_scheduler.py#L5143-L5175) · [随机校验断言](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_priority_scheduler_random.py#L143-L173) · [随机参数](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_priority_scheduler_random.py#L176-L188) |

## 线上观察与判别

- **轮次证据。** 每轮关联请求 ID、请求状态、是否访问、剩余 token/KV 预算、victim 位置和实际配额。固定版 `num_scheduled_tokens` 是请求到本轮 token 数的映射。运行数 gauge 的取值经 `SchedulerStats` 来自 `len(self.running)`，看不到其中哪个请求拿到配额。**本轮漏调度属于请求与轮次的关系。** [调度输出字段](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/core/sched/output.py#L242-L247) · [运行队列计数](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/core/sched/scheduler.py#L2920-L2922) · [指标写入](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/loggers.py#L1145-L1154)
- **服务证据。** `vllm:request_num_preemptions` 在请求结束时记录该请求的抢占次数直方图。漏调度的 C 可能一直 RUNNING，因此即使有 preemption 样本也只能说明某请求被抢占，无法据此证明 C 被遗漏。老版本出现过 `vllm:num_preemptions_total`，服务验证需按目标提交或部署导出核对名字和类型。 [v0.33.0 指标写入](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/loggers.py#L1308-L1310) · [v0.11.2 指标](https://docs.vllm.ai/en/v0.11.2/design/metrics/)
- **延迟证据。** v0.33.0 的 ITL 记录相邻流式输出事件的间隔。逐请求 TPOT 在结束时记一次，固定版实现用 engine 的最后与首个 token 事件差除以生成 token 数减一。单 token 请求记录 0，`bench serve` 的 TPOT 统计会排除此类请求。流式输出合并 token 时，ITL 和 TPOT 也有不同权重。跨前端与 engine 的观测用请求 ID 关联，计算时间差时使用同一进程的时钟或已校准的时间线。 [指标定义与采样](https://docs.vllm.ai/en/v0.33.0/design/metrics/) · [固定版计算](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/stats.py#L594-L615)

最小判别顺序：用已有证据核对漏访问与合法延后，再检查目标版本的测试是否真的覆盖该分支。若要判断生产影响，需要在同一模型、到达序列、优先级分布、并发、KV 容量和调度配置下比较请求事件及 ITL、TPOT、吞吐。仅用一个聚合延迟差会把漏调度、合法延后、真实抢占和执行时间变化混在一起。候选验证报告需写明这些条件和失败分母。 [指标事件说明](https://docs.vllm.ai/en/v0.33.0/design/metrics/#engine-core-events)

## 对调研证据的独立复核

- B300 结论有同次分配原始计时支持，适用范围是 4096³ 预打包 NVFP4 GEMM。DeepInfer 的三列为不同区间的组件测量，端到端影响仍待测。vLLM 有上游 issue 原始输出、固定差异与纯 Python 回放源码三个层次。三个案例均缺少可比较的人工工时。
- 课程快照包含讲次、PDF、录像入口与作业版本。引用出现表当前各条状态为「已登记」。读取范围另在专题和 coverage 笔记中。比较各路线之前，可把两类记录按引用 identity 连起来，明确哪些论文只登记、哪些查了原文和哪些访问失败。
- 仍最能改变决策的低成本验证是用三例已存产物构造判别题。先遮住原结论，要求辨认合法延后、测量口径和缺失证据。用同任务比较人工流程与最小结构化证据模板。记录材料获取、读数、判断和纠正分别用了多少时间。此验证可以给出后续该补工具、改流程还是收集示范的依据。
