---
name: frontend-delivery
description: 通用前端需求交付工作流。当用户要求评审前端需求、制定前端技术计划、按计划实现页面与接口、或完成前端需求的测试与交付总结时使用。支持 Web、H5、PC 管理端和小程序，不绑定具体技术栈。
metadata:
  version: "1.0.0"
---

# Frontend Delivery

从需求到交付的前端工作流。五个阶段、两个人工门禁、五个产物文件。

本文件是编排入口，只描述做什么和何时暂停。具体规则读 `references/` 下对应文件，不要在本文件里推断细节。

## 触发方式

首次启动：

```
使用 frontend-delivery 评审需求。
项目：<项目根目录，可多个>
需求：<需求文档路径，或直接描述>
需求标识：<唯一标识，如 202608131512>
UI目录：<可选>
切图目录：<可选>
接口文档：<可选>
产物目录：<可选，默认见下>
执行模式：<review_only | planning_only | full_delivery，默认 full_delivery>
```

恢复：

```
继续 frontend-delivery。
需求标识：<需求标识>
```

升级执行终点：

```
继续 frontend-delivery，升级为完整交付。
需求标识：<需求标识>
```

## 核心原则

1. 先读资料和代码，再提问；能从项目确定的不问开发。
2. 事实、推断、假设、待确认分开记录，不混为结论。
3. 影响范围、实现、安全或验收的不确定必须暂停。
4. 一次只向开发提一个关键问题；产品问题集中成可转发清单。
5. 只有已确认的结论进 `decisions.md`；未确认的留在 `context.md` 开放问题。
6. 需求未确认不出最终计划；计划未批准不改业务代码。
7. 只改本次需求直接涉及的文件，不顺手重构。
8. 测试结论必须有本次运行证据，未跑的如实说未跑。
9. 阻塞按范围记录，不误停无依赖的工作。

## 产物

默认目录（开发未指定时）：

- 单项目：`<primaryProjectPath>/docs/ai-delivery/<需求标识>/`
- 多项目：`<workspaceRoot>/docs/ai-delivery/<需求标识>/`

| 文件 | 内容 |
| --- | --- |
| `workflow-state.json` | 阶段、状态、输入路径、版本与批准信息 |
| `context.md` | 项目画像、需求理解、评审清单、复用分析、开放问题、可选估时 |
| `plan.md` | UI 与资源映射、文件清单、技术任务、API 清单、验证方案 |
| `delivery.md` | 验证证据、改动总结、未验证项、风险与遗留 |
| `decisions.md` | 已确认的产品/开发/UI/后端决策与需求变更，追加不覆盖 |

模板在 `assets/templates/`，复制后填充：删除未使用的占位行与不适用章节，把 `workflow-state.json` 的 `lastUpdate` 替换为当前带时区时间。估时属于 `context.md` 的按需附录，不进 `plan.md`，不改 `planVersion`。

## 阶段路由

每次调用先读 `workflow-state.json`，按 `stageStatus` 与 `blockingItems` 决定入口；没有状态文件则从阶段一开始。恢复规则见 `references/recovery-scenarios.md`，状态字段与取值见 `references/state-machine.md`。

不得越过 `targetStage`。

### 阶段一：理解上下文与需求评审

做四件事：分析项目、理解需求、复用分析、生成分级评审清单。方法见 `references/project-and-requirement-review.md`。

- 缺少需求描述和需求文档：记 `missing_requirement`，暂停。
- 项目不可访问：记 `inaccessible_project`（范围为该项目），其他项目继续。
- 无法确认包管理器、目标环境、规范优先级或未提交修改归属：暂停询问开发。

产出完整 `context.md`，设置 `requirementVersion`。

**门禁一（需求确认）：** P0 全部解决且开发确认需求理解后进入阶段二。P1 未解决时只能出标注假设的计划草案。P2 的默认方案连同理由先留在 `context.md`，开发明确接受后才追加 `decisions.md`——未明确否决不等于已确认。存在未解决 P0 时记 `p0_unresolved`（范围为问题 ID），`stageStatus.context` 为 `blocked`、`planning` 保持 `pending`。

### 阶段二：制定计划

先判断本次需求是否依赖 UI。依赖判定与 UI 分析方法见 `references/planning-and-ui.md`。

- 依赖 UI 但缺 UI 目录：记 `missing_ui`，范围为具体页面或任务，只停该范围及下游；`context.md` 列出所需页面、状态与交互图。
- 不依赖 UI：`plan.md` 的 UI 章节标"不适用"，继续。
- 不得为缺失的关键 UI 状态自行创造设计。

接口处理：可以识别需要哪些业务接口、可以列出待后端确认的问题，但不得编造 URL、方法、字段或错误码。契约缺失时记 `missing_api_contract`，范围为对应接口任务，其余任务继续。

产出 `plan.md`，设置 `planVersion` 与 `planBasedOnRequirementVersion`。必须暂停的情形见 `references/planning-and-ui.md`。

**门禁二（计划批准）：** P1 全部解决且开发批准当前 `plan.md` 后，将 `approvedPlanVersion` 设为当前 `planVersion`，记录 `approvedAt` 与 `approvedByRole`。等待批准期间 `stageStatus.planning` 为 `awaiting_confirmation`、`implementation` 保持 `pending`；批准后 `planning` 转 `complete`、`implementation` 转 `active`。

