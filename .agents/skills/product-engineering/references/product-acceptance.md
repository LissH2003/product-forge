# 产品验收与完成判定

产品验收由 PE 独立进行，不由 Spec Kit workflow 或任务状态代替。

## 输入与映射

先核对原 Task Identity、用户目标及实施前确定的路线/必需证据。轻量路线可以没有不适用的正式 Spec/Plan/Tasks，但须有路由理由与该限定任务的要求、行动/完成证据；这不是正式产品流程的替代工件。正式路线缺文件绝不能在验收时改为轻量或 N/A。只读调查的验收对象是调查交付物，不能宣称实现或整个产品完成。

按路线读取所需的当前有效 spec、plan、tasks、实际实现、验证结果、独立 Review 和已确认决策。Brief 仅提供背景；正式要求以项目当前权威规格及明确用户变更为依据。正式路线中发现用户变更尚未纳入规格，先回规格阶段，不创建另一份要求列表来绕过。

使用 [acceptance-report](../templates/acceptance-report.md) 建立：产品目标 → User Story/Acceptance Criteria 的真实引用 → 实现位置 → 测试/运行证据 → 状态。优先使用已有标准 ID；无 ID 时引用原文段落位置，不伪造已有编号或复制全部规格。

逐项判断用户能否实现目标，包含失败路径、权限、边界及兼容性等已要求行为。build 成功不能证明业务可用；UI 产品必要时检查实际交互，API 产品检查实际契约/错误路径。只读调研、文档阶段可以交付阶段报告，但不能据此宣布整个产品完成。

## 状态语义

### ACCEPTANCE_PASS_GATE

`ACCEPTANCE_PASS` 是最终状态 PASS 的产生条件，不是新增的第五种状态。逐项记录以下检查的证据/不适用依据，不允许空白、猜测或用无关任务证据代替：

| # | 必需条件 |
|---|---|
| 1 | 所需 Specification 已存在 |
| 2 | Specification 对当前目标状态有效，不是空模板或过期要求 |
| 3 | 所需 Plan 已存在并适用于本次目标 |
| 4 | 所需 Tasks 已存在 |
| 5 | 必需 Tasks 已实际完成，不能只凭勾选 |
| 6 | 授权范围的 Implementation 已完成；纯调查则核对其限定交付物 |
| 7 | 必需测试已执行，有当前版本的运行证据 |
| 8 | 测试结果满足要求，不能把跳过/未运行当通过 |
| 9 | 独立 Code Review 已完成（调查交付物也须独立复核） |
| 10 | 没有未解除的阻塞性 Review Finding |
| 11 | 每个 Acceptance Criterion 均有对应 Evidence，属于原目标 |
| 12 | 没有未解除的关键 UNKNOWN |
| 13 | 没有未确认的关键业务决策 |
| 14 | 没有违反 Risk Gate，所需授权与前置门均满足 |

1–8 中确实不适用的项仅可依据实施前明确的轻量/调查路线说明理由；该说明不等于伪造 PASS。9–14 不能用“任务简单”免除。正式实施路线的 Spec + Plan + Tasks + Implementation + Tests + Review + Acceptance Criteria Evidence 必须完整成立。任何必要条件不满足，`ACCEPTANCE_PASS = 禁止`，不得 PASS、不得 COMPLETED。

Tests Passing ≠ Product Acceptance Passing；Code Review Passing ≠ Product Acceptance Passing；Implementation Finished ≠ Product Acceptance Passing；Spec Kit Workflow Finished ≠ Product Acceptance Passing。不能用“已完成该功能”之类措辞掩盖最终 FAIL/BLOCKED。

| Final Decision | 条件 |
|---|---|
| PASS | ACCEPTANCE_PASS_GATE 的全部必需条件有证据成立；如规定人工签收，已取得 |
| FAIL | 现有证据证明至少一项必需标准不满足 |
| BLOCKED | 证据、能力、环境、关键业务决定或必要签收不足以完成判断；无已证明失败时使用 |
| ACCEPTED_WITH_RISK | 用户明确接受具体、非禁止豁免的已知偏差，批准记录、范围、影响和后续处置齐全；不是 Agent 自行接受 |

逐项原始失败/阻塞结果仍保留；不得因风险批准而涂改成 PASS。若有未获接受的已证实失败，总结为 FAIL；没有此类失败但有未解决阻塞则 BLOCKED；仅在所需风险批准完整且其他门满足时可 ACCEPTED_WITH_RISK。不可豁免的安全/合规约束不能通过用户风险接受绕过。

风险批准必须绑定到当前目标、标准和证据版本。变化后重新核验批准是否仍有效。没有答复不等于接受；不需要用户签收的普通自动化验收也不要新增人为签收环节。

缺少必需正式证据、关键 UNKNOWN、未确认业务语义或被违反的硬门不能用 ACCEPTED_WITH_RISK 豁免。已证实标准失败时保留 FAIL；没有已证实失败但存在这些缺口时为 BLOCKED。风险接受仅适用于已查明、允许豁免且获明确批准的偏差，不更改原始证据。

## Completion

只有同一原 Task Identity 的 ACCEPTANCE_PASS_GATE 满足且 Acceptance = PASS 时，才允许该任务转为 COMPLETED。正式路线必须有完整正式证据链；轻量路线仅能声明其限定任务完成，不等于整个产品或官方流程完成。与此任务无关的修复即使验收 PASS，也不能改变原任务 BLOCKED。

ACCEPTED_WITH_RISK 在 V1 不映射为 COMPLETED；明确保留风险与未完全满足项。FAIL/BLOCKED 同样不得报告 COMPLETED。无需因存在无关的其他 feature 而阻塞当前已授权范围的完成。

最终报告包括：

- 本次目标和已授权范围；
- 正式工件和变更引用；
- 实际验证与 Review 结论；
- 验收四态之一及依据；
- 是否 COMPLETED；否则是哪一步未完成、原因、当前证据、下一步；
- 已知风险、UNKNOWN、尚需用户动作；
- 达到用户要求阶段后停止，不自动转入发布/生产或下一阶段。
