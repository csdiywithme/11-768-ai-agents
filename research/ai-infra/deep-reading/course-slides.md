# L2 与 L20 · 工具调用和搜索的课件阅读

### `https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf`

- 原文：[L2 官方讲义 · Tool Use for Language Model Agents](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf)
- 范围：扫描 60 页抽取文本，细读 pp.13–17、19–25、27–35、38–46、48–57。与原论文和当前协议对照。缓存 PDF SHA-256 为 `671b9d0c849150aa9b5d95a9655b892dbe5fa2d6fc7a6c704c0f8e3f876e74ad`。未逐图核对。
- 方法与限制：模型发出结构化动作，工具层执行并带标识返回。约束解码解决语法，业务仍可能错。独立动作可并行，依赖与副作用决定顺序。p.48–49 的 6.4 和 2.8 秒是示例预算。p.54、56 分别检查选择、参数、轨迹、目标和系统故障。
- 工程用法：对同一 GPU 作业同时读取独立日志和配置可以并行。确定目标 run ID 后才能解析匹配产物。把实际版本、ID、单位和观测缺失写进结果。用格式合法但读取错 run 的反例评估是否真的接上业务检查。

### `https://www.cmu-agents.com/slides/lecture-20-search-basics.pdf`

- 原文：[L20 官方讲义 · Search over Trajectories](https://www.cmu-agents.com/slides/lecture-20-search-basics.pdf)
- 范围：官网 URL 本次网页阅读未展开，使用前次取得并校验的官方 PDF。扫描 31 页抽取文字，细读 pp.2–15、17–25、27–30，结合 p.15、28 图的前次检查。SHA-256 为 `7ef0f29b2c1067258914c786f088326013f423c17a284ffe33069c7983a38c0f`。
- 方法与限制：独立采样增加覆盖，最终结果取决于筛选器。反馈修订可继承证据，也可能继承错误。p.15 的 76% 上限属于给定接受率与首个通过策略的例子。p.30 的分叉可用来定位是候选不足还是选择出错。
- 工程用法：比较 B300 候选时，先记录实际可行的假设数、能完成的正确性和计时、选择结果和费用。先检查是否能区分顺序效应，再开更多改动。若加候选只提高 oracle 覆盖而最终排序未改善，先修测量和筛选。 [L20 原文对照](search-course-readings.md)

### `https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf`

- 原文：[L11 · Advanced RL Algorithms for Agents](https://www.cmu-agents.com/slides/lecture-11-rl-advanced.pdf)
- 范围：58 页文字与关键例子。重点 pp.8–17、23–35、38–48、49–53。PDF SHA-256 为 `e2f23de9092424cde3a72805e3b343d261a2e219518871843e719c4dbe5cc968`。
- 方法与限制：同一重试修复问题连接回报、critic、GAE、组均值、staleness、重要性比率与测试盲区。示例中一条轨迹里的每步共享同一组权重，会鼓励成功中不好的动作，也会压低失败中有价值的动作。故障测试和高通过率分别无法判断归因。
- 工程用法：先判明同一任务全组失败来自环境、无法到达、没有多样性或 verifier。恢复、计划、算子验证分别保留可见状态和真实结果。组奖励可以用于最终成绩，下一步行为单用能判定分歧的样本。 [课程书目中的评估审计](evaluation-validity.md)

### `https://www.cmu-agents.com/slides/lecture-12-rl-systems.pdf`

- 原文：[L12 · Reinforcement Learning Systems](https://www.cmu-agents.com/slides/lecture-12-rl-systems.pdf)
- 范围：43 页文字与关键流程图。重点 pp.5–17、19–27、30–41、43。PDF SHA-256 为 `15a062df1513c0710e4e3bd8cfa247d4b131b966faadc32e5101561c322a8a13`。
- 方法与限制：把 sample、verify、重算 logprobs、反向、optimizer 与权重同步分开，并区分 rollout 的 decode 与训练批的计算。前缀共享、recompute、拓扑、版本、在线 RL 与多租户针对不同负载。末页给出的指标是定位线索。
- 工程用法：A3 或本地推理训练调优先做阶段时间线，标出 overlap、GPU 空闲、工具排队、logprob mismatch 和权重传播。挑可改变关键路径的项。同步更新与中途换权重各自建版本核对，不能按总 GPU 利用率断言训练改进。 [原文对照与版本归属](training-systems-02.md)

### `https://cmu-agents.vercel.app/slides/lecture-06-coding-agents.pdf`

- 原文：[L11 p.3 所引课件地址](https://cmu-agents.vercel.app/slides/lecture-06-coding-agents.pdf) · [现课程官网同讲次 PDF](https://www.cmu-agents.com/slides/lecture-06-coding-agents.pdf)
- 范围：旧域名本轮无法从网页工具取得原件，读取现官网 82 页 L6 对应正文、p.47 harness 和 p.58 修复例子，核对 L11 p.3 的入口来源。
- 状态：替代版本精读
- 方法与限制：L11 用最小修复和四组输入演示 verifier。L6 把 issue、代码库、测试环境与测试结果串起来。旧链接的文件字节和页码未能与现官网逐一比较。
- 工程用法：组装真实性能任务时同时存题目输入、运行前状态、相同环境的真实回执和独立测试。旧示例可解释评分思路，定位正式图表时使用现 PDF 的页码。 [L6 逐项笔记](search-09.md)

### `youtube:jKx7_fOZRzg`

- 原文：[L11 讲师公开录像](https://www.youtube.com/watch?v=jKx7_fOZRzg&t=1440s)
- 范围：回查播放列表记录和历史已见的 24:00、24:20 官方 storyboard。当前网页工具未取得连续音轨、画面或官方字幕。
- 状态：原件受限
- 工程用法：可确认录像示范同步等最慢任务的结构。异步的数量对照和训练稳定性应回到 L11 正文及 AReaL 消融。 [先前静帧证据](../coverage/recording-recheck.md)

### `youtube:dwR6NP9nI-8`

- 原文：[L12 讲师公开录像](https://www.youtube.com/watch?v=dwR6NP9nI-8&t=1850s)
- 范围：回查播放列表记录和历史已见的 30:50 官方 storyboard。当前网页工具未取得连续音轨、画面或官方字幕。
- 状态：原件受限
- 工程用法：静帧确认课程将 full sync、delta sync 与推理分片并列。跑实验时分别留 payload bytes、传输、接收侧加载和端到端阻塞。 [先前静帧证据](../coverage/recording-recheck.md)

### `https://mcp.example/mcp`

- 原文：[L2 p.45 · MCP client 示例](https://www.cmu-agents.com/slides/lecture-02-tool-use.pdf#page=45)
- 范围：页内 Client 初始化与凭据中介示意，回查 pp.43–46。登记地址为教学占位符。
- 状态：示例已核
- 工程用法：在实际实验的工具合同中分别记录 agent 到 broker 和 broker 到上游的作用域、工具及回执。用测试配置验证调用可用，再开真实部署对照。占位符不进生产连通性清单。

### `youtube:jXChFB4JSyw`

- 原文：[L2 官方录像](https://www.youtube.com/watch?v=jXChFB4JSyw)
- 范围：课表和播放列表确认；播放页访问受限。未取得音轨、连续画面或官方字幕。
- 状态：原件受限
- 工程用法：课件与第三方字幕能核对工具格式、调度和权限的教学内容，不能据此声称看过整段录像。

### `https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-02/`

- 原文：[L2 第三方英文字幕](https://romanrose.github.io/llm_learn/generated/courses/cmu-11768-fall26/lecture-02/)
- 范围：31:42–34:27 的生成中约束与长度截断，39:00–42:51 的接口与凭据边界。与 PDF pp.31–46 对照。
- 状态：字幕文本已读
- 方法与限制：生成经过有效文法前缀仍可在闭合前被 token 上限截断。MCP 本身没有自动保证工具权限适当，课件的上游凭据由 server 持有。字幕是第三方文本。
- 工程用法：采集参数语法有效、完成解析、状态合法、远端接受四种结果，分别判断恢复。若日志显示提前停止且对象尚未闭合，不执行半条命令。