`approvedPlanVersion` 与当前 `planVersion` 不一致时，批准无效，不得改业务代码。

### 阶段三：实现功能

按 `plan.md` 的任务依赖顺序执行，跳过不适用步骤。实现规范、复用优先级、Mock 与接口对接规则见 `references/implementation-and-api.md`。

不预设框架写法：先读项目现有同类文件，匹配其技术栈、目录约定与代码风格。

遇到 UI 缺失、契约不符、公共能力影响、与开发未提交修改冲突、需要扩大范围或需要新增依赖时，暂停对应范围，写入 `context.md` 开放问题并询问开发，确认后追加 `decisions.md` 并更新 `plan.md`。

### 阶段四：验证质量

按 `plan.md` 的验证方案执行项目实际具备的检查，记录命令、结果与失败数量。策略与证据要求见 `references/testing-and-delivery.md`。

状态规则：

- 自动验证通过但人工验证项未执行：`stageStatus.verification` 为 `awaiting_confirmation`，在 `delivery.md` 列出待人工执行清单。
- 本次修改导致失败：修复后重跑；无法修复时记 `verification_failed`，范围为受影响模块。
- 只有必要验证全部完成（含人工验证已由开发确认）才转 `complete`。

未跑或失败的项如实写入 `delivery.md`，不得声明全部通过。

### 阶段五：交付总结

补全 `delivery.md`：完成功能、改动文件、对接接口、UI 验收结果、验证证据、未验证项、已知风险、对旧功能影响、需开发人工执行的操作。清单见 `references/testing-and-delivery.md`。

测试账号、环境凭证由开发提供，不得写入任何产物文件。

安装依赖、改环境配置、提交、推送、建 PR、部署、发布均需单独授权。

## 需求变更

收到新信息先分类，不要一律当需求变更：

| 类型 | 判断 | 处理 |
| --- | --- | --- |
| 需求澄清 | 只补原意，不改功能、范围、验收 | 更新 `context.md`，追加决策；未影响计划不重批 |
| 需求变更 | 改功能、业务规则、接口、范围或验收 | 建变更记录、影响分析、更新基线与计划 |
| UI 变更 | 改布局、交互、切图、状态或适配 | 重新分析受影响 UI、资源、代码与视觉验证 |
| 技术调整 | 需求不变，只改实现方案 | 更新 `plan.md`；影响已批准计划时重批 |
| 缺陷修复 | 实现不符合已确认需求 | 按缺陷处理，不抬为需求变更，补测试 |

处理流程、影响分析维度、分阶段规则与记录格式见 `references/change-control.md`。

关键约束：

- 变更确认后更新 `requirementVersion`；影响已批准计划时清空批准信息并标记失效范围。
- `planBasedOnRequirementVersion` 与 `requirementVersion` 不一致时，不得实现受影响部分。
- 只重做受影响范围，不从头重跑无关阶段。
口头或紧急变更：未确认内容写 `context.md` 开放问题并标注待确认，书面确认后才追加 `decisions.md`。是否允许基于临时假设继续开发，按下列边界判断：

- 涉及安全、资金、权限、隐私或数据写入的 P0 内容：不允许基于临时假设开发，等待正式确认。
- 其他内容：只有开发明确授权"按临时方案实现"后才可继续，授权本身记入 `context.md`。
- 临时实现必须可回退，在 `context.md` 记录改动范围与回退方式。
- 任何临时实现都不得交付或部署。
- 超过一个工作日未确认时暂停相关实现。

## 环境能力

使用当前 AI 客户端已有的能力，不假设特定工具名，也不要求 Python：

- 代码与文本搜索（含语义或结构化检索，若可用优先）
- 文件与目录读取
- 图片检查（读取 UI 图与切图）
- 精准补丁式文件编辑
- 项目命令执行（lint、类型检查、测试、构建）

缺少某项能力时说明受影响范围，并请开发提供替代输入或手动执行。

## 参考文件

| 文件 | 用途 |
| --- | --- |
| `references/state-machine.md` | 状态字段、取值、阻塞项与失效项格式、全局状态派生 |
| `references/workflow-gates.md` | P0/P1/P2 判断标准与两个门禁的完整规则 |
| `references/project-and-requirement-review.md` | 项目分析清单、需求分析维度、评审清单生成、复用分析 |
| `references/planning-and-ui.md` | UI 依赖判定、本地 UI 与切图分析、文件计划、验证方案格式、暂停情形 |
| `references/implementation-and-api.md` | 实现顺序、复用优先级、Mock 策略、接口对接与冲突处理 |
| `references/testing-and-delivery.md` | 测试选择、证据要求、交付清单、风险评估 |
| `references/recovery-scenarios.md` | 九个失败与恢复场景的期望行为 |
| `references/change-control.md` | 变更分类、影响分析、分阶段处理、记录格式 |

## 范围

支持：Web、H5、PC 管理端、微信/支付宝等小程序。

排除：原生 iOS、原生 Android。

其他跨端技术不预先排除：先分析技术栈、运行方式与验证条件，再询问开发是否继续。

不做：替产品/UI/后端决定未确认规则、上传源码或需求资料到外部服务、未经授权的提交与发布。
