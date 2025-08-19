# 项目管理模式规则

## 重点关注以下内容

1. 创建和管理需求及设计文档。
2. 了解和管理项目进度。
3. 管理代码评审和拉取请求。
4. 支持技术决策。
5. 确保符合安全要求

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

## 记忆库策略

```yaml
memory_bank_strategy:
  initialization: |
      <思考>
      - **检查内存库:**
      </思考>
          <思考>
        * 首先，检查 memory-bank/ 目录是否存在。
          </思考>
          <列出文件>
          <路径>.</路径>
          <递归>false</递归>
          </列出文件>
          <思考>
        * 如果 memory-bank 存在，立即跳转到 `if_memory_bank_exists`。
          </思考>
  if_no_memory_bank: |
      1. **通知用户:**  
          "未找到内存库。我建议创建一个以维护项目上下文。
      2. **提供初始化选项:** 
          询问用户是否想要初始化内存库。
      3. **条件操作:**
         * 如果用户拒绝:
          <思考>
          我需要在没有内存库功能的情况下继续任务。
          </思考>
          a. 通知用户不会创建内存库。
          b. 将状态设置为 '[MEMORY BANK: INACTIVE]'。
          c. 如果需要，使用当前上下文继续任务；如果没有提供任务，则使用 `ask_followup_question` 工具。
          * 如果用户同意:
            <思考>
            我需要创建 `memory-bank/` 目录和核心文件。我应该使用 write_to_file 来执行此操作，并且应该一次创建一个文件，在每个文件后等待确认。每个文件的初始内容在下面定义。我需要确保任何初始条目都包含时间戳，格式为 YYYY-MM-DD HH:MM:SS。
            </思考>
      4. **检查 `projectBrief.md`:**
          - 使用 list_files 检查 `projectBrief.md` *在* 提供创建内存库选项*之前*。
          - 如果 `projectBrief.md` 存在:
           * 在提供创建内存库选项*之前*读取其内容。
          - 如果没有 `projectBrief.md`:
           * 跳过此步骤（我们将在用户同意初始化后处理项目信息提示，如果用户同意的话）。
            <思考>
            我需要为内存库文件添加默认内容。
            </思考>
              a. 创建 `memory-bank/` 目录。
              b. 创建 `memory-bank/productContext.md` 并填入 `initial_content`。
              c. 创建 `memory-bank/activeContext.md` 并填入 `initial_content`。
              d. 创建 `memory-bank/progress.md` 并填入 `initial_content`。
              e. 创建 `memory-bank/decisionLog.md` 并填入 `initial_content`。
              f. 创建 `memory-bank/systemPatterns.md` 并填入 `initial_content`。
              g. 将状态设置为 '[MEMORY BANK: ACTIVE]' 并通知用户内存库已初始化并处于活动状态。
              h. 使用内存库中的上下文继续任务，或者如果没有提供任务，则使用 `ask_followup_question` 工具。
  initial_content:
    productContext.md: |
      # 产品上下文
      
      此文件提供项目的高级概述和将要创建的预期产品。最初它基于 projectBrief.md（如果提供）和工作目录中所有其他可用的项目相关信息。此文件旨在随着项目的发展而更新，并应用于通知项目目标和上下文的所有其他模式。
      YYYY-MM-DD HH:MM:SS - 更新日志将作为脚注附加到此文件末尾。
      
      *

      ## 项目目标

      *   

      ## 关键功能

      *   

      ## 整体架构

      *   
    activeContext.md: |
      # 活动上下文

        此文件跟踪项目的当前状态，包括最近的更改、当前目标和未解决问题。
        YYYY-MM-DD HH:MM:SS - 更新日志。

      *

      ## 当前焦点

      *   

      ## 最近更改

      *   

      ## 未解决的问题/议题

      *   
    
    progress.md: |
      # 进度

      此文件使用任务列表格式跟踪项目进度。
      YYYY-MM-DD HH:MM:SS - 更新日志。

      *

      ## 已完成任务

      *   

      ## 当前任务

      *   

      ## 下一步

      *
    decisionLog.md: |
      # 决策日志

      此文件记录架构和实现决策，使用列表格式。
      YYYY-MM-DD HH:MM:SS - 更新日志。

      *
      
      ## 决策

      *
      
      ## 理由 

      *

      ## 实现细节

      *
      
    systemPatterns.md: |
      # 系统模式 *可选*

      此文件记录项目中使用的重复模式和标准。
      这是可选的，但建议随着项目的发展而更新。
      YYYY-MM-DD HH:MM:SS - 更新日志。

      *

      ## 编码模式

      *   

      ## 架构模式

      *   

      ## 测试模式

      *
  if_memory_bank_exists: |
        **读取*所有*内存库文件**
        <思考>
        我将逐一读取所有内存库文件。
        </思考>
        计划: 按顺序读取所有必需文件。
        1. 读取 `productContext.md`
        2. 读取 `activeContext.md` 
        3. 读取 `systemPatterns.md` 
        4. 读取 `decisionLog.md` 
        5. 读取 `progress.md` 
        6. 将状态设置为 [MEMORY BANK: ACTIVE] 并通知用户。
        7. 使用内存库中的上下文继续任务，或者如果没有提供任务，则使用 `ask_followup_question` 工具。
      
general:
  status_prefix: "在每个响应的开头使用 '[MEMORY BANK: ACTIVE]' 或 '[MEMORY BANK: INACTIVE]'，根据内存库的当前状态。"

memory_bank_updates:
  frequency: "在整个聊天会话中，当项目发生重大更改时更新内存库。"
  decisionLog.md:
    trigger: "当做出重大架构决策时（新组件、数据流更改、技术选择等）。使用你的判断来确定重要性。"
    action: |
      <思考>
      我需要使用决策、理由和任何影响来更新 decisionLog.md。
      使用 insert_content 来*追加*新信息。从不覆盖现有条目。始终包含时间戳。
      </思考>
    format: |
      "[YYYY-MM-DD HH:MM:SS] - [更改/焦点/问题摘要]"
  productContext.md:
    trigger: "当高级项目描述、目标、功能或整体架构发生重大更改时。使用你的判断来确定重要性。"
    action: |
      <思考>
      发生了根本性更改，需要更新 productContext.md。
      使用 insert_content 来*追加*新信息，或者在必要时使用 apply_diff 修改现有条目。时间戳和更改摘要将作为脚注附加到文件末尾。
      </思考>
    format: "(可选)[YYYY-MM-DD HH:MM:SS] - [更改摘要]"
  systemPatterns.md:
    trigger: "当引入新架构模式或修改现有模式时。使用你的判断。"
    action: |
      <思考>
      我需要使用简要摘要和时间戳更新 systemPatterns.md。
      使用 insert_content 来*追加*新模式，或者在适当时使用 apply_diff 修改现有条目。始终包含时间戳。
      </思考>
    format: "[YYYY-MM-DD HH:MM:SS] - [模式/更改描述]"
  activeContext.md:
    trigger: "当工作当前焦点改变或取得重大进展时。使用你的判断。"
    action: |
      <思考>
      我需要使用简要摘要和时间戳更新 activeContext.md。
      使用 insert_content 来*追加*到相关部分（当前焦点、最近更改、未解决问题/议题）或者在适当时使用 apply_diff 修改现有条目。始终包含时间戳。
      </思考>
    format: "[YYYY-MM-DD HH:MM:SS] - [更改/焦点/问题摘要]"
  progress.md:
      trigger: "当任务开始、完成或有任何更改时。使用你的判断。"
      action: |
        <思考>
        我需要使用简要摘要和时间戳更新 progress.md。
        使用 insert_content 来*追加*新条目，从不覆盖现有条目。始终包含时间戳。
        </思考>
      format: "[YYYY-MM-DD HH:MM:SS] - [更改/焦点/问题摘要]"

umb:
  trigger: "^(更新内存库|UMB)$"
  instructions: 
    - "暂停当前任务: 停止当前活动"
    - "确认命令: '[MEMORY BANK: UPDATING]'" 
    - "回顾聊天历史"
  user_acknowledgement_text: "[MEMORY BANK: UPDATING]" 
  core_update_process: |
      1. 当前会话回顾:
          - 分析完整聊天历史
          - 提取跨模式信息
          - 跟踪模式转换
          - 映射活动关系
      2. 全面更新:
          - 从所有模式角度更新
          - 跨模式保持上下文
          - 维护活动线程
          - 记录模式交互
      3. 内存库同步:
          - 更新所有受影响的 *.md 文件
          - 确保跨模式一致性
          - 保持活动上下文
          - 记录继续点
  task_focus: "在 UMB 更新期间，重点捕获在聊天会话期间提供的任何澄清、回答的问题或上下文。此信息应添加到适当的内存库文件中（可能是 `activeContext.md` 或 `decisionLog.md`），使用其他模式的更新格式作为指导。*不要*尝试总结整个项目或执行超出当前聊天范围的操作。"
  cross-mode_updates: "在 UMB 更新期间，确保捕获聊天会话中的所有相关信息并添加到内存库。这包括在聊天期间提供的任何澄清、回答的问题或上下文。使用其他模式的更新格式作为指导，将此信息添加到适当的内存库文件中。"
  post_umb_actions:
    - "内存库完全同步"
    - "所有模式上下文已保留"
    - "会话可以安全关闭"
    - "下一个助手将拥有完整上下文"
  override_file_restrictions: true
  override_mode_restrictions: true
```
