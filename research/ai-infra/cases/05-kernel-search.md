# B300 GEMM 的下一步优化怎么选

这个案例有两个独立的工作记录。第一项比较同一 FP16 persistent GEMM 的两个候选，观察 SASS、profiler 和独立计时如何改变下一步。第二项是另一个 BF16 工程的首次远程 smoke，观察编译阶段失败后应选择哪一种验证。现有记录能定位实验和产物，个人分工需要另外确认。

## FP16 GEMM：从长等待定位到可证伪的代码改动

在固定 B300、(M=2048, N=9472, K=8192) 的实验中，Nsight Compute 在 epilogue 写回路径采集到大量 long-scoreboard 样本。实验记录通过指令地址、barrier 偏移和源码手工关联，将其中 6940/7168 个样本定位到写回 warp 等待 MMA 的一组指令。样本说明等待在哪里出现。等待时间还可能包含上游供给不足，也需要单独测量它占执行时间的比例。另一条线索是 profiler 汇总的 2.8-way shared-memory 警告。逐 PC 检查时，55 个普通 shared 指令的实际和理想 wavefront 相同。这降低了直接改 shared-memory 布局的优先级。两个重复读取 TMEM base 的静态 LDS 站点占普通 shared load 的 85.48%，但它们各只有 2、5 个采样。搬出循环是一个低改动、可检查的假设。 [PC 定位和逐指令证据](</Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/experiments/mlc_gemm_b300/best_shape_steady/ncu_gui/analysis/20260920T105923Z_78d47693/LOCALIZATION.md:7>)

| 当时可以选择的动作 | 依据及优先级 |
|---|---|
| 把固定的 TMEM base 读移到 persistent tile 循环外 | 同形状比较 SASS、正确性和 CUDA events。候选必须先证明循环内的读被消除，再评估耗时。 |
| 改写 epilogue 与后续 tile 的同步 | 先检查 TMEM 复用、accumulator 的生命周期和 TMA 对 SMEM source 的依赖。旧记录已有 full_late、full_early、release 等尝试。此前 full-stage 方案同时改变了等待和输入流水资源，不能单凭等待样本判定改动会获益。 |
| 改 tile 到 cluster worker 的映射 | 另一组 (K=4096) 实验提供形状边界线索。在 74 个 cluster worker 的映射下，(N=9472→9728) 时 tile 数从 148 增到 152，逻辑轮数从 2 变成 3。工作量增加 2.70%，独立计时增加 35.46%。active SM 从 93.80% 降到 66.80%，Tensor active / SM active 约为 88.4% 和 88.6%。优先在目标 shape 邻域检查低填充尾轮。 |

逻辑轮数来自 persistent kernel 的内部任务分配。上述 shape 对支持低填充尾轮假设。改变 shape 也会改变其他工作，且 (K) 与 hoist 实验不同。NVIDIA A100 的 cuBLAS 例子提供同类机制。它的 tile、调度和硬件条件均与本地实验不同。 [同步依赖和已试改动](</Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/experiments/mlc_gemm_b300/best_shape_steady/ncu_gui/analysis/20260920T105923Z_78d47693/LOCALIZATION.md:84>) · [shape 边界数据](</Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/experiments/mlc_gemm_b300/tuning/FINAL_REPORT.md:37>) · [NVIDIA GEMM 指南 §3.2](https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/index.html)

实做的 hoist 候选将 K 循环中的静态 LDS 从 2 条减为 0 条。三组种子下，两版的 58,195,968 个 FP16 输出逐位相同。与 FP32 参考的容差检查也通过。编译器同时把展开因子从 2 改成 4。因此即使后续测得时间差，也需要考虑生成代码的其他变化。 [候选改动、SASS 与数值检查](</Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/experiments/mlc_gemm_b300/best_shape_steady/tmem_hoist/RESULTS.md:27>)

