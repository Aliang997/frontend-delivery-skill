# 交付报告

**需求标识：** {requirementId}  
**需求版本：** {requirementVersion}  
**批准计划版本：** {approvedPlanVersion}  
**交付时间：** {deliveryTimestamp}  
**交付状态：** {已完成 | 待人工验证确认 | 部分交付（存在阻塞项）}

> 本文档不记录任何测试账号、密码、Token、验证码或其他凭证。凭证由开发提供并自行保管。
>
> 生成规则：填充真实内容后删除所有未使用的占位行和示例行。验证证据只记录本次实际执行的命令与结果——未执行的项归入"待人工执行的验证清单"，不要在证据表里留占位行。

---

## 完成的功能

| 序号 | 项目 | 功能 | 对应计划任务 | 状态 |
| --- | --- | --- | --- | --- |
| 1 | {projectName} | {feature1} | {T1} | 已完成 |
| 2 | {projectName} | {feature2} | {T2} | 已完成 |
| 3 | {projectName} | {feature3} | {T3} | 未完成（原因：{reason3}） |

### 与计划的差异

| 计划内容 | 实际实现 | 原因 | 决策记录 |
| --- | --- | --- | --- |
| {planned1} | {actual1} | {reason1} | {DEV-00X | TECH-00X} |

---

## 创建和修改的文件

### 创建

| 项目 | 文件路径 | 类型 | 说明 |
| --- | --- | --- | --- |
| {projectName} | {createdFilePath1} | 业务 | {purpose1} |
| {projectName} | {createdFilePath2} | 测试 | {purpose2} |

### 修改

| 项目 | 文件路径 | 改动内容 | 影响面 |
| --- | --- | --- | --- |
| {projectName} | {modifiedFilePath1} | {change1} | {impact1} |

### 删除

| 项目 | 文件路径 | 删除原因 |
| --- | --- | --- |
| {projectName} | {deletedFilePath1} | {reason1} |

### 计划外改动

| 项目 | 文件路径 | 改动原因 | 是否已确认 |
| --- | --- | --- | --- |
| {projectName} | {unplannedFilePath1} | {reason1} | {DEV-00X} |

---

## 对接的接口

| 序号 | 项目 | 用途 | 方法 | 路径 | 契约来源 | 对接状态 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | {projectName} | {usage1} | {method1} | {path1} | {contractSource1} | 真实接口已联调 |
| 2 | {projectName} | {usage2} | {method2} | {path2} | {contractSource2} | 仅 Mock（原因：{reason2}） |
| 3 | {projectName} | {usage3} | {method3} | {path3} | — | 未对接（原因：{reason3}） |

### 与接口文档的差异

| 序号 | 差异内容 | 处理方式 | 决策记录 |
| --- | --- | --- | --- |
| {index} | {discrepancy} | {handling} | {DEC-00X | CR-00X} |

---

## UI验收结果

**UI 依赖：** {依赖 UI | 不适用（原因：{reasonNotApplicable}）}

| 页面/状态 | 项目 | UI 图 | 还原情况 | 差异说明 | 是否已确认 |
| --- | --- | --- | --- | --- | --- |
| {pageOrState1} | {projectName} | {uiFileName1} | 一致 | — | — |
| {pageOrState2} | {projectName} | {uiFileName2} | 有差异 | {difference2} | {UCR-00X | 待UI确认} |
| {pageOrState3} | {projectName} | — | 未验收 | 缺少 UI 图 | 待UI提供 |

### 未提供 UI 的状态处理

| 状态 | 实际实现 | 依据 | 是否已确认 |
| --- | --- | --- | --- |
| {stateWithoutUi1} | {implementation1} | {basis1} | {DEC-00X | 待确认} |

---

## 测试和构建证据

**状态含义：** 通过 = 本次实际运行且达到通过标准；失败 = 本次实际运行未达标；未执行 = 本次未运行，必须写明原因。

### 自动验证

| 序号 | 项目 | 类型 | 命令 | 状态 | 结果摘要 | 未执行原因 |
| --- | --- | --- | --- | --- | --- | --- |
| V1 | {projectName} | Lint | {lintCommand} | 通过 | 退出码 0，error 0，warning {n} | — |
| V2 | {projectName} | 类型检查 | {typecheckCommand} | 失败 | {errorCount} 个错误，见下方失败详情 | — |
| V3 | {projectName} | 单元测试 | {unitTestCommand} | 通过 | {passed} 通过 / {failed} 失败，覆盖率 {coverage} | — |
| V4 | {projectName} | 组件测试 | {componentTestCommand} | 未执行 | — | {项目无该测试能力 / 环境不可用 / 未在计划范围} |
| V5 | {projectName} | E2E | {e2eCommand} | 未执行 | — | {reason} |
| V6 | {projectName} | 构建 | {buildCommand} | 通过 | 退出码 0，产出 {buildArtifact} | — |

**运行时间：** {verificationRunTimestamp}

### 失败详情

| 序号 | 失败内容 | 是否本次修改引入 | 复现方式 | 处理状态 | 阻塞项 |
| --- | --- | --- | --- | --- | --- |
| V2 | {failureSummary} | {是/否（既有问题）} | {reproduceSteps} | {已修复并重跑 / 未修复} | {verification_failed / —} |

> 既有问题导致失败时，需记录与本次修改的关系和未覆盖风险；只有开发明确接受该风险并写入 decisions.md 后才继续交付，且不得写成"全部验证通过"。

### 已执行的人工验证

