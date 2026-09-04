# Spec Kit Integration Contract V1

## 基线与来源

2026-09-04 的 Phase 2 已核验官方 `v1.0.4`（`cb610277fdea781fcfa83d20522c2db37c94068d`）源码；这只是参考基线，不是唯一兼容版本，也不是运行认证。先检查本地实际版本与工件，再按需核查该版本官方实现。不得把 main、安装 CLI 的版本和项目工件版本混为一谈。

官方来源（只作核查，不复制模板）：

- [Codex integration](https://github.com/github/spec-kit/blob/v1.0.4/src/specify_cli/integrations/codex/__init__.py)
- [CLI 与版本能力](https://github.com/github/spec-kit/blob/v1.0.4/src/specify_cli/__init__.py)
- [集成管理与状态](https://github.com/github/spec-kit/blob/v1.0.4/docs/reference/integrations.md)
- [feature 路径解析](https://github.com/github/spec-kit/blob/v1.0.4/scripts/bash/common.sh)
- [前置检查脚本](https://github.com/github/spec-kit/blob/v1.0.4/scripts/bash/check-prerequisites.sh)
- [升级边界](https://github.com/github/spec-kit/blob/v1.0.4/docs/upgrade.md)
- [presets](https://github.com/github/spec-kit/blob/v1.0.4/docs/reference/presets.md)、[extensions/hooks](https://github.com/github/spec-kit/blob/v1.0.4/docs/reference/extensions.md)
- [workflow 恢复实现](https://github.com/github/spec-kit/blob/v1.0.4/src/specify_cli/workflows/engine.py)

## Capability Detection First

所有路径和命令都是经探测确认后才可使用的能力；以下路径/参数仅描述上述基线。不要通过初始化、升级、创建 feature 来探测状态。

1. 确定用户授权的项目根、真实路径和当前工作目录。检查相关 `SPECIFY_INIT_DIR` / `SPECIFY_FEATURE_DIRECTORY` 环境覆盖是否指向该目标，不输出整个环境或凭据。
2. 检查实际 specify executable；未找到记 ABSENT，启动失败记 BROKEN，不能推断整机从未安装。
3. 通过帮助确认支持后优先读取 `specify version --features --json`；其基线支持 `--version`/`-V` 和 `version`。`version --json` 单独使用无效。无法解析版本记 UNKNOWN。
4. 检查初始化标志与元数据。基线 CLI 以 `.specify/` 目录为最低标志；仅此目录存在不等于完整初始化。无目录先结束项目级命令探测，不运行 init。
5. 已确认支持且在目标根执行 `specify integration status --json`，读取 findings、manifests、default/installed integration。基线 warning 可返回 0；非零结果也可能包含有用 JSON。`integration list` 是项目级清单，`specify check` 只是工具检查，均不能替代健康诊断。
6. 比对 integration 元数据、manifest 和磁盘 Skills，记录实际绝对路径及内容指纹。基线 Codex 默认为项目 `.agents/skills/speckit-<name>/SKILL.md`，但不按该候选路径盲目执行。
7. 检查宿主能否发现/明确加载该 Skill、是否存在同名异源副本。确认目标来源后完整读取其说明和必要引用；磁盘存在、元数据声称已安装和会话可执行是不同状态。
8. 读取生效 hooks、preset、override 与相关脚本；根据实际阶段判断副作用。任何必需行为 UNKNOWN，停止该阶段写操作。

不要为查询最新版本而自动联网更新；本地能力探测不要求最新版本。版本标志不是完整命令目录，缺失键记 UNKNOWN，不默认 false/true。

## SPEC_KIT_REQUIRED_GATE

使用路由已绑定的 `spec_kit_required`，不得根据探测失败反向选择轻量路线：

- false：仅执行路由明确的轻量任务；不声称已运行官方阶段。
- true：逐项核对当前路线/阶段必需的官方能力、项目状态、输入工件及已知副作用。全部满足才允许进入对应官方阶段；阶段升级或环境变化时复核。新 feature 尚未生成的输出不当作前置文件，但不能据此跳过生成它的官方阶段。
- true 且任一必需能力缺失、BROKEN 或无法确认：`SPEC_KIT_REQUIRED_GATE = BLOCKED`，原任务 BLOCKED，停止依赖实施；不得只在结尾附一句“未走完整 Spec Kit”后继续写代码。

门受阻时只允许 Discovery、只读调查/诊断、风险分析、收集事实、记录 UNKNOWN 和恢复条件；独立诊断不得改变目标。禁止修改业务源码、业务测试、绕过门的配置，禁止伪造/模拟官方工件或宣称 Spec 完成、Acceptance PASS、COMPLETED。缺少工具不是安装授权；“用户已授权开发”也不是绕过质量门的授权。

报告应保留原 Task Identity、缺失能力及证据、受阻阶段、未执行的动作和可验证的恢复条件。技术缺失与业务确认分别记录；一个门通过不解除另一个门。

## Capability Snapshot

在会话或项目约定的 PE 记录位置保留以下事实，不写回官方元数据：

| 分组 | 内容 |
|---|---|
| Project | 项目根、初始化标志、完整性与读取异常 |
| CLI | executable、版本、来源、已验证参数和退出码 |
| Integration | 默认/已安装项、findings、manifest 版本及管理文件变化 |
| Skills | 实际名称、绝对路径、来源、指纹、可加载状态 |
| Feature | 路径、选择依据、解析来源、输入工件版本 |
| Customization | 生效 hooks/presets/overrides、额外副作用 |
| Compatibility | 本次确认的能力、UNKNOWN、受阻阶段 |
| Evidence | 核查时间、命令结果摘要、文件引用与解除条件 |

环境、feature、受管文件或版本变化后重新探测。证据快照不是第二套 Spec Kit 状态机。

## Feature Detection

基线路径优先级是 `SPECIFY_FEATURE_DIRECTORY`，其次 `.specify/feature.json` 的 `feature_directory`，否则报错。Git 分支不是 feature 路径；`SPECIFY_FEATURE`/BRANCH 名称不能替代目录解析。

基线 Bash 的 `check-prerequisites.sh --json --paths-only` 使用不持久化解析；它不验证目录和规格存在，需另行检查。普通模式可能持久化环境覆盖到 feature.json，不属于严格只读检查。只有确认实际脚本及其引用安全后才能执行；其他版本/脚本变体先核验，不套用参数。

严格区分缺失 JSON、损坏 JSON、类型错误、目录缺失、越界路径和权限错误。确认解析目录在授权范围，spec/plan/tasks 的存在与有效性按阶段检查。不选择“最新目录”或“编号最大目录”。多个候选且请求不能消歧才询问用户。

## Integration Contract

使用已安装且来源明确的官方 constitution、specify、clarify、plan、tasks、analyze、implement、converge Skills；checklist 按需。基线 Codex 使用 `$speckit-<name>`，这是 Skill 引用而不是 shell executable。加载并遵循实际官方 Skill 才算执行，打印引用不算调用。不存在统一已认证的 Skill 结果 JSON API。

每次阶段绑定：项目根、feature、Skill 路径/指纹、输入工件、阶段授权、允许变更范围与质量门。执行后检查真实输出、diff、指针、测试和 hooks 结果，不以退出码或自然语言“完成”替代验证。

PE 只组织输入和核验输出。不得生成另一套规格命令，导入私有 specify_cli 类，复制官方模板，修改官方 Skill，手写状态/manifest，或改变哈希去掩盖定制。

### Ownership

| 内容（基线路径） | 归属与写入边界 |
|---|---|
| `.specify/` 整体 | 混合所有权，禁止整体覆盖/删除 |
| `integration.json`、`integrations/*.manifest.json` | Spec Kit 管理状态；PE 只读 |
| `init-options.json`、`feature.json` | 工具维护项目选择/本地指针；通过官方行为维护，不手工伪造 |
| `scripts/`、核心 `templates/` | 官方基础设施，可能有用户定制；PE 不改 |
| `templates/overrides/`、扩展项目配置 | 项目/用户定制；V1 不新增或覆盖 |
| `memory/constitution.md` | 项目治理，用户决定实质原则；官方 constitution 阶段维护 |
| `extensions/`、`presets/`、`workflows/` | 安装、配置、运行记录混合；禁止笼统接管 |
| `.agents/` 整体、其他 Skills | 共享宿主目录/各作者所有，不清理或禁用 |
| 官方 `speckit-*/SKILL.md` | 官方安装与定制流程维护；PE 读取执行 |
| 本 Skill 的 `SKILL.md` | PE 维护者所有，业务流程不自修改 |
| `specs/` 或真实 feature 工件 | 项目所有，官方阶段维护；不建平行 truth source |
| `AGENTS.md` | 项目/用户规范；遵循不重写，检查可选 agent-context 扩展影响 |
| 源码、测试、业务配置 | 项目所有；实施授权内修改 |
| PE Brief/决策/审查/验收记录 | PE 编排证据，引用正式工件，不复制其内容 |

### 定制、版本与升级

基线有独立 CLI 版本、integration/shared manifest 版本及扩展版本；升级 CLI 不自动等同刷新项目文件。manifest-aware upgrade 保护项目规格和 constitution，对改动管理文件有保护；强制 init 是更宽的恢复操作，不能当普通重试。本 Skill 不自动执行上述维护。

基线 preset 有覆盖/组合优先级，命令物化到默认 integration；非默认 integration 不自动拥有相同定制。读取实际 Skill，不自建 resolver。hooks 的 `auto_execute_hooks` 在基线是保留字段，不是安全关闭开关；检查实际 hooks 及前后动作，不擅自绕过强制 hook。

workflow 是可选项。V1 主路径在当前会话执行官方 Skills，不启动额外 Codex 进程。已存在 workflow 可查状态；恢复前确认定义、run_id、暂停步骤、授权与副作用。基线 resume 会重跑当前步骤，只接收 paused/failed，不保证 exactly-once。不得编辑 state.json 强行恢复。workflow completed 不代表验收通过。

## Failure / Recovery

| 情况 | 处理 |
|---|---|
| CLI 未安装/损坏 | 查已配置路径和原因，给出维护建议；不安装，不模拟 CLI |
| 未初始化 | 准备冲突路径与影响清单，报告缺少前置条件；不自动 init |
| 版本/状态 schema 不兼容 | 查本地帮助及对应官方实现；无法确认则 UNKNOWN，停止依赖写操作，不自动升降级 |
| integration 缺失 | 对照 metadata/manifest/文件，建议官方维护路径；不手工生成 Skills |
| Skill 缺失/宿主未加载 | 区分磁盘缺失、同名冲突、禁用或刷新问题；不复制远程模板替代 |
| feature 状态异常 | 检查原始状态和环境覆盖；不删除指针试错 |
| spec 与代码不一致 | 定位证据，按已授权官方流程修复；业务语义变化先确认 |
| implement 中断 | 核对实际 diff、任务状态、测试和副作用后续做；不重新编号/清空任务 |
| 用户退出 | 保留现场，停止；新接续请求重新预检，不自动后台继续 |
| 未完成 feature | 匹配请求并从失效阶段恢复，不无条件重新 specify |
| 多 feature | 显式绑定一次写目标；歧义调查后确认，禁止靠目录时间猜测 |

已有项目先读取原有规则与工件，不重新初始化。非 Spec Kit 项目的接入、已有 integration 的增装/切换、升级和损坏状态修复属于独立维护，不在 V1 自动执行范围。提供影响清单和安全下一步；不擅自 commit/stash 用户文件。

UNKNOWN 必须带来源、已尝试检查、受影响阶段和下一步。哈希不一致是变更证据，不等于恶意或自动不兼容；不得为消除警告覆盖定制。