同一 GPU 的 12 个 ABBA/BAAB block 中，候选几何平均收益为 +0.3746%。本次 block bootstrap 的 95% 描述区间为 [−3.1389%, +3.9052%]。12 个 block 都是首测版本更快。按顺序分组后，ABBA 给出 −5.4013%，BAAB 给出 +6.5031%。记录保留了原版。另一次全量 NCU replay 出现 Xid 43 和 LaunchFailed。缩小采集范围后在另一张 GPU 上取得有效报告，可以交叉检查 shared load 的减少。崩溃原因仍未知，跨卡 profiler 结果也无法补足同卡计时。 [独立计时及 profiler 记录](</Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/experiments/mlc_gemm_b300/best_shape_steady/tmem_hoist/RESULTS.md:27>) · [14,400 个事件的独立复核](</Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/experiments/mlc_gemm_b300/best_shape_steady/tmem_hoist/runs/hoist_abba_v2/independent_benchmark_audit.md:7>) · [Nsight Compute replay 和可重复性](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html)

**这个结果改变的下一步。** 若目标仍是确定 hoist 是否值得保留，先用同一 binary 做 A/A，沿用 slot 顺序和计时方式，估计测量的分辨率。若 A/A 也反复出现同方向的首测优势，先让计时稳定，再比较 A/B。若 A/A 稳定，复测同卡、同形状的 A/B。若任务的实际 workload 经常落在尾轮边界附近，另做一对只改 tile 分配的实验，预先写明期望看到的 active SM 和 elapsed time 变化。几个动作可以按所回答的工程问题排序。

## BF16 smoke：生成代码以前先定位失败阶段

另一项 B300 smoke 使用相同源码快照作为 baseline 和 candidate，目的为验证远程执行链。用户批准的范围为一次 smoke、最多 600 秒、关闭自动函数重试。记录显示只提交了一次。调用方和远端 worker 报告的状态不同。CUDA 13.1.80 在编译阶段以退出码 255 结束。构建日志先出现由 `-arch=sm_103a` 产生的通用 `compute_103` PTX，再出现通用 `sm_103` target 拒绝 `tcgen05`。本次运行未得到正确性、性能和 profiler 结果。开发记录随后把调用改成显式 `-gencode=arch=compute_103a,code=sm_103a`。新快照已整理，待同工具链编译验证。 [首次运行、授权和终态](</Users/simida/Library/Mobile Documents/iCloud~md~obsidian/Documents/AI_infra/labs/b300-codex-workflow/work/attempts/first-smoke-20261104/experiment.md:5>) · [原始 build.log](</Users/simida/Library/Mobile Documents/iCloud~md~obsidian/Documents/AI_infra/labs/b300-codex-workflow/work/attempts/first-smoke-20261104/artifacts/logs/build.log:1>) · [NVCC 13.1 §4.2.7](https://docs.nvidia.com/cuda/archive/13.1.1/cuda-compiler-driver-nvcc/index.html#gpu-architecture-arch)

**这个结果改变的下一步。** 如果能在同一编译器镜像执行 compile-only 检查，先保存实际的虚拟和真实 target、工具链版本、退出码及生成产物。只有在编译成功后才安排正确性和计时。若显式 architecture-specific target 仍拒绝该指令，再查生成 PTX 的目标声明和指令形态。工作流应读取 worker 的阶段结果。把编译错误、数值错误、计时不稳定和已验证的性能回退都汇总为候选低分，会使后续选择丢失重要信息。

## 可以迁移到 AI Infra 工作的决策

KernelEvolve 的图搜索把候选选择、代码变换和评估分开，并保存候选祖先及历史。这里值得先借用历史和分阶段反馈：每个候选记录原始现象、能推翻假设的观察、下一步成本和与其他候选的关系。论文的 fitness 可将编译、运行或精度失败都置零。若借用此评分方式，仍应让失败阶段和计时可信度进入下一轮决策。 [KernelEvolve v4 §3.1–3.2](https://arxiv.org/html/2512.23236v4)

可用这两个案例做一次决策回放。在揭示结局前，分别只给出 profiler PC 证据、同卡计时序列和原始编译日志。比较工程师自行判断、固定检查流程及模型辅助判断能否选中下一项有区分力的动作。预先检查所选动作是否有可反驳的假设，是否引用实际证据，是否遵守硬件和工具链约束，以及它能否避免不必要的 GPU 重跑。由真实使用中的错误和节省的决策时间来判断要保留规则、工作流、可执行 harness，还是标注决策样本。现有案例能检验这些选择的可行性。它们还无法确定哪种交付对用户长期工作的收益更大。
