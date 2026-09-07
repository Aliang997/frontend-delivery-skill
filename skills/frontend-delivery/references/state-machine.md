# 完整流程状态与范围批准

本文是 workflow-state.json 的唯一字段定义。只用于完整流程；小改动和常规需求不为满足此状态机增加产物。
版本 2 以任务批准替代全局计划批准。旧文件恢复见 recovery-scenarios.md。

## 信息归属

context.md 保存需求与问题，plan.md 保存任务定义、版本、依赖和进度，decisions.md 保存确认历史，delivery.md 保存验证结果。
JSON 只保存阶段摘要、任务批准和阻塞/失效索引，不复制正文。
更新顺序：先写事实、问题或决定，再更新计划及结果，最后更新状态索引。中断后据实际文件核对，不补写不存在的批准。

## 字段

- schemaVersion：固定为 2。缺失视为旧版；未知更高版本保留原文件，不能猜语义覆盖。
- requirementId：需求内稳定且目录名可跨平台使用的标识。
- workspaceRoot、outputDir、primaryProjectPath：当前机器规范化绝对路径。
- projectPaths：项目根路径数组。归属按路径段包含关系选最深项目，不能用裸字符串前缀匹配；重复根先去重。
- requirementDocPath、uiDir、assetsDir、apiDocPath：可空；无目录不代表没有文字或现有实现依据。
- uiSourceType：figma、image、text、existing、mixed、none；旧状态中的 local 继续按本地图片或资源读取，新记录使用 image。具体 Figma 节点或图片来源在计划中记录。
- executionMode：review_only、planning_only、full_delivery。
- targetStage：上述模式分别对应 context、planning、delivery。先按用户请求选模式，不能把模板默认值当授权。
- requirementVersion：需求基线的来源版本或带时区时间；无基线为 null。
- planVersion：当前计划可执行内容版本，无计划为 null。
- planBasedOnRequirementVersion：整个计划已核对到的需求版本，无计划为 null。
- currentStage：主要推进或等待的阶段，不用于判断某个任务是否可执行。
- status：active、awaiting_confirmation、blocked、complete。
- stageStatus：context、planning、implementation、verification、delivery 各自状态。
- taskApprovals：当前任务批准数组。
- blockingItems：尚未解除的阻塞数组。
- invalidatedItems：需重做或重新检查的阶段范围数组。
- lastUpdate：每次保存的带时区 ISO 8601 时间；实际状态不可为 null。

初始化填写真实标识、路径、模式和时间。未进入阶段、未知版本保留 pending 或 null，不伪造完成。
目录与输出位置须在授权范围内，迁移机器时先重新定位再更新路径。

## 任务定义与版本

plan.md 每个任务保留：
- 稳定 ID、所属项目、验收项 ID、可执行内容、准确文件路径。
- 前置任务 ID、接口或 UI 来源、必要验证 ID。
- taskVersion：初版为 1，可执行内容变化时递增。
- 进度：pending、active、complete；阻塞和失效引用 JSON 对应项，不维护第二份原因。

每个验收项有任务，每个任务有验证方式；无依赖写“无”。
任务删除后 ID 不复用，必要时在决定里注明替代任务。
进度、错字、结果和估时变化不增加 taskVersion。
需求、文件范围、契约、依赖或验证要求改变时，提高相关任务版本；下游实际前提或验证依据改变时一并更新。
影响尚不确定时，先用范围级阻塞保护可能受影响部分，不假定未列出的任务安全。

## taskApprovals

每项字段：
- taskId：计划中实际存在的任务 ID。
- taskVersion：被批准的任务版本，正整数。
- approvedAt：实际确认的带时区时间。
- approvedByRole：已知确认角色，不虚构姓名。
- decisionId：decisions.md 中批准记录的 ID。

每个任务最多一项，保存最后一次实际批准；历史在 decisions.md。
批准时核对前提、P0/P1、范围和验证方案。批准不能越过依赖。
任务版本改变后原批准失效，即使旧项尚未清理；未变化任务保留批准。
批准记录列出任务及版本、planVersion 和批准范围，不能只有“已批准”。
全局 planVersion 或 requirementVersion 变化不自动取消所有批准。

## blockingItems

每项字段：
- code：missing_requirement、missing_ui、missing_api_contract、p0_unresolved、api_conflict、inaccessible_project、verification_failed、pending_decision 之一。
- scope：项目、页面、文件、接口或问题的可读标识；确实覆盖整个需求时用 all。
- taskIds：受影响的已知任务 ID 数组；未拆任务时可为空，按 scope 保守限定。
- questionId：context.md 问题 ID；没有需要人回答的问题时可为 null。
- details：事实、暂停原因和解除条件，不含凭证。
- createdAt：带时区时间。

