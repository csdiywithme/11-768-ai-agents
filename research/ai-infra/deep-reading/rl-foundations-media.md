# L9 · 策略梯度、学习信号与录像范围

### `https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf`

- 原文：[L9 · RL Foundations for Agents](https://www.cmu-agents.com/slides/lecture-09-rl-basics.pdf)
- 范围：46 页文字，重点 pp.2–6、9–16、18–24、25–41、44–45。PDF SHA-256 为 `c8724ca6fbadb2afef38d627bbab2e09f8e8077fb1bb4dc0a4a078fb8669c1ca`。
- 方法与限制：找隐藏整数的例子说明示范动作、真实目标、观察不到的环境状态和模型自生错误轨迹会错开。策略梯度仅对策略生成的动作算分，奖励到未来用于按步调整。组内完全一致的奖励使中心化优势为零。pp.31–33 以两臂例比较 baseline 对梯度方差的影响。
- 工程用法：选 profiler 的任务应把“工具是否可用”“现有证据能否区分瓶颈”“输出是否真的触发下一步”写入状态。若同组都失败，检查可达性和 verifier 再增加训练轮次。先评人工正确动作和无训练基线，按错误归属决定处理输入、工具、规则或示范。

### `https://cmu-l3.github.io/anlp-spring2026/static_files/anlp-s2026-16-rl.pdf`

- 原文：[CMU ANLP Spring 2026 L16 · Reinforcement Learning](https://cmu-l3.github.io/anlp-spring2026/static_files/anlp-s2026-16-rl.pdf)
- 范围：50 页官方 PDF。重点 pp.19–25、28–34、36–43、48–50。SHA-256 为 `92e174a97b0aa36a0222c49bb9f303d33d73bf75360a986c725945f7e352ee48`。
- 方法与限制：从环境状态与观测的差别推到轨迹似然，再用 score function 把不可微的环境反馈放到策略更新。CartPole 作示例。后段分开讨论如何估计动作权重以及如何控制更新步长。
- 工程用法：把真实执行回执存成不可由候选自己定义的观察，再判断要修改反馈、分配信用或限制策略变化。调试时先用可手算的两步轨迹核对动作 mask、reward-to-go、baseline。若外部状态变了但未记录，不把无法解释的奖励波动归为梯度算法差异。

### `https://doi.org/10.1007/BF00992696`

- 原文：[Springer 正式 DOI](https://doi.org/10.1007/BF00992696) · [正式出版页扫描件](https://gwern.net/doc/reinforcement-learning/model-free/1992-williams.pdf)
- 范围：出版信息、PDF 定理 1、延迟反馈讨论、§8.3 与附录 A。出版入口给摘要，正文来自同刊同页码的扫描件。
- 方法与限制：权重增量含奖励减 baseline 与动作 log-likelihood 导数。baseline 在指定条件下与该随机输出独立，期望梯度性质来自该条件。定理不规定哪种 baseline 方差最小，也不能保证每次更新改进。
- 工程用法：A3 先做 1–2 个可枚举决策的确定性梯度检查。观察同组中心化、样本自包含与其他 baseline 的区别，记录采样策略及版本。再用多 seed 看学习曲线和样本方差，不从一次最终 reward 推出估计更好。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-09/`

- 原文：[L9 第三方英文字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-09/)
- 范围：英文 01:03:40–01:10:03、01:13:46–01:17:09 选段，核对官方课件 pp.34–45。未把译稿当官方逐字稿。
- 状态：字幕文本已读
- 方法与限制：口头解释同组全 0 或全 1 缺相对信号，可换题、补示范、改善观测或反馈。哪一项有效取决于失败机制。讲师强调此处 GRPO 是教学简式。
- 工程用法：保留验证失败、环境启动失败、工具权限缺失、无候选通过及真实通过。比较各类比例变化后再决定过滤、改任务或训练。不能以丢弃大量失败组制造表面成功率增长。

### `youtube:paAcPaaYZGM`

- 原文：[L9 官方录像](https://www.youtube.com/watch?v=paAcPaaYZGM)
- 范围：课程入口与视频标识核对。网页工具未返回连续画面或音轨；同讲英文字幕按另一条来源记载。
- 状态：原件受限
- 工程用法：有关 R、baseline 和 DrGRPO 的公式以原 PDF 与原论文为准。字幕对口头解释有帮助，但需用时间和课件校验术语。

### `https://www.youtube.com/playlist?list=PLSN0qpDfUvTM`

- 原文：[Graham Neubig · CMU 11-768 课程播放列表](https://www.youtube.com/playlist?list=PLSN0qpDfUvTM)
- 范围：当前网页标题及 2026-11-12 缓存的 playlist 条目。课程 L1–L14 的登记入口可在本地覆盖表复查。
- 状态：仅核对入口
- 工程用法：按讲次记录实际可播放的原件和确实看过的时间段。只出现在播放列表不代表内容已读。 [官方静帧与访问记录](../coverage/recording-recheck.md)
