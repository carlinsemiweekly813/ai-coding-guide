# Cursor 最佳实践

> Cursor 是基于 VS Code 的 AI IDE，集成了代码补全、Chat、Composer（Agent）三大核心功能。它的优势在于和编辑器的深度集成——选中代码直接对话、在编辑器内实时预览修改。

---

## 核心概念

| 概念 | 说明 | 用途 |
|------|------|------|
| **Tab 补全** | 基于上下文的代码补全 | 日常编码提速 |
| **Chat** | 侧边栏对话，可选中代码提问 | 理解代码、问答 |
| **Composer** | Agent 模式，可跨文件修改 | 复杂任务、重构 |
| **Rules** | `.cursor/rules/*.md` 规则文件 | 控制 AI 行为 |
| **@引用** | `@file` `@folder` `@web` 等 | 精确指定上下文 |
| **Notepads** | 可复用的上下文片段 | 复杂项目的上下文管理 |

---

## 快速上手

### .cursorrules — 项目规则

在项目根目录创建 `.cursorrules` 或 `.cursor/rules/` 下放规则文件：

```markdown
# 项目规则

## 技术栈
- React 18 + TypeScript + Tailwind CSS
- 状态管理：Zustand
- 路由：React Router v6
- 测试：Vitest + Testing Library

## 代码风格
- 函数组件 + Hooks，不用 Class 组件
- Props 用 interface 定义，不用 type
- 文件命名 kebab-case，组件命名 PascalCase
- 每个组件一个文件

## 禁止
- 不要用 any 类型
- 不要用 useEffect 做数据获取，用 TanStack Query
- 不要直接操作 DOM，用 React ref
```

### @ 引用技巧

```
# 引用文件
@src/components/Button.tsx 这个组件的 Props 类型不对，帮我修一下

# 引用文件夹
@src/api/ 这些接口的错误处理不统一，帮我统一改成...

# 引用文档
@https://tanstack.com/query/latest 参考官方文档，帮我把 useEffect 里的数据获取改成 useQuery

# 引用终端
@terminal 看看刚才的报错信息，帮我修
```

---

## 提示词技巧

### 1. Composer 大任务拆解

```
用 Composer 模式。按以下步骤执行：
1. 先读 src/pages/ 下所有页面组件，理解路由结构
2. 创建 src/layouts/DashboardLayout.tsx 统一布局
3. 把所有页面组件改成使用新布局
4. 确保路由配置正确
每步完成后暂停，等我确认再继续。
```

### 2. 选中代码直接对话

选中一段代码后按 `Cmd+L`（macOS）进入 Chat：

```
# 选中一段复杂的正则表达式
解释这个正则在做什么，有没有边界情况没覆盖到？

# 选中一个函数
这个函数的时间复杂度是多少？有更优的写法吗？

# 选中一段样式代码
把这段 CSS 改成 Tailwind 的写法
```

### 3. 用 Notepads 管理复杂上下文

对于大型项目，创建 Notepad 保存常用上下文：

```
Notepad: "API 规范"
---
所有 API 返回格式：
{ code: number, data: T, message: string }

错误码：
- 400: 参数错误
- 401: 未认证
- 403: 无权限
- 500: 服务器错误

认证方式：Bearer Token in Authorization header
```

对话时引用：`@notepad:API规范 按这个格式实现用户列表接口`

---

## 进阶技巧

### Rules 分文件管理

```
.cursor/rules/
├── global.md          # 全局规则（代码风格、命名等）
├── react.md           # React 相关规则
├── api.md             # API 开发规则
├── testing.md         # 测试规则
└── security.md        # 安全规则
```

每个文件可以设置触发条件：

```markdown
---
globs: ["src/components/**/*.tsx"]
---

# React 组件规则
- 所有组件必须有 displayName
- Props 超过 3 个必须用 interface 拆分
- 必须处理 loading 和 error 状态
```

### 用 superpowers-zh 增强 Rules

手动写 Rules 太慢？用 superpowers-zh 一键安装方法论：

```bash
cd /your/project
npx superpowers-zh
# 自动安装到 .cursor/rules/，包含 brainstorming、debugging、verification 等
```

安装后 Cursor 在匹配文件时会自动加载对应的 skill 规则。

### 模型选择策略

| 场景 | 推荐模型 | 原因 |
|------|---------|------|
| Tab 补全 | cursor-small | 速度快，延迟低 |
| 简单问答 | Claude Sonnet | 性价比高 |
| 复杂重构 | Claude Opus | 理解力最强 |
| Composer 大任务 | Claude Opus | 多文件协调能力好 |

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Tab` | 接受补全 |
| `Cmd+L` | 打开 Chat（选中代码自动带入） |
| `Cmd+I` | 打开 Composer |
| `Cmd+K` | 行内编辑（选中代码后） |
| `Cmd+Shift+L` | 把当前文件加入 Chat 上下文 |

---

## 常见陷阱

| 陷阱 | 说明 | 解决 |
|------|------|------|
| 规则太长 | `.cursorrules` 几千行，AI 记不住 | 拆分到 `.cursor/rules/` 用 globs 按需加载 |
| Composer 失控 | Agent 模式改了不该改的文件 | 用 `@file` 限定范围，或在 rules 里标明禁区 |
| 补全太激进 | Tab 补全一次生成太多代码 | 设置里调整补全长度，或按 `Esc` 拒绝 |
| 上下文不够 | AI 不理解项目结构 | 用 `@folder` 引用关键目录，写好 `.cursorrules` |

---

## 配置模板

直接复制到你的项目 `.cursor/rules/` 目录下：

| 模板 | 用途 |
|------|------|
| [global.cursorrules.md](templates/global.cursorrules.md) | 全局规则（代码风格、命名、禁止事项） |
| [api.cursorrules.md](templates/api.cursorrules.md) | API 开发规则（仅在 API 目录下生效） |

### 模型配置

Cursor 的模型选择在设置界面（`Cursor Settings > Models`）中配置，不需要手动编辑 JSON。

---

## 延伸阅读

- [Cursor 官方文档](https://docs.cursor.com)
- [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) — 社区 Rules 集合（38k+ star）
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills 方法论（也支持 Cursor）
