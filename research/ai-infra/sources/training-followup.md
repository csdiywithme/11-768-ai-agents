# L11–L12 原始资料复核

核验日期为 2026-11-12。以下结果来自论文或作者的工程报告。本轮检查了算法假设、对照条件和数字分母，未重跑实验。课程入口为 [L11 p.24–27](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf#page=24) 与 [L12 p.14–17](https://www.cmu-agents.com/slides/lecture-12-rl-systems.pdf#page=14)。

## CTPO · 定位采样分布的差异

同一个 token 的概率比只能说明「在相同历史下，两个策略选这个 token 的概率有多不同」。它也可能来自目标策略几乎不会到达的历史。CTPO 对当前位置以前的动作概率比连乘，包含到达历史的概率差异。它省去当前 token 之后的比率，减少无关后缀引入的方差。无偏性证明针对原文的 token-level MDP、适用的采样支持与只依赖当前前缀的真实 advantage。实际算法采用组内结果奖励作替代，再加按位置增长的 clipping 区间。实际目标的偏差应单独评估。[CTPO v1 §2、§3.1–3.3、附录 A](https://arxiv.org/html/2605.07331v1#S3)

作者用 Qwen3-4B/14B 做最多 5 轮的 Python 工具数学题。四项竞赛评估的 avg@32 平均分较最强基线 GSPO 高 3.7、3.3 个百分点。14B 在 BRUMO 和 HMMT 上分别有其他方法更高。报告均值时应保留任务与采样口径。A3 用当前 token 的比率和过期组丢弃。若以后已有 RL 轨迹且训练不稳，可以先按策略版本、位置和任务比较比率与剪裁率，再定位是模板和 token 对齐、行为策略陈旧，还是验证器信号。这个检查所需的是可观测的采样记录。[CTPO v1 §4.1–4.4、表 2–3](https://arxiv.org/html/2605.07331v1#S4) · [A3 训练契约](/Users/simida/csdiy/11-768-ai-agents/assignment-3/ASSIGNMENT.md:189)

## PULSE · 分开看权重补丁和训练通信

PULSE v1 的稀疏权重补丁在 v2 称为 **PULSESync**。它比较两个 checkpoint 的 BF16 表示，发送变化的下标和新值。接收端覆写这些值并核对 hash。FP32 主权重中的小更新仍可累积，直到改变下一次可见的 BF16 值。约 99% 是特定设置下连续 BF16 表示保持相同的比例。论文中的梯度仍接近稠密。[PULSE v1 §3–4](https://arxiv.org/html/2602.03839v1#S3) · [PULSE v2 §3、§4.1–4.2](https://arxiv.org/html/2602.03839v2#S4)

v2 的 grail 实验在 Qwen2.5 7B 的 MATH 与 MBPP 设置中报告平均上传 108 MB，对照是 14 GB 的完整 BF16 checkpoint。按十进制单位换算约 **130× 载荷缩减**，各次接收端 checksum 验证通过。它衡量传输体积，编码、解码、链路等待与冷启动需要另外计时。若节点错过多个版本，周期性全量 checkpoint 与补丁链可恢复状态。更密集的全量 checkpoint 缩短恢复路径，同时增加存储和传输开销。[PULSE v2 §5、图 6、附录 J](https://arxiv.org/html/2602.03839v2#S5)

v2 另有 **PULSELoCo**，用于 trainer 之间传输 DiLoCo 风格的 FP32 伪梯度。未发送的更新留在误差反馈缓冲区。Qwen2.5-7B 的 8 个本地更新步配置中，编码后的每 worker 载荷为 1.77 GB，对应 DiLoCo 的 30.5 GB，约 **17.2×**。两项数字使用不同传输对象和频率。权重逐位还原的证据适用于 PULSESync。PULSELoCo 的结果是指定配置下最终 MATH 表现接近 DiLoCo，部分早期 checkpoint 较慢。[PULSE v2 §4.3、§5、附录 F.3](https://arxiv.org/html/2602.03839v2#S4.SS3)

## R3 · 相同 checkpoint 还需核对运行时状态

R3 记录推理中每层、每 token 的 top-k expert mask。训练前向复用该 mask，再用训练侧 logits 计算门控权重。它对齐 expert 选择，同时保留梯度。Qwen3-30B-A3B 的 SGLang 采样与 Megatron 重算对照中，估计 KL 从约 0.0015 降至 0.00075。这个指标仍有残差。v3 还增加了多轮软件工程实验。Qwen3-30B-A3B 在 SWE-bench Verified 的单次训练对照中，表内最佳 pass@1 为 31.8 与 38.6，出现于不同训练步。此结果不能直接换算为 Infra 工作成功率。[R3 v3 §3、§4、§5.3.1、附录 E](https://arxiv.org/html/2510.11370v3#S4)

**核对表 4 的分母。** 作者正文把 3.45% 写成吞吐下降。表中 42,321 → 40,910 token/s，对应吞吐下降 **3.33%**。同样 token 数所需时间增加 **3.45%**，原始 10 步墙钟则从 4,820 s 到 4,967 s，增加 **3.05%**。使用这条数据时应注明取的是 token/s、每 token 时间，还是本次运行墙钟。[R3 v3 §5.4.1、表 4](https://arxiv.org/html/2510.11370v3#S5.SS4.SSS1)

## Prefix sharing · 从同一前缀的节省到整条流水线

- **AReaL-DTA** 用 DFS 只保留当前分支的训练状态。必要时分块重算，以少量额外前向换显存。v2 §5.3 的 **8.31×** 来自 τ²-bench 的单卡梯度计算消融，基线为 Dense+CKPT，不含 optimizer 更新。对 Sparse+CKPT 的增益为 1.52–1.70×。8B 的 **2.28×** 是整条异步 RL 流水线在 8 张 H800 上的对照，rollout 与 trainer 的卡数从 2:6 调至 5:3。v2 附录 B.1 中，固定为 2:6 时的 8B 吞吐为 22.6 → 27.5 K tokens/s，约 1.22×。附录 B.2 中前缀共享弱时，一些配置变慢。引用 v2 时用 §5.1 的 GRPO 口径，v1 同处写为 PPO。[AReaL-DTA v2 §5.1–5.3、附录 B](https://arxiv.org/html/2602.00482v2#S5) · [v1 §5.1](https://arxiv.org/html/2602.00482v1#S5.SS1)
- **ZoRRo** 对相同 prompt 的多个响应共享训练计算。作者工程报告图 3 的约 **6×** 是 Arctic-Text2SQL-R2 的 actor update，最大 prompt 16K、生成 4K、每题 16 个 rollout。图 2 的约 **3.5×** 是 32 张 H200 上 VeRL 加整组优化后的 RL iteration 对照，包含训练和生成侧改变。图 1 的 TFLOP/s 按带重复工作的基线计算量折算，可以用来表示节省的工作，不能当作硬件实际吞吐。[Snowflake 工程报告，图 1–3、脚注 1](https://www.snowflake.com/en/blog/engineering/zorro-enterprise-rl-training/)

## 对产物选择的影响

这些论文适合转成**可核验的证据字段和离线评估反例**。先让现有助手和带字段检查的版本分别读同一组证据。可检查的字段包括基线、设备、并行布局、统计分母、运行阶段、依赖关系、策略版本和复核方法。例如，用「同一 checkpoint、路由不一致」检查是否能定位到运行时，用「单卡阶段 8.31×、整条流水线 2.28×」检查是否能守住收益边界。若字段检查能消除误报，后续可以评估脚本或固定工作流的维护成本。若提供完整字段后仍反复误判，再核验人工示范的一致性，并保留未参与设计的新任务评估。三类历史案例目前可用来区分失败原因，仍缺评估 SFT 收益所需的独立数据和模型结果。
