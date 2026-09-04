# Evidence First 与质量门

## 证据规则

每条关键证据记录：项目/feature、工件路径及版本或指纹、动作、执行环境、时间、实际结果、限制和对应标准。日志只保存必要摘要并脱敏，不复制凭据或敏感业务数据。

代码/规格变更后检查旧证据是否失效。测试未运行写 NOT_RUN，工具/环境不可用写 BLOCKED，确实不适用写 N/A 并说明理由；不能把这些结果写成 PASS。命令成功只是证据之一。

Task Evidence、Test Evidence、Review Evidence、Acceptance Evidence 必须绑定原 Task Identity 和当前输入版本。退出码 0 不能单独证明 Task Completed；测试通过不能单独证明 Product Accepted。

下表是正式路线的门。轻量路线只按工作流路由预先明确的适用性裁剪；正式路线的 Spec/Plan/Tasks 不能因为缺失而记 N/A。实施前检查 `SPEC_KIT_REQUIRED_GATE` 与 `HUMAN_GATE`；任一阻塞都禁止依赖实施。最终状态逐项按产品验收参考的 `ACCEPTANCE_PASS_GATE` 计算，不由此表中的单个门替代。

下表是 PE 的准入/出口检查，不实现官方阶段；执行动作均遵循当次加载的官方 Skill。每阶段均有 Input、Action、Expected Output、Validation、Failure Handling。

| 阶段/门 | Input | Action | Expected Output | Validation | Failure Handling |
|---|---|---|---|---|---|
| Intake | 用户请求 | 确定目标与终点 | 任务范围 | 范围与权限可区分 | 先调查，关键业务歧义按自主规则处理 |
| Discovery | 仓库与规范 | 只读调查 | 技术事实和未完成工作 | 引用实际文件，不猜技术栈 | 缺失与 UNKNOWN 分开 |
| Authorization | 范围、动作 | 风险分级 | 当前允许动作 | 不借全流程扩大权限 | 停止未经授权动作 |
| Brief | 目标与事实 | 整理初始产品信息 | Brief 或会话摘要 | 目标、非范围、验收信号清楚 | 不制造平行规格 |
| Capability | 已绑定路由、本地工具/状态 | 集成参考中的探测与 SPEC_KIT_REQUIRED_GATE | 能力快照及门结论 | 所需官方能力可用；阶段输入/副作用明确 | 原任务 BLOCKED；不得修改业务源码/测试或绕过门的配置 |
| Constitution | 原则与既有治理 | 复用或官方更新 | 有效 constitution | 不是未填模板；不增设未经确认原则 | 治理冲突不靠削弱规则解决 |
| Specification Gate | 产品意图 | 官方 specify；按需 clarify | 有效 spec 与质量结果 | 用户故事、边界、可验证标准；关键歧义已解决 | 回官方规格流程，不直接写业务实现 |
| Clarify | spec、查证答案 | 官方澄清能力 | 更新的正式规格 | 已知答案不重复问；剩余阻塞明确 | 不用 Agent 推断伪装用户答案 |
| Plan Gate | spec、架构事实 | 官方 plan | 有效 plan 与适用设计工件 | 对应需求，复用约定，风险与验证可落地 | 回计划阶段；不要求无关工件凑数 |
| Task Gate | spec、plan | 官方 tasks | tasks 与依赖 | 需求有实现/验证任务；保留既有历史 | 不私造另一份执行 backlog |
| Analyze | spec、plan、tasks | 官方 analyze | 一致性报告 | 无阻塞矛盾；核心分析只读，hooks另核验 | 按官方批准边界修复并复查 |
| Implementation Gate | 有效 tasks、plan | 官方 implement | 代码、测试、配置与完成状态 | 任务勾选、实际代码、diff、规格符合度一致 | 修复后重验；不凭勾选通过 |
| Test Gate | 当前代码、验证标准 | 先窄后广测试 | 可复核结果 | 适用的单元/集成/回归、lint、typecheck、build；按产品需要运行 UI/API 场景 | 必需验证失败为 FAIL，无法运行为 BLOCKED |
| Converge | 已实施代码和正式工件 | 官方 converge | 收敛结论或追加任务 | 基线仅追加任务，不改 spec/plan、不改旧任务；无缺口 tasks 不变 | 新任务回 Implement，再验证 |
| Review Gate | 当前 diff、正式工件、测试 | 独立审查步骤 | review-report | 下述审查维度均被考虑；阻塞发现已复查 | 修复后重跑相关测试与审查 |
| Acceptance Gate | 原 Task Identity、产品标准与当前证据 | 逐项检查 ACCEPTANCE_PASS_GATE | acceptance-report | 14 项必需条件满足；正式证据与原目标一致 | 缺必需证据禁止 PASS/COMPLETED；按验收四态保留失败/阻塞 |
| Completion | 全部必需门结果 | 汇总并核对 | 最终报告 | 只有全门通过且 Acceptance PASS 才 COMPLETED | 明确未完成阶段、证据与下一步 |

## 验证选择

从仓库实际脚本与 CI 找命令，先核验副作用。依次扩大：修改单元 → 相邻集成/回归 → 适用静态检查和构建 → 关键用户场景。验证范围与变更风险相称，不要求所有项目具备所有工具。

对实现必须检查实际源码与 Git diff（包括未跟踪文件）；无 Git 时使用实际文件变更证据，不因此假称无变更。必须运行的验证因环境受限无法执行，记录缺口而不是跳过后宣称通过。

## 独立 Code Review

在 Implement/Converge 后单独进入审查角色，重新读取当前正式工件、实际变更和证据；不能沿用实现阶段的自述作为结论。独立是独立判断步骤，不要求创建第二个 Agent 或平台。

逐项检查：Spec 符合度、Plan 符合度、Tasks 完整性、遗漏、无关修改、回归、安全、测试缺口、维护风险、项目规范。测试通过不能替代审查。

使用 [review-report](../templates/review-report.md)。发现应含位置、影响、严重性、证据及修复建议；没有发现也写明检查范围与局限，不能声称无任何缺陷。决策为 PASS、CHANGES_REQUIRED 或 BLOCKED。后续修复若改变代码，相关测试和审查证据必须更新。

## 本 Skill 的静态自审规则

Skill 维护时检查：frontmatter/name/description 合规、明确触发边界、引用可达、渐进加载、四模板可用；无重复官方命令/模板、无绝对本机路径绑定、无版本唯一许可、无授权扩张、UNKNOWN 不被吞掉、Review 与 Acceptance 独立、完成条件一致。

用反例走查：只读请求不得实现；旧代码不得覆盖明确新要求；缺官方能力不得仿造；feature 歧义不得猜；测试成功但标准失败不得完成；风险接受不得等同 PASS；中断不得盲目重放。静态规则通过不证明真实 Agent 行为或 Spec Kit 端到端可用。
