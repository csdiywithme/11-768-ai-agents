# 外部原文 · 书目、服务基线与人机协作

### `https://aclanthology.org/2026.acl-long.1023.pdf`

- 原文：[BrowseComp-Plus · ACL 2026 正式 PDF](https://aclanthology.org/2026.acl-long.1023.pdf)
- 范围：§§3.2–3.4、4 与评估表的固定语料、hard negatives、检索与最终答案。与预印本阅读记录交叉核对。
- 方法与证据：用有证据覆盖的固定文档库替代不透明的搜索 API，并加入容易误取的负例。可把检索是否提供必要证据和模型是否据此解决分开。在开放互联网有新版本或没收录的项目时需要另外评价。
- 工程用法：将同仓不同 commit、同算子不同 GPU、相似 error log 和正确产物一起放入冻结语料。依次测证据获取、条件核对、动作和最后结果。再用实时搜索验证版本更新能力。

### `https://pmc.ncbi.nlm.nih.gov/articles/PMC12935541/`

- 原文：[OpenScholar 的 PMC 文章页](https://pmc.ncbi.nlm.nih.gov/articles/PMC12935541/) · [Nature 正式 PDF](https://www.nature.com/articles/s41586-025-10072-4.pdf) · [Europe PMC 同文 JATS](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12935541/fullTextXML)
- 范围：PMC 页面在本轮返回机器人检查，实际读 Nature PDF 与 Europe PMC JATS 的语料、检索、局限。
- 状态：替代版本精读
- 方法与证据：正式论文主要评估用 peS2o v2。发布的数据存储为 v3，约 4,500 万论文。标注对证据截止日期、专家覆盖和句级引用有边界。
- 工程用法：每条建议分别记录代码、文档、运行和检索日期。实际评估冻结语料快照，公开发现和匹配证据的分母。PMC 这个 UI 是否成功访问不影响同篇原文的证据，不重复增加一份独立研究。

### `https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12935541/fullTextXML`

- 原文：[Europe PMC · OpenScholar JATS 全文](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12935541/fullTextXML)
- 范围：官方 XML 中 Methods 的语料与三种检索来源、Limitations。下载 SHA-256 为 `24d397e26283d8943ceda9b005924495574161203408951af34ec8e384088fa0`。
- 方法与证据：检索结合数据存储、Semantic Scholar 摘要与网页来源。数据存储的更新日期只约束这一路。作者指出某些问题没有取到最具代表性的论文，写作质量偏好和引用证据需要分别评价。
- 工程用法：诊断源码时记录精确 commit 与用户运行环境。若生成证据和源文件引用很强但抓错版本，将任务记错并保留版本筛选的反例。此 XML 与 Nature PDF 为同一正式出版物。

### `https://proceedings.mlr.press/v306/chen26da.html`

- 原文：[PMLR 306 · How can we assess human-agent interactions?](https://proceedings.mlr.press/v306/chen26da.html) · [PMLR 正式 PDF](https://raw.githubusercontent.com/mlresearch/v306/main/assets/chen26da/chen26da.pdf)
- 范围：§§3–4、6.2–7、图 4。PDF SHA-256 `28f7ef2bac95e46366d650a8fcce01b27b4e182b12a05db82b5d42c039dea5b1`。
- 方法与证据：在 15k 用户、36k 会话中有 1,747 条真实评级轨迹。使用预测标签增加信息，再用真实标注校正预测误差，假设有标和无标属于相同底层分布。四个实验置信区间平均缩窄 39.5%。这是一项估计精度结果，用户选择评分会影响假设。
- 工程用法：先收真实 AI Infra 审阅、接受与返工信息。将同种任务按实际难度和用户分层。要用模型估人工负担，先检查标注样本和缺标样本的差异，再用留出人评纠偏。没有足够样本时直接报告原始工作时间和分歧。

### `https://metr.org/blog/2026-02-24-uplift-update/`

- 原文：[METR · We are Changing our Developer Productivity Experiment Design](https://metr.org/blog/2026-02-24-uplift-update/)
- 范围：招募、选题、退出、质量与并发工作对第二轮测量的影响。
- 方法与证据：开发者是否愿意加入以及是否愿意提交需要不用 AI 完成的任务，可能随真实收益改变。等待期间处理别的任务时，单项墙钟与主动用时分离。作者认为新研究中央估计不能可靠反映当前提升。
- 工程用法：记录未进入、未完成和换题的分母，分开测总完成、主动操作、等待、返工和产物质量。算子与框架的人工基线先用真实抽样和匹配题，别只筛已有优胜案例。

### `https://valeriechen.github.io/`

- 原文：[Valerie Chen 研究主页](https://valeriechen.github.io/)
- 范围：2026-11-13 的研究方向与 Publications，查 PULSE、RECAP、Code with Me or for Me 和 Beyond the Commit。
- 状态：仅核对入口
- 工程用法：按开发者真实时间、协作过程和可复查产物找作者论文。主页是来源定位，实证条件归各原文。 [PULSE 正式论文](https://proceedings.mlr.press/v306/chen26da.html)

### `https://docs.vllm.ai/en/latest/benchmarking/`

- 原文：[vLLM · Benchmark Suites](https://docs.vllm.ai/en/latest/benchmarking/)
- 范围：CLI、parameter sweeps、dashboard 与 recipe conversion。2026-11-13。
- 方法与限制：区分服务负载、forward/latency、离线吞吐和启动。可生成 recipe 配置或参数扫描。入口提供可用工具，不提供某个场景的最优结果。
- 工程用法：给 workload 问题先选择对应 suite，把实际用户到达和 SLO 与离线跑分分开。若想验证候选建议，只用可复现配置，在选参预算相同下对比已有 sweep。

### `https://docs.vllm.ai/en/latest/benchmarking/dashboard/`

- 原文：[vLLM · Performance Dashboard](https://docs.vllm.ai/en/latest/benchmarking/dashboard/)
- 范围：CI 图表、手动运行、commit 镜像、结果文件与 suite 的变量。2026-11-13。
- 方法与限制：面板用固定的 CI 测试套件追踪跨提交趋势，产物含 Markdown 与 JSON。不同 CPU/GPU、配置或数据条件可能改变趋势。
- 工程用法：先从公开 dashboard 找到要查的版本范围，固定镜像、workload 和测试配置复跑指定版本。对目标部署保留独立服务指标。只有公开曲线且没有同条件本地实验时，写为上游证据。

### `https://docs.vllm.ai/en/latest/contributing/`

- 原文：[vLLM · Contributing](https://docs.vllm.ai/en/latest/contributing/)
- 范围：开发流程、测试限制、PR 与 AI-assisted contributions。2026-11-13。
- 方法与限制：项目要求人工理解并复查 AI 辅助改动、运行相关测试，部分测试依赖 GPU 或 CI。上游模板规定的是交付要求，没有给出本地修改一定成立的保证。
- 工程用法：提交候选前收集真实问题、固定 diff、失败与成功的运行、硬件和局限。先验证是否已有修复或相同 issue，再判断自有实验是否提供新证据。未跑平台相关测试就明确写状态。

### `https://docs.vllm.ai/en/latest/usage/metrics/`

- 原文：[vLLM · Production Metrics](https://docs.vllm.ai/en/latest/usage/metrics/)
- 范围：General Metrics、speculative decoding、histogram families 与 bucket cardinality。2026-11-13。
- 方法与限制：waiting reason 区分 capacity 与 deferred。抢占、latency、TTFT、ITL 和 KV 采样各有分母和更新点。为每标签与副本增加 buckets 会扩大时序数。文档 latest 需与部署实际 exposition 对齐。
- 工程用法：容量与临时延后分别出图。按实际 SLO 设置有限 buckets，检查计时范围、请求完成和 preemption 的对应。聚合指标出现变化时用少量 request trace 验证机制。
