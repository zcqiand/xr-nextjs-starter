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

#### 文件结构

- 保持与组件相同的目录结构
- 文件名为 `[ComponentName].spec.ts`
- 按功能分组测试用例

#### 测试用例设计

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

#### 文件结构

- 保持与仓库相同的目录结构
- 文件名为 `[RepositoryName].spec.ts`
- 按 CRUD 操作分组测试用例

#### 测试用例设计

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
