---
name: product-engineering
description: 使用官方 Spec Kit 编排从产品需求到验证、Code Review 和产品验收的研发流程。适用于新产品、新功能，以及需要规格驱动交付的修复、重构、迁移或已有 feature 接续。负责上下文、授权、证据与完成判定；集成而不替代 Spec Kit。单纯问答、只读审查或局部编辑不自动升级为完整产品研发。
---

# Product Engineering

Spec Kit 管规格与 SDD 生命周期；本 Skill 管产品研发编排与完成判定。

## 启动与边界

先确定用户要求的是分析、设计、实施还是接续，以及允许停在哪个阶段。明确阶段限制优先于默认完整生命周期；仅请求设计不授权实现。说明本 Skill 的用途与当前选择的范围。

检查仓库再询问：区分现有事实、目标要求、项目规则和推断。仓库事实用于回答“现在是什么”，不能否决用户明确的变更目标。发现冲突时记录差异，遵循适用指令与授权；不要凭空填补关键业务语义。

本 Skill 不安装或升级 Spec Kit，不生成替代命令，不改其官方 Skills、模板或内部状态，不自动部署、创建 Issue/PR，也不创建多 Agent 平台。源码、测试与配置仍属于项目。不要在业务执行中修改本 Skill 自身。

## 硬性执行门

以下硬门优先于默认生命周期中的实施动作，不能用“已授权开发”或“实现很简单”豁免：

- `SPEC_KIT_REQUIRED_GATE`：先 Discovery → Workflow Routing → 明确是否依赖 Spec Kit → Capability Detection → Gate。需要的官方能力不可用时，原任务 BLOCKED；禁止修改业务源码、业务测试或绕过门的配置，只能继续安全只读调查并报告恢复条件。
- `HUMAN_GATE`：关键业务权限/数据语义无法从需求与项目规则确定时，原任务 BLOCKED，先提供有上下文的确认问题。纯函数、局部实现或技术可逆不降低业务风险。
- `TASK_BINDING`：原目标、Task Identity、证据、验收始终绑定。阻塞原任务的 UNKNOWN 不得用另一项修复消除；无关发现不能替代原任务完成。
- `ACCEPTANCE_PASS_GATE`：按已确定的路线逐项检查所需证据，缺失必需正式证据、关键 UNKNOWN、业务确认或 Risk Gate 未解除时，禁止 PASS 和 COMPLETED。ACCEPTED_WITH_RISK 永不自动成为 COMPLETED。

轻量任务的适用条件由路由参考明确决定，不能在发现 Spec Kit 缺失后临时降级。门是本 Skill 的执行约束，不是新的工具、数据库或官方状态字段。

## 按需加载

不要一次加载全部文档或模板。参考路径相对于本 Skill 目录，不是业务仓库根。

| 何时 | 加载 |
|---|---|
| 开始任务，或请求范围改变 | [工作流路由](references/workflow-routing.md) |
| 首次有副作用的操作前，或出现歧义、风险、UNKNOWN | [自主决策与授权](references/autonomy-policy.md) |
| 首次进入 Spec Kit 阶段，或环境/feature/版本变化 | [集成契约](references/spec-kit-integration.md) |
| 定义阶段出口、判断阶段是否成功、执行 Review | [证据与质量门](references/evidence-and-quality-gates.md) |
| 定义产品验收及最终完成结论 | [产品验收](references/product-acceptance.md) |

需要具体记录时才使用对应的轻量模板；不为凑齐文档生成空壳。

## 默认生命周期

0. Intake：确定目标、请求类型、范围与终点。
1. Repository Discovery：发现仓库事实与未完成工作。
2. Workflow Routing / Risk & Authorization Check：绑定原任务与范围，明确 Spec Kit 是否必需及需确认事项。
3. Product Brief：整理目标与初始验收信号，不替代正式规格。
4. Spec Kit Capability Detection / SPEC_KIT_REQUIRED_GATE：正式路线探测所需官方能力；缺失则 BLOCKED，禁止进入依赖它的实施阶段。轻量路线按已记录的适用性处理。
5. Constitution：复用已有效的原则；仅在需要且获授权时更新。
6. Specify：使用官方能力定义本次变更。
7. Clarify：按需解决有实质影响的剩余歧义。
8. Plan：使用官方能力形成技术设计。
9. Tasks：使用官方能力拆解任务。
10. Analyze：使用官方能力检查工件一致性。
11. Implement：使用官方能力实施并验证。
12. Converge：使用官方能力检查实现缺口。
13. Code Review：独立于 Implement 的审查步骤。
14. Product Acceptance：依据产品目标和证据独立验收。
15. Completion Report：报告真实完成程度并停止于授权终点。

这是正式路线的编排顺序，不是另一套 Spec Kit 命令。轻量路线依路由参考执行，不强求不适用的官方阶段。有效既有阶段可复用，但必须核验工件仍适用于本次目标。不得为继续一个 feature 而无条件重跑 specify 或重建 tasks。

## 执行纪律

每阶段依据质量门明确 Input、Action、Expected Output、Validation、Failure Handling。正式路线的官方阶段先验证前置条件，再加载并执行已确认来源的官方 Skill，最后检查工件和实际变更；轻量路线遵循已确定的适用门。打印 Skill 名称或命令退出 0 都不是完成证据。

绑定同一项目根、feature、输入版本与授权范围。缺少官方能力时报告 UNKNOWN/缺失状态并停止依赖它的阶段，不自行重实现。可继续不依赖该能力的已授权只读调查。

测试从最窄相关范围逐步扩大。测试失败、评审缺陷或规格改变时，回到受影响阶段，更新证据；不篡改标准消除失败。不把任务勾选、workflow completed 或 converge 声明等同于产品验收。

最终验收只使用 PASS、FAIL、BLOCKED、ACCEPTED_WITH_RISK。只有全部必需质量门通过且产品验收为 PASS，才能报告 COMPLETED；风险接受不得静默升级为完全完成。按验收参考输出未完成阶段、原因、证据及下一步。到达用户指定终点立即停止。
