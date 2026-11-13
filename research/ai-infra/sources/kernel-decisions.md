# 算子决策的来源和适用条件

沿 NVIDIA 的技术博客和文档继续探索。从 model co-design 的 shape sweep 出发，回到 tile 和 worker 排布的机制，再用架构和工具链文档检查哪些候选可以实现。采用实际 shape 计数、SASS 和匹配计时检验迁移。下表中的公开材料用于生成假设和界定边界。

| 来源 | 能帮助做的判断 | 使用条件 |
|---|---|---|
| [NVIDIA AI Model Co-Design，2026-07-10，Figure 5](https://developer.nvidia.com/blog/ai-model-co-design-hardware-friendly-llm-design/) | GB300 上固定 256×128 base tile、clusterMMA、4×2 CGA，细扫 N，观察吞吐如何随 tile 边缘填充变化。若目标 workload shape 可选择，应把 shape 分布与 kernel 几何一起看。 | 作者图展示 NVFP4 下 tile padding 的影响。本地 148→152 例的首要解释是 persistent worker 的尾轮填充。两种机制需要分别数 tile 边缘和轮内并行度。 |
| [NVIDIA GEMM 性能指南 §3.1–3.2](https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/index.html) | 边缘 tile 利用率和末轮 worker 利用率可以分别下降。给 profiler 指标之前就可以用整数计数做预判。 | 文档测量采用 A100、CUDA 11.2、cuBLAS 11.4、108 SM 和固定 tile。移植到 B300 时，从实际代码重算并行 worker 及 round。 |
| [CUTLASS 3.0 GEMM API 的 CollectiveBuilder](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/gemm_api_3x.html) · [Blackwell SM100 GEMMs](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/blackwell_functionality.html) | 先列出 tile、cluster、mainloop stage 和 schedule 的资源耦合，筛掉不适配的候选。接口文档中的 `StageCountAuto` 是根据每 stage 的共享内存成本作选择的例子。 | 3.0 文档的 builder 示例用 SM90。Blackwell 文档给出 SM100 的功能组合。B300 具体能力和资源需回到实际 target 与二进制核对。 |
| [NVCC 13.1.1 §4.2.7](https://docs.nvidia.com/cuda/archive/13.1.1/cuda-compiler-driver-nvcc/index.html#gpu-architecture-arch) | 用虚拟架构及真实架构区分 PTX 和汇编 target。架构特定的真实 arch 简写可能包含通用虚拟 target。应在生成的编译产物里验证是否编译了额外路径。 | 文档举例为 SM90a，B300 证据来自本地 CUDA 13.1.80 的原始 build.log。改后的显式 target 只有待验证状态。 |
| [KernelEvolve v4，2026-07-06，§3.1–3.2 和 Figure 5](https://arxiv.org/html/2512.23236v4) | 把候选选择、变换、适应度函数分开。用 parent-child 关系保存试验历史，沿用有效发现。Greedy、MCTS 和演化搜索都可以实现成选择策略。 | 论文实现不为用户任务自动给出搜索策略。在单卡时序变化主导测量的案例中，先把计时可信度纳入反馈。不同失效类别也有不同的下一步。 |
