# Frontend Delivery 工作流程图

## 1. 五阶段总览

```mermaid
graph TD
    Start([开始]) --> CheckState{是否存在<br/>状态文件?}
    CheckState -->|否| Stage1[阶段一<br/>理解上下文与需求评审]
    CheckState -->|是| Recovery[按状态恢复]
    
    Recovery --> Stage1
    Stage1 --> Gate1{门禁一<br/>P0全部解决?}
    Gate1 -->|否| Block1[暂停等待确认]
    Gate1 -->|是| Stage2[阶段二<br/>制定计划]
    
    Block1 -.产品确认.-> Gate1
    
    Stage2 --> Gate2{门禁二<br/>P1解决且批准?}
    Gate2 -->|否| Block2[等待批准]
    Gate2 -->|是| Stage3[阶段三<br/>实现功能]
    
    Block2 -.开发批准.-> Gate2
    
    Stage3 --> Stage4[阶段四<br/>验证质量]
    Stage4 --> Stage5[阶段五<br/>交付总结]
    Stage5 --> Complete([交付完成])
    
    Stage1 -.需求变更.-> ChangeFlow[变更控制]
    Stage2 -.需求变更.-> ChangeFlow
    Stage3 -.需求变更.-> ChangeFlow
    Stage4 -.需求变更.-> ChangeFlow
    ChangeFlow -.处理完毕.-> Recovery

    style Gate1 fill:#ff9999
    style Gate2 fill:#ff9999
    style Block1 fill:#ffcccc
    style Block2 fill:#ffcccc
    style Complete fill:#99ff99
```

## 2. 状态机转换

```mermaid
stateDiagram-v2
    [*] --> pending
    
    pending --> active
    pending --> blocked
    
    active --> awaiting_confirmation
    active --> blocked
    active --> complete
    
    awaiting_confirmation --> active
    awaiting_confirmation --> blocked
    
    blocked --> active
    blocked --> pending
    
    complete --> invalidated
    invalidated --> active
    
    complete --> [*]
```

## 3. 门禁一：需求确认

```mermaid
graph LR
    A[生成context.md] --> B{检查P0问题}
    B -->|有未解决P0| C[标记blocked]
    B -->|P0全部解决| D[标记complete]
    
    C --> E[输出待确认清单]
    E -.等待产品回复.-> F[更新文档]
    F --> B
    
    D --> G[进入阶段二]

    style C fill:#ffcccc
    style D fill:#ccffcc
```

## 4. 门禁二：计划批准

```mermaid
graph LR
    A[生成plan.md] --> B{检查P1问题}
    B -->|有未解决P1| C[列出影响<br/>等待解决]
    B -->|P1全部解决| D{开发是否批准?}
    
    C -.P1解决.-> B
    
    D -->|要求修改| E[修改计划<br/>更新版本]
    D -->|等待中| F[awaiting_confirmation]
    D -->|批准| G[记录批准信息]
    
    E --> D
    G --> H[进入阶段三]

    style F fill:#ffffcc
    style G fill:#ccffcc
```

## 5. 变更分类决策树

```mermaid
graph TD
    A([收到新信息]) --> B{变更分类}
    
    B -->|需求澄清| C1[更新context.md]
    B -->|需求变更| C2[影响分析<br/>更新基线]
    B -->|UI变更| C3[重新分析UI]
    B -->|技术调整| C4[更新plan.md]
    B -->|缺陷修复| C5[直接修复]
    
    C1 --> D1[追加decisions.md]
    C2 --> D2[清空批准信息<br/>重新批准]
    C3 --> D3[更新资源映射]
    C4 --> D4[必要时重批]
    C5 --> D5[补充测试]
    
    D1 --> E[继续执行]
    D2 --> E
    D3 --> E
    D4 --> E
    D5 --> E

    style C2 fill:#ff9999
    style D2 fill:#ffcccc
```

## 6. 阻塞范围控制

