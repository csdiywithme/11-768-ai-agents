# 扩展来源 · GPU 性能、计时和候选搜索

以下文档与论文用于区分设备执行、容量约束、计时扰动和 workload 影响。对照 B300 GEMM、DeepInfer 与 CS336 的本地记录。所有新动作是下一步验证，历史实测仍以各案例为准。

### `https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html`

- 原文：[CUDA C++ Best Practices 13.5](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html#assess-parallelize-optimize-deploy)
- 范围：Assess/Parallelize/Optimize/Deploy、性能上限、计时、带宽与资源取舍。
- 方法与边界：先按使用者目标找热点与可优化比例，再改实现并验证。部件耗时比例取决于串行依赖与 workload。引用理论上限前先给分母。
- 工程用法：明确 CUDA kernel 和当前端到端路径的实际关系，核对一个候选若无限快会省下的上限。若主要成本在排队、编译或等待其它流，先补关键依赖，避免追一个无关的热点。

### `https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/index.html`

- 原文：[Matrix Multiplication Background User's Guide](https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/index.html)
- 范围：算术强度、tile 形状、§3.1 Tile Quantization、§3.2 Wave Quantization 及 Fig. 7–8。
- 方法与边界：前者浪费来自不满 tile，后者来自末轮任务不足以填满设备。Fig. 8 的 108 SM 阈值来自固定 256×128 tile 的 A100 测量。
- 工程用法：从 B300 实现重算边缘填充、逻辑 tile、有效 worker 与轮数。保留只跨一种边界的形状对和配对计时。若尾轮活跃 SM 没按预期变化，检查实际调度、驻留或流水资源。

### `https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html`

- 原文：[Nsight Compute · Profiling Guide](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html)
- 范围：Kernel/Application/Range Replay、replay matching、缓存与时钟控制、采集开销。
- 方法与边界：指标可能分多个 pass。Kernel Replay 可改变并发，Application Replay 要匹配相同语义的 launch，Range Replay 的指标可能属于整段。时钟和缓存处理会改变条件。
- 工程用法：对同一代码用干净事件计时确认差异，再用最少所需 counters 解释。若报告含多个 replay 或跨卡，应先查目标路径和匹配。出现 Xid、错误或不稳定计时时，保留失败并用更小诊断分支核对。

### `https://docs.nvidia.com/nsight-compute/ComputeTriage/`

- 原文：[Nsight Compute · Compute Triage Guide](https://docs.nvidia.com/nsight-compute/ComputeTriage/)
- 范围：workload 大小、Speed Of Light、根据瓶颈选下一动作、何时停微调。
- 方法与边界：先看是否有效填满 GPU，再查资源和可达到的上限。微调接近架构限制时，可以转向算法、库或 workload。同一算法跨设备会换瓶颈。
- 工程用法：做候选筛选时记录当前饱和资源和期望会变的读数。若其它优化无法达到目标，比较更换分解方式或已有库。计时无法分辨变化时优先修测量，再判定是否到上限。

### `https://docs.nvidia.com/nsight-systems/AnalysisGuide/index.html`

- 原文：[Nsight Systems · Post-Collection Analysis Guide](https://docs.nvidia.com/nsight-systems/AnalysisGuide/index.html)
- 范围：SQLite 导出、CUDA runtime/GPU activity 的 schema、correlationId 和 CUDA Graph 示例。
- 方法与边界：可把 CUDA API 和 GPU activity 关联起来。上下文、进程和 graph node 也会决定解释。关联描述执行来源，区间重叠本身无法确认等待由哪个事件导致。
- 工程用法：将进程/设备、stream、correlation ID、NVTX 和统计范围一起保存。先检测缺事件、碰撞和重复归因。若要回答关键路径，增加 consumer 的依赖边或单变量干预，再用无 profiler 计时复核。 [CS336 时间线](../cases/09-distributed-training.md)

### `https://docs.nvidia.com/dl-cuda-graph/latest/troubleshooting/performance-issues.html`

- 原文：[CUDA Graph Best Practices · Performance Issues](https://docs.nvidia.com/dl-cuda-graph/latest/troubleshooting/performance-issues.html)
- 范围：GPU/CPU 瓶颈、capture range、graph 与 node 级观察、trace 开销。
- 方法与边界：graph 减少重复的 CPU 提交，收益取决于 launch 是否阻塞实际关键路径。采集粒度会改变观测开销。整体利用率只是线索。
- 工程用法：先看 GPU 空档与主机 launch 是否对应。再做相同 workload 的 graph on/off clean A/B，并以时间线验证捕获范围和 warm-up。若启动已提前很多且 GPU 大部分时间在执行，先查设备端瓶颈或未捕获区域。

### `https://docs.nvidia.com/cutlass/latest/media/docs/cpp/gemm_api_3x.html`

- 原文：[CUTLASS 3.0 GEMM API](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/gemm_api_3x.html)
- 范围：device/kernel/collective/tiled/copy 层次、dispatch policy、CollectiveBuilder、StageCountAuto。
- 方法与边界：mainloop、epilogue、tile、schedule 和 stage 各有接口，实际资源互相制约。该页 builder 的早期例子和支持范围以 SM90 为主。
- 工程用法：给生成器明确哪些参数和硬件特性可一起变更。先用编译资源报告过滤，记录与手工目标的偏差。改变 stage 后若 shared memory 或寄存器限制了驻留，不能单靠单项数据复用作选择。

### `https://docs.nvidia.com/cutlass/latest/media/docs/cpp/blackwell_functionality.html`

- 原文：[CUTLASS · Blackwell SM100 GEMMs](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/blackwell_functionality.html)
- 范围：支持矩阵、cluster/tile、数据类型、block-scaled 和 epilogue fusion 例子。
- 方法与边界：组合依 dtype/layout 与架构。mainloop 需要知道 epilogue 的共享内存以选择 stage。SM100 文档不能自动替代目标 B300 工具链验收。
- 工程用法：先在固定目标的编译产物核对选中的内核、实际 tile/cluster、资源和正确性。若只改变源配置而实际 dispatch 相同，继续查落地路径，不把相同计时当作反证。

### `https://docs.nvidia.com/cuda/archive/13.1.1/cuda-compiler-driver-nvcc/index.html`

- 原文：[CUDA 13.1.1 · nvcc](https://docs.nvidia.com/cuda/archive/13.1.1/cuda-compiler-driver-nvcc/index.html#gpu-architecture-arch)
- 范围：§4.2.7 架构选项、虚拟与真实架构、编译阶段、Blackwell 支持表。
- 方法与边界：PTX 虚拟 target 决定可使用特性，真实 target 决定二进制生成。简写可能增加编译路径。版本和实际命令共同决定结果。
- 工程用法：对 B300 smoke 先收集真实展开的 nvcc 参数、PTX target、ptxas 阶段和错误。用相同工具链做编译探针。通过后再排正确性与性能，避免把编译日志失败当成运行回归。

### `https://developer.nvidia.com/blog/ai-model-co-design-hardware-friendly-llm-design/`

- 原文：[NVIDIA · AI Model Co-Design](https://developer.nvidia.com/blog/ai-model-co-design-hardware-friendly-llm-design/)
- 范围：Hardware-Friendly shapes、clusterMMA/CGA 和 Fig. 5 的 GB300 NVFP4 N sweep。
- 方法与边界：tile 协同提高复用并扩大有效 tile。指定 256×128 tile 和 4×2 CGA 下，N 跨某些边界出现锯齿。文档讨论的 shape 为其具体 kernel 配置。
- 工程用法：使用实际输入 shape 分布决定是否优先做 tile 配合。不要把模型维度对齐原则直接当作 persistent worker 尾轮解释。分别测边缘浪费和波次变化。

### `https://developer.nvidia.com/blog/the-modern-cuda-toolbox-in-practice-a-step-by-step-optimization-walkthrough/`

- 原文：[NVIDIA · The Modern CUDA Toolbox in Practice](https://developer.nvidia.com/blog/the-modern-cuda-toolbox-in-practice-a-step-by-step-optimization-walkthrough/)
- 范围：逐步建立 baseline、Nsight Systems/Compute、优化 CUDA kernel 与回看数据传输和分配。
- 方法与边界：先测整条应用，再逐步改热点。一个阶段提速后再次检查瓶颈。该 walkthrough 用图像处理，收益只适用于示例。
- 工程用法：为 DeepInfer 的 fused gather 和 prefetch 分别检查同请求依赖与设备区间。若主要代价转到 launch、H2D 或排队，改下一项实验。保留总时延，避免把一系列局部收益相加。

### `arxiv:2512.23236v4`

- 原文：[KernelEvolve v4](https://arxiv.org/html/2512.23236v4)
- 范围：§§3.1–3.2 的搜索、变换、fitness、停止，Fig. 10 的轨迹和生产验证说明。
- 方法与边界：用代码和执行反馈维护候选树。候选、下一项变换和评估分开。成功算子的速度以指定 PyTorch 基线和场景衡量，不代表当前本地实现。
- 工程用法：记录每个候选的父节点、失败阶段、正确性、匹配计时和成本。先问现有 verifier 是否能区分真变化。若 +0.37% 落在顺序噪声中，应减少后续自动候选预算，并做一项分辨力检查。 [B300 原始工作分支](../cases/05-kernel-search.md)

### `arxiv:2512.23236`

- 原文：[KernelEvolve，未固定登记链接本次解析为 v4](https://arxiv.org/html/2512.23236v4)
- 范围：与上述 v4 共用正文阅读。原无版本标识单独保留，本轮未逐版比较。
- 方法与边界：搜索预算、每次执行阶段、最终可进入生产的条件不同。论文报告的耗时和 speedup 要以其运行域理解。
- 工程用法：本地同一候选先通过编译和输出门槛，再判断其计时是否可比。工程评估以用户目标 workload 的下一步和结果为准。
