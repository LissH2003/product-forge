# 工作流路由

## Repository Discovery

只在用户指定仓库和相关路径内调查。先读适用的 AGENTS.md、README、项目规范，再检查 Git 状态、已有工作和下列信号；不要广扫用户主目录、会话记录或凭据。

| 识别项 | 证据示例 |
|---|---|
| 项目类型、语言、Framework | 目录、源码入口、依赖声明及锁文件 |
| Package Manager、Build Tool | 锁文件、项目脚本、Makefile、构建配置 |
| Test Framework、Linter、Formatter、Type Checker | 配置、脚本、测试目录及 CI 实际调用 |
| CI、Docker、Deployment configuration | CI 配置、Dockerfile、compose、部署声明；只读，不能据此部署 |
| 项目级约束 | AGENTS.md、README、架构决策、贡献规范与用户要求 |
| Spec Kit 与现有 feature | 依集成参考验证元数据、工件、实际 Skills |
| 未完成工作 | 未提交/未跟踪文件、未完成任务、失败验证、已有运行记录 |

未找到记为 NOT_FOUND；无法读取或解释记为 UNKNOWN，不混同“不存在”。事实调查顺序优先仓库证据，但授权和目标来自用户与适用指令：现有实现不是业务正确性的天然证明。命令从配置中发现后先检查副作用，不能直接执行未知的 test/build 脚本。

## 场景路由

Discovery 后、任何实施前，在会话或现有 PE 记录中绑定：Task Identity、用户原目标、授权范围、任务类型、`spec_kit_required`（true/false）、判定证据、必需阶段/验收证据。它不是新的 Spec Kit 任务清单。

| 任务类型 | Spec Kit 判定与路线 |
|---|---|
| NEW_PRODUCT | 默认 true，正式路线 |
| NEW_COMPLEX_FEATURE | 默认 true；V1 新功能（含新增语言偏好、缓存能力）使用此正式路线，不因代码量小改成维护 |
| MIGRATION | 默认 true，正式路线 |
| HIGH_RISK_CHANGE | 默认 true，并独立检查 HUMAN_GATE / 风险授权 |
| BUG_FIX | 已知预期行为、范围局部、无新增关键业务语义/高风险/强制 SDD 约束时，可明确 false，轻量修复 |
| SMALL_REFACTOR | 保持行为、范围局部、无高风险/强制 SDD 约束时，可明确 false，轻量重构 |
| MAINTENANCE | 不新增产品行为、无关键权限/数据语义变化且无强制 SDD 约束时，可明确 false，轻量维护 |
| INVESTIGATION | 只读调查可明确 false，不授权实施或宣称整个产品完成 |

已有规范、用户明确要求正式 SDD、需要更新正式工件，或风险升级时，即使标签是 BUG_FIX/SMALL_REFACTOR，也必须 true。接续任务复核并保留原路线；原路线不明时调查，不默认轻量。判定尚未知且影响实施时原任务 BLOCKED。

路由一旦确定依赖 Spec Kit，不能因能力缺失、测试可跑、用户已授权开发或自认为简单而降级。只有目标/约束确有新证据变化才重新路由，记录原因；工具缺失本身不是重路由依据。

轻量路线：Discovery → 明确路由和证据适用性 → 风险门 → 复现/调查 → 授权内局部实施（调查不实施）→ 必需测试 → 独立 Review → 限定该任务范围的 Acceptance。正式 Spec/Plan/Tasks 不适用时须在实施前记录理由；不是创建替代正式规格，也不是验收时豁免缺失文件。

以下为需要 Spec Kit 时的正式路线；阶段名引用入口生命周期，执行细节由官方 Skills 提供。

