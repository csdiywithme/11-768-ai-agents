# DeepInfer profiling：从组件成本到 decode 关键路径

## 输入与证据范围

组件基线固定 DeepInfer `5537415`、DeepSeek-V4.1-Flash-1017 的模型配置和单张 B300。测试矩阵为每 rank 请求数 `{8,64}`、序列长度 `{8K,128K,1M}`、MTP `{0,5}`，共 12 点。页号、token 历史及 lookup 使用合成输入。真实模型配置提供结构维度。正式数据分成两个 Python 进程，先无 profiler 计时，再在同一 allocation 采短 trace。本次从保存的 provenance 核对两阶段 harness SHA 相同，completion 记录各 12 点通过，H2D CSV 每点保留 10 个事件。 [配置和输入边界](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:36) · [执行协议](/Users/simida/my/projects/deepinfer_optimize/profiling/results/components-paired/allocation-protocol.json:1) · [clean 完成记录](/Users/simida/my/projects/deepinfer_optimize/profiling/results/components-paired/clean/completion.json:1) · [trace 完成记录](/Users/simida/my/projects/deepinfer_optimize/profiling/results/components-paired/trace/completion.json:1)

## 实际调查过程

| 阶段 | 证据与判断 |
|---|---|
| 沿原代码定位工作 | CPU 将每请求的页目录复制到连续紧凑表，向 GPU 提交 offsets 和紧凑表。GPU 算 metadata，再将页目录展开成每 query 一行。64 请求、1M、MTP5 时紧凑表 2 MiB 加 260 B offsets，GPU 有效展开写入 12 MiB。 [固定源码和派生](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:17) |
| 分开度量 CPU、传输与 GPU | CPU 测组合调用的主机 wall time，包含 pinned 分配、token packing 等。H2D 记录两段 GPU memcpy 的 active duration 之和，用 correlation ID 找到 CPU 提交。GPU 展开测热缓存 Graph 中的平均调用。报告分别保留口径和时钟。 [度量定义](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:53) |
| 排除有干扰的试跑 | 早期每个配置点之后启动 profiler 的试跑测得约 150 µs 的 CPU 值。正式结果改成同一 allocation 的两个独立进程，并舍弃早期值。旧轮与新轮也混有 allocation、allocator 和主机差异，原因仍待隔离。 [干扰和处理](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:61) |
| 检查正确性 | 原 eager kernel 与官方 reference 从独立 sentinel 开始，比较 14 个输出，覆盖尾部、dummy、query、块与压缩边界。保留的执行快照与当前维护脚本有差别。维护脚本中的 poison 后 replay 检查只经过静态编译。 [正确性范围](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:63) · [脚本版本](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:75) |
| 尝试原版完整引擎的前置通信 | 八卡 B300 的基础 NCCL 和 GDAKI properties 通过，GDAKI full 与原版两类 storage 初始化失败。关闭可选 LAG 查询后进入 comp channel 创建，再用 native probe 将失败收窄至 DevX event channel。读取保存结果确认 16 个 NIC 的普通 verbs 打开与普通 channel 均成功，DevX channel 均返回 EINVAL。宿主层归因待证。 [前置检查结果](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:107) · [原生 API 探针](/Users/simida/my/projects/deepinfer_optimize/profiling/results/verbs-event-probe/result.json:7) |

64 请求、1M、MTP5 的三项为 **34.24 / 41.12 / 14.77 µs**，依次为 CPU 组合调用、GPU 两次 H2D active duration 之和、热缓存 GPU 展开。三项使用不同区间，且引擎中可能跨轮重叠。此案例能说明组件成本与测量边界。原版端到端关键路径、TPOT、收益幅度和优化后正确性待测。 [原始数据行](/Users/simida/my/projects/deepinfer_optimize/profiling/results/components-paired/component-costs.csv:13) · [证据边界](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:15)

## 瓶颈与待检验解释

- **已见的返工。** 试跑的计时口径被修订。通信调查依次检查设备暴露、基础通信、GDAKI properties、full 路径与最小原生 API。保留的日志能复核调查分支。个人工时、操作者分工、定位过程中重复读取资料的次数未知。 [试跑修订](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:61) · [通信调查](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:107)
- **两种性能解释。** CPU 构表与 H2D 可能阻塞 metadata 和模型计算，也可能被上一轮计算覆盖。用完整引擎时间线对齐 CPU、CUDA runtime、copy stream、consumer stream 和前一轮结束，测出实际暴露的等待。再用无 profiler 的同配置对照核对 TPOT 和吞吐。 [待验证方案](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:126)
- **两种部署解释。** EINVAL 与 runtime 缺少对应 DevX 事件对象处理相符，设备和用户态版本等条件仍需隔离。可用固定 SDK、驱动和设备配置，对照支持原生 DevX 的环境或由平台确认 runtime 能力。当前证据定位至 API，平台归因仍是候选。 [平台证据边界](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:118)
- **下一步优化仍有取舍。** 复用历史页表可能减少重打包和上传，但需处理请求换入换出、重排、slot 复用、块增长与 MTP 回退。若绝大部分工作被覆盖，性能收益可能不足以抵消状态管理。先测关键路径，再决定是否实现。 [源码和候选解释](/Users/simida/my/projects/deepinfer_optimize/profiling/RESULTS.md:30)

## 可验证的产物

用离线证据索引和小脚本关联 GPU 事件、CPU 调用、配置及执行版本。基线用同一批 trace 和日志做人工诊断。评估任务要求指出仍缺的证据和下一项低成本判别实验。反例包括把三个组件值相加当 TPOT、把 Graph p95 当服务 p95、将基础 NCCL 通过当 full 通过、将 EINVAL 写成已确认的宿主原因。

如果信息已齐全、重复操作也有工具，而模型仍频繁过度归因，再比较人工示范、固定检查流程和多步 harness。将新的模型版本、拓扑和故障类型留给评估。

## 反向检索问题

1. 怎样从跨 CPU 与 GPU 的多个来源配对同一运行，保留时钟、stream、依赖和版本？
2. 怎样建立 verifier，抓住“每个数字都对，但端到端结论错误”的陈述？
3. 怎样保存 Agent 已排除的解释、权限与 GPU 预算，使它能从暂停的调查点继续？
4. 当平台预检阻塞主实验时，怎样让工具回传可区分的失败类型，并选出最小的下一项观测？
