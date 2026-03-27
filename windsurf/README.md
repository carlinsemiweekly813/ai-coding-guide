# Windsurf 最佳实践

> Windsurf 是 Codeium 推出的 AI IDE，核心特色是 **Cascade** — 一个能感知你编辑行为的 AI Agent。它会自动追踪你的操作（文件切换、代码修改、终端输出），主动提供帮助，而不是等你提问。

---

## 核心概念

| 概念 | 说明 | 用途 |
|------|------|------|
| **Cascade** | 上下文感知 Agent | 自动理解你在做什么，主动建议 |
| **Flows** | 操作流追踪 | 记录你的编辑轨迹作为上下文 |
| **Write 模式** | 直接写代码 | 类似 Cursor Composer |
| **Chat 模式** | 对话问答 | 理解代码、问问题 |
| **Rules** | `.windsurfrules` | 项目级规则配置 |
| **@引用** | `@file` `@folder` `@web` 等 | 精确指定上下文 |

---

## 快速上手

### 安装

从 [windsurf.com](https://windsurf.com) 下载安装，支持 macOS / Windows / Linux。基于 VS Code，现有 VS Code 扩展大部分兼容。

### .windsurfrules — 项目规则

在项目根目录创建 `.windsurfrules`：

```markdown
# 项目规则

## 技术栈
Vue 3 + TypeScript + Pinia + Element Plus

## 代码规范
- 使用 Composition API（setup 语法糖）
- 组件用 PascalCase，文件用 kebab-case
- Store 按功能模块拆分
- API 请求统一走 src/api/ 下的封装

## Cascade 行为
- 修改组件时自动检查 Props 类型是否正确
- 修改 API 接口时提醒更新对应的 TypeScript 类型
- 不要自动重构我没要求改的代码
```

### Cascade 的正确用法

Cascade 会追踪你的操作。利用这一点：

```
1. 先手动打开几个相关文件（让 Cascade 知道你在关注什么）
2. 做一个小修改（让 Cascade 理解你的意图）
3. 这时候 Cascade 的建议会比直接提问更准确
```

---

## 提示词技巧

### 1. Write 模式 — 大任务

```
用 Write 模式。
把 src/views/user/ 下的用户管理页面从 Options API 迁移到 Composition API。
保持功能不变，只改写法。
一个文件一个文件来，每个文件改完让我确认。
```

### 2. 利用 Flows 上下文

```
# 你刚在终端跑了测试，看到了报错
# Cascade 已经知道了，直接说：
刚才测试报错了，帮我看看什么原因。
# 不需要贴报错信息，Cascade 已经从 Flow 里拿到了
```

### 3. @ 引用

```
@src/api/user.ts @src/types/user.ts
这两个文件的类型定义不一致，帮我统一。
以 types/user.ts 为准。
```

---

## 与 Cursor 的区别

| 维度 | Windsurf | Cursor |
|------|---------|--------|
| 核心理念 | **主动感知**，追踪你的操作自动提供帮助 | **按需触发**，你提问它才响应 |
| Agent | Cascade（自动追踪 Flows） | Composer（手动触发） |
| 上下文 | 自动从操作流推断 | 需要手动用 @ 引用 |
| Rules | `.windsurfrules` 单文件 | `.cursor/rules/` 可按 globs 拆分 |
| 模型 | 自研 + Claude/GPT | Claude/GPT/自研 |
| 适合 | 喜欢 AI 主动帮忙的人 | 喜欢精确控制的人 |

---

## 常见陷阱

| 陷阱 | 说明 | 解决 |
|------|------|------|
| Cascade 太主动 | 你只是浏览代码它就开始建议修改 | 设置里调整 Cascade 敏感度 |
| Flows 上下文错乱 | 切了太多文件，Cascade 搞不清你在做什么 | 开新 Cascade 会话，重新开始 |
| Write 模式范围失控 | 改了不该改的文件 | 用 @ 引用限定范围 |

---

## 延伸阅读

- [Windsurf 官方文档](https://docs.codeium.com/windsurf)
- [awesome-windsurf](https://github.com/detailobsessed/awesome-windsurf) — 社区资源集合
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills 方法论（也支持 Windsurf）
