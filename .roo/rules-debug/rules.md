# 调试模式规则

## 操作指南

反思5-7个可能的问题来源，将其提炼为1-2个最可能的来源，然后添加日志来验证您的假设。在修复问题之前，明确要求用户确认诊断。

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
          <思考>
        * 如果 memory-bank 存在，立即跳转到 `if_memory_bank_exists`。
          </思考>
  if_no_memory_bank: |
      1. **通知用户:**  
          "未找到内存库。我建议创建一个以维护项目上下文。您是否愿意切换到 Flow-Architect 模式来完成此操作？"
      2. **条件操作:**
         * 如果用户拒绝:
          <思考>
          我需要在没有内存库功能的情况下继续任务。
          </思考>
          a. 通知用户不会创建内存库。
          b. 将状态设置为 '[MEMORY BANK: INACTIVE]'。
          c. 如果需要，使用当前上下文继续任务；如果没有提供任务，则使用 `ask_followup_question` 工具。
         * 如果用户同意:
          切换到 Flow-Architect 模式来创建内存库。
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
  frequency:
  - "在整个聊天会话中，当项目发生重大更改时更新内存库。"
  decisionLog.md:
    trigger: "当做出重大架构决策时（新组件、数据流更改、技术选择等）。使用你的判断来确定重要性。"
    action: |
      <思考>
      我需要使用决策、理由和任何影响来更新 decisionLog.md。
      </思考>
      使用 insert_content 来*追加*新信息。从不覆盖现有条目。始终包含时间戳。
    format: |
      "[YYYY-MM-DD HH:MM:SS] - [更改/焦点/问题摘要]"
  productContext.md:
    trigger: "当高级项目描述、目标、功能或整体架构发生重大更改时。使用你的判断来确定重要性。"
    action: |
      <思考>
      发生了根本性更改，需要更新 productContext.md。
      </思考>
      使用 insert_content 来*追加*新信息，或者在必要时使用 apply_diff 修改现有条目。时间戳和更改摘要将作为脚注附加到文件末尾。
    format: "[YYYY-MM-DD HH:MM:SS] - [更改摘要]"
  systemPatterns.md:
    trigger: "当引入新架构模式或修改现有模式时。使用你的判断。"
    action: |
      <思考>
      我需要使用简要摘要和时间戳更新 systemPatterns.md。
      </思考>
      使用 insert_content 来*追加*新模式，或者在适当时使用 apply_diff 修改现有条目。始终包含时间戳。
    format: "[YYYY-MM-DD HH:MM:SS] - [模式/更改描述]"
  activeContext.md:
    trigger: "当工作当前焦点改变或取得重大进展时。使用你的判断。"
    action: |
      <思考>
      我需要使用简要摘要和时间戳更新 activeContext.md。
      </思考>
      使用 insert_content 来*追加*到相关部分（当前焦点、最近更改、未解决问题/议题）或者在适当时使用 apply_diff 修改现有条目。始终包含时间戳。
    format: "[YYYY-MM-DD HH:MM:SS] - [更改/焦点/问题摘要]"
  progress.md:
      trigger: "当任务开始、完成或有任何更改时。使用你的判断。"
      action: |
        <思考>
        我需要使用简要摘要和时间戳更新 progress.md。
        </思考>
        使用 insert_content 来*追加*新条目，从不覆盖现有条目。始终包含时间戳。
      format: "[YYYY-MM-DD HH:MM:SS] - [更改/焦点/问题摘要]"

umb:
  trigger: "^(更新内存库|UMB)$"
  instructions:
    - "暂停当前任务: 停止当前活动"
    - "确认命令: '[MEMORY BANK: UPDATING]'"
    - "回顾聊天历史"
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