```mermaid
graph TD
    A[遇到阻塞] --> B{阻塞范围}
    
    B -->|全局| C1[缺少需求描述<br/>missing_requirement]
    B -->|项目级| C2[项目不可访问<br/>inaccessible_project]
    B -->|页面级| C3[缺少UI<br/>missing_ui]
    B -->|任务级| C4[缺少接口契约<br/>missing_api_contract]
    B -->|问题级| C5[P0未解决<br/>p0_unresolved]
    
    C1 --> D1{其他工作<br/>能继续?}
    C2 --> D2{其他项目<br/>能继续?}
    C3 --> D3{其他页面<br/>能继续?}
    C4 --> D4{其他任务<br/>能继续?}
    C5 --> D5{是否影响<br/>制定计划?}
    
    D1 -->|否| E1[全局blocked]
    D1 -->|是| E2[部分继续]
    D2 -->|是| E2
    D3 -->|是| E2
    D4 -->|是| E2
    D5 -->|是| E3[context=blocked]
    D5 -->|否| E2
    
    E1 --> F[等待解除]
    E2 --> F
    E3 --> F
    F -.解除阻塞.-> G[恢复执行]

    style E1 fill:#ff6666
    style E2 fill:#ffcc66
    style G fill:#66ff66
```

## 7. 产物文件关系

```mermaid
graph LR
    subgraph 输入
        I1[需求文档]
        I2[UI目录]
        I3[切图目录]
        I4[接口文档]
        I5[项目代码]
    end
    
    subgraph 产物
        P1[workflow-state.json]
        P2[context.md]
        P3[plan.md]
        P4[delivery.md]
        P5[decisions.md]
    end
    
    subgraph 输出
        O1[业务代码]
        O2[测试用例]
        O3[交付报告]
    end
    
    I1 --> P2
    I2 --> P2
    I2 --> P3
    I3 --> P3
    I4 --> P3
    I5 --> P2
    I5 --> P3
    
    P2 --> P1
    P2 --> P3
    P3 --> P1
    P3 --> O1
    P3 --> O2
    
    O1 --> P4
    O2 --> P4
    P4 --> O3
    
    P5 -.追加记录.-> P2
    P5 -.追加记录.-> P3
    P5 -.追加记录.-> P4

    style P1 fill:#e1f5ff
    style P2 fill:#fff4e1
    style P3 fill:#ffe1f5
    style P4 fill:#e1ffe1
    style P5 fill:#f5e1ff
```

## 8. 完整执行流程（含异常路径）

```mermaid
graph TD
    Start([接收需求]) --> A1[登记输入]
    A1 --> A2[分析项目]
    A2 --> A3[分析需求]
    A3 --> A4[需求评审]
    A4 --> G1{门禁一}
    
    G1 -->|P0未解决| Wait1[暂停]
    G1 -->|通过| B1[检查UI依赖]
    
    Wait1 -.确认后.-> G1
    
    B1 --> B2[分析UI和资源]
    B2 --> B3[制定技术方案]
    B3 --> B4[生成plan.md]
    B4 --> G2{门禁二}
    
    G2 -->|P1未解决或未批准| Wait2[暂停]
    G2 -->|批准| C1[按计划实现]
    
    Wait2 -.批准后.-> G2
    
    C1 --> C2{遇到问题?}
    C2 -->|UI缺失| Wait3[范围阻塞]
    C2 -->|契约不符| Wait3
    C2 -->|需扩大范围| Wait4[询问确认]
    C2 -->|正常| C3[实现完成]
    
    Wait3 -.解决后.-> C1
    Wait4 -.确认后.-> C1
    
    C3 --> D1[执行验证]
    D1 --> D2{验证结果}
    D2 -->|失败| D3{能修复?}
    D2 -->|通过| D4{需人工验证?}
    
    D3 -->|是| Fix[修复]
    D3 -->|否| Wait5[记录失败]
    
    Fix --> D1
    Wait5 -.解决后.-> D1
    
    D4 -->|是| Wait6[等待确认]
    D4 -->|否| E1[生成交付报告]
    
    Wait6 -.确认后.-> E1
    
    E1 --> End([交付完成])

    style G1 fill:#ff9999
    style G2 fill:#ff9999
    style Wait1 fill:#ffcccc
    style Wait2 fill:#ffffcc
    style Wait3 fill:#ffcccc
    style Wait4 fill:#ffffcc
    style Wait5 fill:#ffcccc
    style Wait6 fill:#ffffcc
    style End fill:#99ff99
```

---

## 说明

1. **简化设计**：将原来的超大图拆分成 8 个独立图表，每个聚焦一个核心流程
2. **避免节点冲突**：每个图表使用独立的节点 ID（A1, B1, C1 等）
3. **清晰的视觉层次**：
   - 红色：门禁和关键决策
   - 黄色：等待确认
   - 粉色：阻塞状态
   - 绿色：完成状态
4. **渐进式阅读**：从总览到细节，从状态机到具体流程

每个图表都可以独立渲染，适合在文档、GitHub、Obsidian 等工具中查看。