| 场景 | 起点与复用 | 需要的后续阶段 | 不能做 |
|---|---|---|---|
| New Product | Intake → Discovery；空仓库不猜技术栈事实 | Brief → Capability Detection → 全部适用阶段 | 不自动初始化 Spec Kit，不自行搭建替代规格体系 |
| New Feature | 先确认与现有 feature 不重复，复用 constitution | Specify → 按需 Clarify → Plan → Tasks → Analyze → Implement → Converge → Review → Acceptance | 不重写无关 feature |
| Bug Fix | 先复现、定位症状与预期行为；复用有效关联规格 | 用官方能力补齐受影响规格/计划/任务，随后 Analyze 到 Acceptance；回归测试必需 | 不把当前错误代码当正确需求；普通局部修复不自动套完整流程 |
| Refactor | 明确必须保持的行为和兼容约束 | 复用/更新对应官方工件，再 Analyze 到 Acceptance | 不顺带改变业务行为或做无关重构 |
| Migration | 先检查数据、API、运行环境与回退边界 | 官方规格链路 + 迁移验证 + Review + Acceptance | 本地设计与测试不授权生产迁移 |
| Existing Feature Continue | Discovery → Capability Detection → 核验既有工件与中断证据 | 从最早失效/未完成阶段恢复，然后 Review、Acceptance | 不按编号或时间猜目标，不清空 tasks 历史 |

Bug Fix 的核心路由不依赖可选 bug extension；若项目已有该扩展，先按集成契约核验，再使用其官方能力，不自行仿制它。

## 状态、回流与记录

### TASK_BINDING：最小任务状态

在会话或现有 PE 记录中使用 PENDING、IN_PROGRESS、BLOCKED、COMPLETED、CANCELLED；不新增状态存储或 Workflow Engine，不修改官方状态。

| 转移 | 必需证据/限制 |
|---|---|
| PENDING → IN_PROGRESS | 已绑定目标/范围并开始调查；不代表实施 Gate 已通过 |
| PENDING/IN_PROGRESS → BLOCKED | 影响原目标的 UNKNOWN、缺少必需能力/证据或未获关键确认；UNKNOWN 是阻塞原因而非第六种任务状态 |
| BLOCKED → IN_PROGRESS | 原阻塞解除的证据及重新核验的门；“继续”或另一任务成功不足以解除 |
| IN_PROGRESS → COMPLETED | 同一 Task Identity 的 Acceptance PASS 与全部必需证据，依产品验收参考判定 |
| 未完成状态 → CANCELLED | 用户明确取消/替换该目标；技术阻塞不能代替用户取消 |

原目标 → Task Identity → Task/Test/Review/Acceptance Evidence 必须可追溯。发现完成证据失效时撤回完成判断并回到受影响状态；关键 UNKNOWN 仍在则 BLOCKED。不得 UNKNOWN → COMPLETED。

无关任务必须独立标识、独立授权和验收，不覆盖原任务状态；其成功不能解除原任务 BLOCKED。最终答复先报告用户原任务状态，再说明额外发现。详见自主决策参考的 UNKNOWN 边界。

- 产品需求明确时不要强制用户逐阶段批准；采用自主决策参考中的权限边界。
- 复用阶段要记录工件路径、当前版本/指纹、适用范围和复用依据；文件存在不等于质量门通过。
- 规格或业务目标变更：重新检查 Plan、Tasks、Analyze 和验收映射。实现变更：重跑相关验证和 Review。
- 技术修复走当前授权范围；业务语义改变不能借 converge 偷偷扩大需求。
- 一个 checkout 中一次只绑定一个 feature 写流程。发现并行写者时暂停相冲突操作，不自建锁服务或多 Agent 平台。
- 用户中途停止后不启动后台继续；再收到接续请求时重新核验环境和现场。

Brief 使用 [product-brief](../templates/product-brief.md)。正式 spec 建立后，Brief 只保留目标背景与工件引用，不再维护平行的功能要求。决策使用 [decision-record](../templates/decision-record.md)。

模板输出优先进入项目已约定的文档位置；没有约定时，可在开发授权内使用 `.product-engineering/`，按明确 feature/运行标识隔离。只持久化目标、引用、证据和决策，不复制 spec/tasks；只读请求输出到会话。不得用此记录替代或伪造 Spec Kit 运行状态。
