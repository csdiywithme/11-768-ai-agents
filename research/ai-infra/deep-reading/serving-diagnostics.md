# 外部工程 · 从请求事件判别调度、测量和负载

本批沿 vLLM、llm-d 与 NVIDIA 的站点读文档、测试和源码。运行条件、版本和观察区间写入每项。未运行新的服务或 GPU 测试。

### `https://github.com/vllm-project/vllm/issues/49097`

- 原文：[vLLM #49097](https://github.com/vllm-project/vllm/issues/49097)
- 范围：问题报告、R4 状态表、预期和实际配额，查看关闭关联。
- 方法与证据：作者报告某轮同时发生 in-flight prefill 延后和另一请求被抢占，第三个仍在 running 的请求未取得配额。问题报告含请求级证据，未给相同负载下线上尾时延效应。
- 工程用法：同一轮要同时取请求是否合格、是否被访问、预算和输出中的 token。若无这些字段，先补最小状态采集。明确延后条件后再推断调度遗漏。

### `https://github.com/vllm-project/vllm/commit/4d341ca829d7fbad351b3a5a17d1405e63dc5bf2`

- 原文：[上游历史差异和回归](https://github.com/vllm-project/vllm/commit/4d341ca829d7fbad351b3a5a17d1405e63dc5bf2)
- 范围：修改的 scheduler 分支、独立回归用例、末尾请求配额和状态断言。
- 方法与证据：游标调整根据 victim 所在位置。token 预算是否需要归还根据 victim 本轮是否被分配。回归将检查落到原应获得服务的请求，而非仅检查总数；末尾断言 C 在配额映射中且保持 RUNNING。
- 工程用法：候选设计要能解释两个不同状态变量的写入条件。审核回归至少比合入前、合入后和允许合法延后的用例；再加正配额断言。旧提交取网页公开差异，本轮未在本地运行两个版本。 [历史证据与复查边界](../sources/vllm-followup.md)

### `https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_priority_scheduler_random.py`

- 原文：[随机压力测试的固定 commit](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_priority_scheduler_random.py#L143-L188)
- 范围：固定本地 Git 对象，L143–188 的输出断言与矩阵。
- 方法与限制：测试组合 prefix caching、spec tokens 和两类资源压力。检查新、缓存请求集合与配额 key 的一致性，核对总 token 数及相邻字段。某请求若在各输出集合中同时缺席，集合相等仍成立。
- 工程用法：将随机一致性、特定状态的业务进度和真实负载端到端分成三项测试。给最容易被漏掉的请求加应有配额的定向断言。若随机测没有命中错误分支，保留已有覆盖，单独增加定向用例。

### `https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_scheduler.py`

- 原文：[调度器测试固定 commit](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/tests/v1/core/test_scheduler.py)
- 范围：L1323–1374、4557–4569、4625–4664、5143–5175 的实际断言。
- 方法与限制：throttling 用例使部分 prefill 合法延一轮；priority 与 KV 压力断言低优先级被抢占；等待队列测试断言排序；恢复测试按 runner 版本检查不同输出中的历史 token。部分 test name 的语义比实际断言更广。
- 工程用法：代码助手报告“有测试覆盖”时列出具体断言和经过的分支。短步未服务先判断合法节流，再检查访问顺序，恢复时额外比较历史 token。不要将其它分支的绿色结果当作新候选的红绿证据。

### `https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/core/sched/output.py`

- 原文：[SchedulerOutput 固定 commit](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/core/sched/output.py#L235-L254)
- 范围：固定本地 Git 对象，new/cached requests、配额、spec token 与总配额的定义。
- 方法与限制：worker 已缓存的请求只发送差量。num_scheduled_tokens 用 request ID 映射到本轮 token 数。spec 字段可合法缺席，代表该请求无 spec token。
- 工程用法：对一个 req_id 用同轮 map 和实际 runner 输出关联，计算不配额、零输出、推测未接受各自事件。仅有总配额不能排除某请求被漏掉。需分清响应中缺字段与值为零。

### `https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/core/sched/scheduler.py`

- 原文：[Scheduler 固定 commit](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/core/sched/scheduler.py)
- 范围：固定本地 Git 对象，运行请求遍历与 token 预算相关分支，L2919–2924 的计数。
- 方法与限制：running 是调度器持有的集合状态，get_request_counts 用队列长度计算。真正送到 runner 的本轮工作来自独立输出。固定版与历史修复提交不同。
- 工程用法：先匹配源码 commit，按请求记录 eligibility、访问、配额和 runner 回执。若只有 running 数和总 tokens，同一观测可由多种机制产生。增加定向观测比提高 dashboard 采样频率更能区分。

### `https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/loggers.py`

- 原文：[Prometheus logger 固定 commit](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/loggers.py)
- 范围：L1138–1159 的 running/waiting gauges，L1299–1320 的完成样本和 KV compute。
- 方法与限制：running gauge 来自 scheduler stats。每请求抢占次数在完成时观察，尚未完成请求不会形成这次直方图样本。prefill KV compute 用 prompt tokens 减已缓存部分。
- 工程用法：比较长尾时显式处理未完成、超时与取消。若完成者样本少且 workload 仍在运行，先核对队列与右删失，再解释抢占次数或缓存收益。

### `https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/stats.py`

- 原文：[请求指标计算固定 commit](https://github.com/vllm-project/vllm/blob/4ba787ec9959e566d3b587a796dff07673324da5/vllm/v1/metrics/stats.py#L585-L620)
- 范围：完成请求时 queue、prefill、decode、inference、TPOT 的端点，核对单 token。
- 方法与限制：TPOT 是首尾生成事件差除以生成 token 数减一，单 token 记录 0。代码注释写 first SCHEDULED，归档设计文档还有 most recent SCHEDULED 表述。实际受事件更新逻辑决定，不能只按文字选口径。
- 工程用法：用少量请求逐项重算客户端与 engine 指标。包括单 token、合并流式事件和发生抢占。比对字段前先写采样点、聚合权重与排除条件。

### `https://docs.vllm.ai/en/v0.11.2/design/metrics/`

- 原文：[vLLM v0.11.2 Metrics](https://docs.vllm.ai/en/v0.11.2/design/metrics/)
- 范围：指标清单、interval definitions 和 Prometheus 的收集说明。
- 方法与限制：旧版 num_preemptions_total 是聚合 Counter，ITL 和 TTFT 用独立 Histogram。某指标文档可用于该版本排查，不能假定其它版本用同名或同样采样。
- 工程用法：升级前将旧 dashboard 的每个 query 对应到目标版本 exposition 和更新点。特别对每请求次数与计数总量分别核验分母。若上游字段改名或换类型，先验证告警表达式再判断变化。

### `https://docs.vllm.ai/en/v0.33.0/design/metrics/`

- 原文：[vLLM v0.33.0 Metrics](https://docs.vllm.ai/en/v0.33.0/design/metrics/)
- 范围：Metrics Collection、Interval Calculations、Engine Core Events、Frontend Stats、KV Cache Residency。
- 方法与限制：引擎记录请求事件，前端利用批次输出做指标统计以减少 inner-loop 工作。文档要求区间用同进程的 monotonic 端点。KV 生命周期等指标来自抽样块。不同观测可覆盖不同请求和时间段。
- 工程用法：把同一 req_id 的 frontend TTFT、engine queue、prefill 与流式输出区间并排。跨进程比较用可关联的事件或校准时钟。若关键缺口是一个请求一轮的预算，直接取调度事件。

### `https://docs.vllm.ai/en/latest/contributing/profiling/`

- 原文：[vLLM Profiling](https://docs.vllm.ai/en/latest/contributing/profiling/)
- 范围：工具开销、PyTorch worker/前端采样、Nsys、Proton、trace 保存与停止。2026-11-13。
- 方法与限制：Nsight Systems 用于低干扰的时间线，PyTorch Profiler 可按需加 shape、stack、内存而增开销。Proton 提供聚合和 CUDA Graph 归属。多个 worker 独立落盘。停止 profiler 后还有 flush，且 worker 和前端观察窗口不同。
- 工程用法：先做不带 profiler 的对照作为性能证据。若不知 CPU 排队、GPU 执行和同步哪个占主导，选窄窗口 Nsys；若缺栈或 shape，再开对应采集。不能把 flush 等待当作推理时间，也要等产物验证后再标任务完成。

### `https://docs.nvidia.com/nvshmem/api/latest/using.html`

- 原文：[NVIDIA · Using NVSHMEM](https://docs.nvidia.com/nvshmem/api/latest/using.html)
- 范围：PE/GPU 映射、CUDA Execution Model、collective launch、数据一致性和完成语义。
- 方法与限制：使用 device-side 同步或 collective 的 kernel 应使用 collective launch 以保证进展。一般一侧 RMA 与同步有不同启动要求。可见顺序和远端完成是独立约束，需按 API 及 stream 检查。
- 工程用法：DeepInfer overlap 候选先写 producer、consumer、PE 和数据可见点。只移前取数而无数据就绪保证的候选进入正确性检查。用小规模重复校验输出、等待完成和进展，再跑计时。工具仅把已观察到的 copy 与关键路径并列。

### `https://docs.nvidia.com/nvshmem/release-notes-install-guide/best-practice-guide/performance.html`

- 原文：[NVIDIA · NVSHMEM Performance](https://docs.nvidia.com/nvshmem/release-notes-install-guide/best-practice-guide/performance.html)
- 范围：16 字节对齐、block 集体变体、不同 transport 和配置调优建议。
- 方法与限制：对齐有利于运行时合并传输。线程合作对 NVLink 的收益和其它远端 transport 不同。文档先要求证据显示通信是瓶颈，并建议一次改一个配置。
- 工程用法：在固定数据分片和远端比例下分别比较对齐、请求粒度、共同参与线程数。测 bytes、通信墙钟、被覆盖时间和总 step。若 kernel 快而通信仍藏在计算后，应报告瓶颈没有移动，避免部署更复杂的参数。

### `https://github.com/deepseek-ai/DeepInfer/blob/main/doc/inference_deployment_qa.md`

- 原文：[DeepInfer 上游部署问答](https://github.com/deepseek-ai/DeepInfer/blob/main/doc/inference_deployment_qa.md)
- 范围：集群与存储条件、EPLB、DP prefill/decode 调度、compute quota 机制。2026-11-13 的 main。
- 方法与证据：部署把长短 prompt 分桶，prefill chunk 按含已计算 prefix 的拟合成本和时间预算选择。热门 expert 可复制，报 slowest rank/mean。3FS 与生产配置使前缀命中、存储和网络条件不同于本地 smoke。
- 工程用法：先记录真实输入长度、缓存状态、调度时间预算和各 rank 等待，再比较固定 chunk、时间预算与不同实例配比。线上容量与实测数字仅用于选应采指标。单机本地无法验证相同线上优势。

### `https://github.com/deepseek-ai/DeepSpec`

- 原文：[DeepSpec 官方仓库](https://github.com/deepseek-ai/DeepSpec)
- 范围：README 的 data preparation、training、eval 和 released checkpoints。2026-11-13。
- 方法与限制：pipeline 先用 target 重新生成、构建 cache，再训 drafter 与测 acceptance。默认 Qwen3-4B target cache 约 38 TB。作者给出的 checkpoint 用对应 target 的 non-thinking 样本训练。训练数据准备和端到端 serving 时延要单独测。
- 工程用法：先在固定 target 上按 workload、thinking mode、长度和温度分桶检查 proposed/accepted、校验时间与实际收益。若已有 drafter 的低接受率仅集中在某分布，先做该分布诊断与 checkpoint 比较，再考虑代价高的 target cache 与训练。

### `https://github.com/llm-d/llm-d-benchmark`

- 原文：[llm-d-benchmark](https://github.com/llm-d/llm-d-benchmark)
- 范围：scenario、DoE、计划与部署、smoketest、workload/harness 参数。2026-11-13。
- 方法与限制：scenario 统一部署规格，smoketest 核对实际 pod，DoE 编排 infra 与 workload 组合。不同 load generator 可换，但其请求语义、时钟和输出指标要分别对照。文档明确区分部署和负载 override。
- 工程用法：先用同一场景和镜像跑人工/现成方案基线。比较 Agent 建议的候选是否比场景矩阵更快找到改进。核对实际 deployment 与渲染配置，再比较相同负载的指标和 GPU-hours。单独记录哪部分可以直接复用已有工具。

### `https://github.com/llm-d/llm-d/blob/main/guides/README.md`

- 原文：[llm-d Well-Lit Path Guides](https://github.com/llm-d/llm-d/blob/main/guides/README.md)
- 范围：路由、KV、P/D、各平台、RL rollout 集成和 smoke/bench 入口。2026-11-13 的 main。
- 方法与限制：指南按部署场景提供已测的构件，路由、引擎、加速器、网络和缓存依赖不同。main 会领先 release，清单可作为起点，实测兼容范围要回到各 guide。
- 工程用法：把目标模型、设备、拓扑、吞吐与尾时延填入同一比较矩阵。先选现有可跑的配置复现关键指标，再改变调度或并行。对需要跨节点的候选核验实际网络再评估。

### `https://github.com/llm-d/llm-d/releases/tag/v1.0.0`

- 原文：[llm-d v1.0.0 发布说明](https://github.com/llm-d/llm-d/releases/tag/v1.0.0)
- 范围：已签名 tag 指向 378b42d，组件表、实际 model-server image、P/D 基础设施要求和 testing matrix。
- 方法与限制：项目统一了多项组件版本，但基础 vLLM 的 image tag 随设备路径变化，CUDA 与 TPU 也有差异。P/D guide 有 LeaderWorkerSet 与 DisaggregatedSet 依赖。发布说明区分核心、生态与孵化。
- 工程用法：重跑前按 router、sidecar、engine、插件、集群依赖记录镜像摘要和配置。若结果跟原博文不同，先对比实际启动的 image、拓扑和缓存路径，避免在版本漂移下归因。

### `https://llm-d.ai/blog/llm-d-v1.0-state-of-the-art-inference-ready-for-day-2`

- 原文：[llm-d v1.0 · 发布工程报告](https://llm-d.ai/blog/llm-d-v1.0-state-of-the-art-inference-ready-for-day-2)
- 范围：Agent-Aware Serving、拓扑、跨设备支持与 TPU 对照。核对 release note 对应组件。
- 方法与证据：session-aware routing 尽量将后续请求发往仍持 KV 的副本，满载时重平衡。并介绍 turn priority 与服务端会话状态。在指定 TPU 7x agentic coding 对照，前缀路由加 host KV offload，相对 Kubernetes Service 报告 TTFT 中位数下降超过 85%、总吞吐 2.4 倍。
- 工程用法：将副本黏性、负载保护和 host offload 分别消融。每次固定会话到达图、流量升高与热点分布。若黏性提高 hit rate 同时增加队列长尾，应调溢出阈值并看 SLO goodput。TPU 的组合增益不能外推给其它设备或单项插件。
