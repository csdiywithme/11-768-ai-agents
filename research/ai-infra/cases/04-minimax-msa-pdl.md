# MSA decode C2：同一改动在不同 PDL 条件下反转

## 证据与归属

本例来自 LCPU 课程作业的 C2 MiniMax M3 MSA decode。与它并列的 C1 FlashKDA 有自己的上游基线和 split2 负结果。C2 的 1.117× 属于 MSA partial+merge 链。C1 原版 FlashKDA 对同语义 FLA 的 1.48–2.89× 是复现。C1 新增 split2 的六个计时形状均退化。课程提供任务、上游快照和参考。实验记录只确认用户授权，由 Codex 主代理和多个子代理协作完成实现、GPU 运行和审查。个人实现范围与用时没有细分记录。 [归属记录](/Users/simida/Documents/Ohmyresume/docs/coursework-resume/lcpu-c1-c2-results.md:3) · [C1 基线及挑战数据](/Users/simida/Documents/Ohmyresume/docs/coursework-resume/lcpu-c1-c2-results.md:39)

- **版本与硬件。** 单张 B300 SXM6 AC、CC 10.3、148 SM，driver 580.95.05，CUDA toolkit 13.1.80，Torch 2.10.0+cu130，Triton 3.6.0。基线固定 vLLM `d4da0c55af3aa231b6209bf77871f3ed36eab0d2`。本轮核对最终运行的 candidate.py SHA256 为 `3e4a2ff87b352c7398ce814cf7ea81cf4e4c1fea33fbb5fe6bec8ee115644773`，并核对两个保存的 partial 函数改成同名后的 AST 相同。默认改动为按输出通道分块的单 warp merge。 [固定环境](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/WRITEUP.md:13) · [运行 provenance](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/results/candidate-paired-20260913T022747Z/candidate-environment.json:1) · [最终改动](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/WRITEUP.md:159)
- **形状与测量。** B={1,4,8,16} × TP={1,4} × KV={BF16,FP8}，D128、GQA16、top-k16、page128、seq8192、dql1、BF16 Q，FP8 K/V 标量 0.25/0.5。TP 是单卡 head 分片形状。每个形状、seed 和 PDL 设置的两端共用同卡、输入和 PDL。每 graph 64 次 partial+merge，21 次交错测量。范围止于热 graph 中的算子链。 [输入与 TP 口径](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/WRITEUP.md:13) · [计时协议](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/experiments/CANDIDATE_EXPERIMENTS.md:27)

## 重算原始结果

本轮仅重新读取 `paired.json`，从 64 组的原始样本重算每侧中位数。每组速度比为基线中位数除以候选中位数。每组 21 对，共 1,344 对。64 组计时后各检查两侧输出，共 128 份，均 finite 且 NRMSE <0.02。完整冻结验收另外在 eager、PDL=false 条件下使用独立 FP64 参考和 168 条 baseline 校准。168 条 baseline 与 168 条候选 held-out 记录均通过同一 digest 的验收。两套检查有各自的覆盖范围。 [原始样本](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/results/candidate-paired-20260913T022747Z/paired.json:1) · [冻结校准](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/results/candidate-calibrate-20260913T020759Z/calibration.json:2) · [held-out 记录](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/results/candidate-verify-20260913T022150Z/candidate-heldout.json:2) · [验收范围](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/experiments/CANDIDATE_EXPERIMENTS.md:18)

| 条件 | seed × 形状 | 速度比范围 | 几何平均 | 退化 |
|---|---:|---:|---:|---|
| PDL=false | 2 × 16 | 1.066–1.268× | 1.11749× | 0 条 |
| PDL=true | 2 × 16 | 0.724–1.388× | 1.08396× | 8 条，4 种组合均在两个 seed 退化 |

PDL=true 的四种退化为 TP1/B1/BF16、TP1/B1/FP8、TP4/B4/BF16、TP4/B4/FP8。TP4/B4/FP8 的两 seed 中位时间均值为 7.337 → 9.997 µs。整个 PDL=true 矩阵的几何平均仍大于 1。采样未涵盖这些退化点的完整 PDL 时间线，调度原因仍需判别。 [PDL 分项表](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/experiments/CANDIDATE_EXPERIMENTS.md:58) · [范围与归因](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/WRITEUP.md:169)

## 对证据索引与检查器的检验

**可以通过弱检查的反例。** 「候选通过全部 graph 输出检查，PDL=true 几何平均约 1.084×，因此可以替换生产路径。」前两项与原始样本相符，第三项越过了证据范围。只核对数字、有限性或全局平均的 verifier 会放过。把「C2 的 1.117×」挂到「C1 FlashKDA」或把 graph P95 写成服务 p95，也是同类错误。

把每条性能主张关联到实现、基线、GPU、PDL、形状、计时层和原始记录。报告同域汇总值时同时列出退化切片。验收检查应抓住上面的错配，同时允许「PDL=false 的课程域均加速」「PDL=true 总体均值超过 1，但四种形状退化」等准确结论。本例独立于前三个案例，但结论已知。它增强了先验证证据索引和小检查器的依据，可作有标签的评估样例。正式比较仍需独立留出。它也要求保留分组级别的条件和结果。没有新试验或人工作业时间数据，工具节省与复杂路线的优势仍待对照。

## 跨到服务 workload 的线索

同一套记录还比较固定的上游 CUTLASS 与 Triton。按实测离散点，CUTLASS 的 attention-only/full 首次获益点在 TP1 为 B8/B16，在 TP4 为 B32/B64。这里的 full 额外包含本实验的 Q 量化和 GPU metadata 更新，仍是热 graph。CUTLASS 使用额外的 FP8 Q 并有 P numerator 舍入差异，尚未通过完整冻结候选验收。这个例子支持先从目标 workload 采集实际 batch、head 分片、PDL 和精度约束，再决定是否研究 dispatch。离散测点无法给出精确门槛，也没有提供服务端到端收益。 [固定版本与计时层](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/experiments/CUTLASS_EXPERIMENTS.md:3) · [离散对照](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/experiments/CUTLASS_EXPERIMENTS.md:23) · [精度边界](/Users/simida/csdiy/wmhpc-training-camp-x-lcpu-ai-infra-seminars/assignment02/team/c2_msa_decode/experiments/CUTLASS_EXPERIMENTS.md:69)