| 序号 | 项目 | 验证内容 | 环境/设备 | 状态 | 结果 | 执行角色 |
| --- | --- | --- | --- | --- | --- | --- |
| M1 | {projectName} | {manualCheck1} | {environment1} | 通过 | {result1} | 开发 |
| M2 | {projectName} | {manualCheck2} | {environment2} | 失败 | {result2} | 开发 |

### 缺少测试能力时的替代验证

| 缺失能力 | 项目 | 已采用的替代方式 | 未覆盖风险 |
| --- | --- | --- | --- |
| {missingCapability1} | {projectName} | {alternativeApproach1} | {uncoveredRisk1} |

---

## 待人工执行的验证清单

> 自动验证通过但以下人工验证尚未执行时，验证阶段保持 `awaiting_confirmation`，交付阶段不得标为 `complete`。开发执行并反馈结果后回填上方"已执行的人工验证"。

| 序号 | 项目 | 待验证内容 | 所需环境/设备 | 所需账号与权限 | 通过标准 | 未执行原因 |
| --- | --- | --- | --- | --- | --- | --- |
| M3 | {projectName} | {pendingManualCheck1} | {environment1} | {roleDescription1}（由开发提供） | {criteria1} | {AI 无法访问该环境 / 需真机 / 需业务权限} |
| M4 | {projectName} | {pendingManualCheck2} | {environment2} | {roleDescription2}（由开发提供） | {criteria2} | {reason2} |

**当前验证阶段状态：** {awaiting_confirmation | complete}

---

## 未验证内容

| 序号 | 项目 | 未验证内容 | 原因 | 风险 |
| --- | --- | --- | --- | --- |
| 1 | {projectName} | {unverified1} | {reason1} | {risk1} |

---

## 已知风险和遗留事项

### 风险

| 序号 | 项目 | 风险 | 触发条件 | 影响 | 等级 | 缓解措施 |
| --- | --- | --- | --- | --- | --- | --- |
| R1 | {projectName} | {risk1} | {trigger1} | {impact1} | {高/中/低} | {mitigation1} |

### 遗留事项

| 序号 | 项目 | 遗留内容 | 原因 | 建议处理 | 关联记录 |
| --- | --- | --- | --- | --- | --- |
| L1 | {projectName} | {leftover1} | {reason1} | {suggestion1} | {DEC-00X / 阻塞项} |

### 未清除的阻塞项

| 阻塞码 | 范围 | 详情 | 解除条件 |
| --- | --- | --- | --- |
| {blockingCode1} | {scope1} | {details1} | {releaseCondition1} |

---

## 对旧功能的影响

| 序号 | 项目 | 被影响的能力 | 影响方式 | 回归验证情况 |
| --- | --- | --- | --- | --- |
| 1 | {projectName} | {existingCapability1} | {impactType1} | {通过 / 失败 / 未执行（{reason}）} |

### 修改的公共能力

| 项目 | 公共能力 | 改动内容 | 其他调用方 | 是否已确认 |
| --- | --- | --- | --- | --- |
| {projectName} | {sharedCapability1} | {change1} | {otherCallers1} | {DEC-00X} |

**未修改的公共能力：** {unchangedSharedCapabilities}

---

## 需要开发人工执行的操作

| 序号 | 操作 | 说明 | 是否已授权 |
| --- | --- | --- | --- |
| 1 | 安装或升级依赖 | {dependencyDetail | 无} | 需单独授权 |
| 2 | 修改环境配置 | {envConfigDetail | 无} | 需单独授权 |
| 3 | 执行待人工验证清单 | 见上方 M{n} 各项 | — |
| 4 | 提交与推送代码 | {branchSuggestion} | 需单独授权 |
| 5 | 创建 PR | {prSuggestion} | 需单独授权 |
| 6 | 部署与发布 | {releaseOrderNote} | 需单独授权 |

### 多项目发布顺序

| 顺序 | 项目 | 说明 |
| --- | --- | --- |
| 1 | {projectName1} | {releaseNote1} |
| 2 | {projectName2} | {releaseNote2} |

---

## Delivery {changeId} / {deliveryDate}

> 沿用原需求标识处理需求变更时，在本文档末尾追加以变更编号命名的独立交付批次。禁止修改上方已有批次的内容。每个变更一个批次；下一次变更再追加新的同结构区块。

### 变更范围

**关联决策记录：** {CR-001 | UCR-001 | TECH-001}

**变更内容：** {changeSummary}

| 项目 | 改动文件 | 改动内容 | 类型 |
| --- | --- | --- | --- |
| {projectName} | {filePath1} | {change1} | {创建/修改/删除} |

**受影响接口：** {affectedApis | 无}

**受影响 UI：** {affectedUi | 无}

**不受影响范围：** {unaffectedScope}

### 验证证据

| 序号 | 项目 | 类型 | 命令 | 状态 | 结果摘要 | 未执行原因 |
| --- | --- | --- | --- | --- | --- | --- |
| V1 | {projectName} | {type1} | {command1} | {通过/失败/未执行} | {summary1} | {reason1} |

**回归范围：** {regressionScope}

**人工验证：** {manualVerificationStatus}

### 未验证项与风险

| 序号 | 未验证内容 | 原因 | 风险 | 等级 |
| --- | --- | --- | --- | --- |
| 1 | {unverified1} | {reason1} | {risk1} | {高/中/低} |

**待人工执行：** {pendingManualItems | 无}

---

**说明：** 本文档由 frontend-delivery skill 自动生成。验证状态只记录本次实际运行结果，未执行与失败项如实标注，不得声明全部通过。
