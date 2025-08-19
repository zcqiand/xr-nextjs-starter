# 代码模式规则

## 期望的回答

- 实现代码必须完整提供，不得省略
- 包含 TypeScript 的类型定义
- 遵循安全最佳实践进行实现
- 提供考虑响应式设计的 UI/UX 建议
- 用日文详细说明（注：如需中文可改为“用中文详细说明”）
- 组件设计需便于测试
- git commit 时需通过 lefthook 的 pre-commit 钩子
- 修改代码文档时务必同步更新相关文档

## 应用程序编码规范

- 遵循现有的 ESLint、Prettier 设置
- 组件设计采用 Composition API
- 函数和组件需包含适当的注释
- 使用 Next.js 的 SuspenseBoundary 管理加载状态
- 逻辑与组件分离，提取为工具函数
- 为提高逻辑复用性，使用函数式组件
- 为提高组件复用性，充分利用插槽
- 组件设计需便于测试
- 分离副作用

### 命名规则

- 目录：kebab-case（短横线命名）
- 文件名：kebab-case（短横线命名）
- 组件名：PascalCase（帕斯卡命名）
- 变量名：camelCase（驼峰命名）
- 常量名：SNAKE_CASE（下划线命名）
- React hook 名：camelCase（驼峰命名）
- CSS 类名：使用 Tailwind CSS 的类名

### Next.js 特有规则

- 避免 “params should be awaited” 错误，必须 await params。例如：`const { id } = await params;`
- params 必须以 Promise 形式接收。例如：`{ params: Promise<{ id: string }> }`

## 组件实现规范

### 按组件类型的规范

#### 按钮类

- 基于 shadcn/ui 的按钮组件实现
- 点击处理函数命名为 handle[Action]Click 形式
- 实现 disabled 状态的视觉反馈（btn-disabled）
- 统一 loading 状态表现（loading 属性）
- 按照按钮类型正确使用样式（btn-primary, btn-ghost 等）
- 点击事件需通过 TypeScript 保证类型安全

#### 模态框类

- 基于 shadcn/ui 的 modal 组件实现
- 通过 isVisible 属性控制显示
- 实现焦点陷阱
- 支持键盘操作（Escape 键）

#### 列表类

- 基于 shadcn/ui 的 table 组件实现
- 实现分页功能
- 统一实现排序与筛选功能
- 统一空状态展示（empty-state）
- 统一 loading 状态展示
- 实现自定义错误处理函数进行错误处理

### 错误处理

- 使用 Next.js 的 ErrorBoundary 进行错误处理
  - 使用 try-catch 进行适当的错误处理
- 显示对用户友好的错误信息
- 记录错误日志
  - 使用 Pino 记录错误日志

## 测试实现规范

- 测试用例采用 AAA 模式编写

### 组件测试

#### 组件测试文件结构

- 保持与组件相同的目录结构
- 文件名为 `[ComponentName].spec.ts`
- 按功能分组测试用例

#### 组件测试用例设计

- 验证组件挂载状态
- 检查 Props、事件、插槽的行为
- 验证条件分支下的显示/隐藏
- 用户交互测试
- 错误状态处理
- 避免通过 wrapper.vm 检查组件内部实现

#### 测试数据

- 使用 Factory 模式生成测试数据
- 准备贴近实际的测试数据
- 测试边界值与异常值

#### 测试数据生成与 Mock 处理

- 每个测试用例需能清楚看到测试数据生成过程
- 避免统一生成测试数据和 Mock 处理

### 仓库测试

#### 仓库测试文件结构

- 保持与仓库相同的目录结构
- 文件名为 `[RepositoryName].spec.ts`
- 按 CRUD 操作分组测试用例

#### 仓库测试用例设计

- 验证基本 CRUD 操作
- 全面测试错误情况
- 校验数据一致性
- 验证包含关联关系的查询

#### 测试数据管理

- 创建和删除测试数据
- 确保清理过程彻底执行

### 测试数据生成及 Mock 处理规则

#### 明确数据作用域

- 全局数据（Factory 生成的基础数据）放在最顶层 describe 块前
- 针对具体测试用例的数据在各用例内定义
- 只在特定测试组使用的数据在对应 describe 块内定义

#### Mock 处理实现

- Mock 函数用 `vi.hoisted` 定义

```ts
const { fetchFromRepository, validateUtil } = vi.hoisted(() => ({
  fetchFromRepository: vi.fn(),
  validateUtil: vi.fn()
}))
```

#### 仓库的 Mock

- 必须以 `{ data, error }` 格式返回
- 无错误时明确设置 `error: null`
- 有错误时用字符串或 `{ message: string }` 返回

```ts
// 成功情况
repositoryMock.mockResolvedValue({ data: result, error: null })

// 错误情况
repositoryMock.mockResolvedValue({
  data: null,
  error: { message: '数据获取失败' }
})
```

#### 工具类的 Mock

- 返回值格式根据函数实现自由决定
- 按类型定义返回值

```ts
// 成功情况
validateUtil.mockResolvedValue(true)
formatUtil.mockReturnValue('formatted text')
calculateUtil.mockReturnValue(100)

// 错误情况
validateUtil.mockRejectedValue(new Error('验证错误'))
```

## 代码变更后的检查

1. 检查构建：npm run build

2. 执行变更文件的单元测试

- 测试文件命名规则: `[FileName].spec.ts`
- 测试文件目录: `src/spec/` 下对应目录
  - 组件: `src/spec/components/`
  - 工具类: `src/spec/utils/`
  - 仓库: `src/spec/repositories/`

示例：

### 执行指定测试文件

```bash
npm run test:unit src/spec/utils/example.spec.ts
```

### 执行指定目录下所有测试

```bash
npm run test:unit src/spec/utils/
```

注意：

- 测试文件需对应变更的源码文件
- 连续测试失败时，应向用户报告问题并等待指示

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
