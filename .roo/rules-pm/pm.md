# 项目管理模式规则

## 开发流程

开发将按照以下阶段进行。

```mermaid
flowchart LR
    %% AI模式
    subgraph Modes [AI模式]
        direction LR
        PMMode[PM模式<br>项目管理]
        ArchMode[Architect模式<br>系统设计]
        CodeMode[Code模式<br>实现与测试]
    end

    %% 主流程
    Start([开始]) --> RC[1.规则<br>确认]
    RC --> RD[2.需求<br>定义]
    RD --> DS[3.设计]
    DS --> IM[4.实现]
    IM --> TS[5.测试]

    %% 阶段结束时的共通处理（主流程下方）
    subgraph PhaseEnd [阶段结束时的处理]
        direction LR
        Commit[成果<br>git commit]
        --> Review[请求用户<br>评审]
    end

    %% 各阶段与结束处理的连接（右下方向）
    RC --> PhaseEnd
    RD --> PhaseEnd
    DS --> PhaseEnd
    IM --> PhaseEnd
    TS --> PhaseEnd

    PhaseEnd --> NextPhase{是否有下一个阶段？}
    NextPhase -->|Yes| NextStart[进入下一个阶段]
    NextPhase -->|No| End([完成])

    %% 模式与各阶段的关联（上部到下部的箭头）
    PMMode -.-> RD
    ArchMode -.-> DS
    CodeMode -.-> IM
    CodeMode -.-> TS

    %% 样式定义
    classDef phaseNode fill:#f5f5ff,stroke:#333,stroke-width:1px
    class RC,RD,DS,IM,TS phaseNode

    classDef processNode fill:#fff5f5,stroke:#333,stroke-width:1px
    class Commit,Review,Correct processNode

    classDef modeNode fill:#e6f7ff,stroke:#0066cc,stroke-width:1px
    class PMMode,ArchMode,CodeMode modeNode

    classDef decisionNode fill:#fffbf5,stroke:#333,stroke-width:1px
    class Fix,NextPhase decisionNode
```

## 需求定义阶段

```mermaid
flowchart TD
    Start([需求定义开始]) --> RD1[创建分支]
    RD1 --> RD1a[feature/功能名]
    RD1 --> RD1b[fix/bug名]
    RD1 --> RD1c[hotfix/紧急bug名]
    RD1a --> RD2[编写需求与功能列表]
    RD1b --> RD2
    RD1c --> RD2
    RD2 --> RD3[记录到 docs/requirements-definition.md]
    RD3 --> RD4[遵循 markdownlint 标准规则]
    RD4 --> End([需求定义完成])

    classDef reqNode fill:#f9f9f9,stroke:#333,stroke-width:1px
    class RD1,RD1a,RD1b,RD1c,RD2,RD3,RD4 reqNode
```

1. 分支命名：请以 feature/[功能名]、fix/[bug名]、hotfix/[紧急bug名] 创建 git 分支并开始工作
2. 请将针对所给指令的需求与功能列表记录到 `docs/requirements-definition.md` 文件中

- markdown 文件需遵循 markdownlint 的标准规则

### 文档编写规范

#### requirements-definition.md

1. 需求定义文档结构

   - 项目概要
   - 功能需求列表
   - 非功能需求列表
   - 约束条件
   - 前提条件

2. 功能列表格式
   - 功能ID
   - 功能名
   - 功能概要
   - 优先级
   - 依赖关系
   - 限制事项

#### markdownlint 规范

- 缩进为2个空格
- 标题层级按顺序使用
- 代码块需指定语言
- 列表使用统一符号
- 换行最多两行