pending_decision 覆盖工作区冲突、范围或依赖选择、必要验证不可用及其他待外部决定的情形，不只留在正文。
拆出任务后补齐 taskIds；影响和下游从计划确定。未建立映射时先分析，不把空 taskIds 当作无影响。
阻塞限制依赖该输入的执行与验收，解决问题、修复失败和补充计划本身仍可进行。
精确移除已解除项；同一范围不同原因可并存。

## invalidatedItems

每项字段：
- stage：context、planning、implementation、verification、delivery。
- scope、taskIds：沿用阻塞范围规则。
- reason：失效原因。
- changeId：decisions.md 中已经确认的变更记录 ID。
- createdAt：带时区时间。

未确认信息先记问题和阻塞，不引用不存在的变更决定。
失效表示旧产出不能作为有效依据，不禁止针对该产出的重做。
- context：相关需求重新分析并确认后解除。
- planning：相关计划更新且必要任务批准有效后解除。
- implementation：对应代码确已重做后解除。
- verification：对应证据重新取得，或按验证规则形成有效的既有问题风险接受结论后解除。
- delivery：更新后的任务与证据已核对、结果已记录后解除。

批准新计划只解除 planning 失效，不能清掉实现或验证待办。
局部失效保留其他结果；阶段整体需重做才标 invalidated，但任何必要局部工作尚未恢复时，该阶段不能保持 complete。

## 某任务能否实现

依次确认：
1. 当前终点允许实现，操作已授权。
2. 用户和项目限制允许，没有未解决的他人工作冲突。
3. 任务有当前定义；taskApprovals 的 ID 和版本匹配，并能找到真实批准记录。
4. 业务前提已确认，输入齐全，没有覆盖本次执行的未决阻塞。
5. 计划前提有效，前置任务已完成且无使当前依赖失效的待重做结果。

版本匹配不代替前提和依赖检查。
全局 planBasedOnRequirementVersion 尚未同步时，只有影响已隔离、确认及计划未变化的任务可沿用原批准；其他任务先补计划。
空 taskApprovals 不授予权限。小改动与常规需求按 workflow-gates.md 的直接授权执行，不套此数组。

最小示例：
- 正例：终点允许实现、操作获准，T-02 当前版本已批准，输入及前置任务有效；T-01 的阻塞不影响 T-02，可执行 T-02。
- 反例：T-02 的批准版本匹配，但它依赖的 T-01 已失效；不能仅凭批准执行，先恢复该前提。

## 阶段状态

pending：尚未开始。
active：正在产出、修复或重做。
awaiting_confirmation：供批准、选择或人工验证的产出和步骤已齐，下一步只等决定或结果。
blocked：剩余必要工作缺资料或环境，或冲突尚未形成可供决定的方案。
invalidated：全部既有结论需要重做。
complete：本阶段产出目标已满足，不代表下一阶段授权。

阶段完成条件：
- context：分析、验收项、复用影响和问题清单完整。仍有问题可完成评审，但明确需求未确认，保留问题和阻塞。
- planning：完整计划或符合用户请求的草案已产出，缺口明确。计划编写完成与任务批准分别维护。
- implementation：当前范围必要任务确已实现，未决临时实现不算正式完成。
- verification：满足 testing-and-delivery.md 的必要验证条件，无待执行必要验证。
- delivery：实现与必要验证完成，风险处理和报告齐全；不表示已部署。

context、planning 可为 complete 而后续仍阻塞或待批准。只评审/计划时不得为完成委托强行解决业务决定或开始实现。

## 全局状态派生

先核对事实、开放问题、任务与阶段索引一致，再依次判断：
1. 已达用户终点且满足上述产出条件：complete。
   - review_only、planning_only 允许保留阻止后续实现的问题，报告必须写明。
   - full_delivery 必须完成必要任务、验证及风险处置，且无影响交付的阻塞或失效项。
2. 有已获准执行任务，或可做的分析、修复、重做：active。
3. 无可执行工作，必要资料或环境缺失、冲突尚无可供决定的方案，或失效无法继续修复：blocked。
4. 无可执行工作，供决定的方案或人工验证步骤与前提已齐，只等任务批准、明确方案的选择、人工结果或风险决定：awaiting_confirmation。

未达用户终点且无其他可执行工作时，缺接口契约等事实输入属于 blocked，即使正在等人回复；所需输入已齐，只等选择或确认才属 awaiting_confirmation。不能仅凭 blockingItems 是否为空或 pending_decision 这个 code 判断。
最小示例：
- 正例：T-01 缺契约，但已获准且独立的 T-02 可执行，全局为 active；T-02 完成后若仅剩缺契约的任务，才为 blocked。
- 反例：所需输入已齐，只待用户批准具体任务，却因存在 pending_decision 就标 blocked；应为 awaiting_confirmation。

currentStage 取主要的最早待推进阶段；多个范围可分别在计划、实现或验证中。
必要人工验证未完成时可先写交接报告，但 delivery 不得 complete。
