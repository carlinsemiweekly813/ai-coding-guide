# Trae 最佳实践

> Trae 是字节跳动推出的 AI IDE（基于 VS Code），主打**免费使用 Claude 和 GPT 模型**。对中国开发者友好——界面支持中文、国内网络直连、免费额度充足。适合入门 AI 编程和预算敏感的团队。

---

## 核心概念

| 概念 | 说明 | 用途 |
|------|------|------|
| **Builder** | Agent 模式，自主完成任务 | 复杂任务、跨文件修改 |
| **Chat** | 侧边栏对话 | 问答、理解代码 |
| **补全** | 行内代码补全 | 日常编码 |
| **Rules** | `.trae/rules/` | 项目级规则 |
| **@引用** | `@file` `@folder` `@web` | 精确指定上下文 |

---

## 快速上手

### 安装

从 [trae.ai](https://trae.ai) 下载安装，支持 macOS / Windows / Linux。

注册后直接可用，**不需要自己的 API Key**。

### Rules — 项目规则

创建 `.trae/rules/project_rules.md`：

```markdown
# 项目规则

## 技术栈
React + TypeScript + Ant Design + UmiJS

## 代码规范
- 使用函数组件 + Hooks
- 状态管理用 @umijs/max 内置的 Model
- 请求用 umi-request，不要直接用 fetch/axios
- 样式用 CSS Modules

## 中文规范
- 注释用中文
- commit message 用中文
- 变量名用英文
```

### Builder — Agent 模式

Builder 是 Trae 的 Agent，类似 Cursor 的 Composer：

```
用 Builder 模式。
参考 src/pages/user/list.tsx 的写法，
新建一个 src/pages/order/list.tsx 订单列表页。
要求：
1. 表格用 Ant Design ProTable
2. 支持按日期、状态、金额筛选
3. 支持导出 Excel
4. 操作列：查看详情、取消订单、退款
```

---

## 提示词技巧

### 1. 利用免费模型

Trae 免费提供 Claude 和 GPT，合理分配：

```
# 复杂任务 — 用 Claude
Builder 模式 + Claude：重构整个认证模块

# 简单任务 — 用 GPT
Chat 模式 + GPT：解释这段代码 / 写个注释
```

### 2. 中文友好

Trae 对中文支持最好，可以完全用中文交互：

```
帮我把 src/utils/request.ts 的请求封装改一下：
1. 加上统一的 loading 状态管理
2. 错误提示用 Ant Design 的 message 组件
3. 401 错误自动跳转登录页
4. 网络超时设为 10 秒
```

### 3. Ant Design 生态

Trae 对国内常用组件库支持好：

```
@https://ant-design.antgroup.com/components/table-cn
参考 Ant Design 官方文档，
给 ProTable 加一个自定义的行展开功能，
展开后显示订单的商品明细。
```

---

## 与 Cursor 的区别

| 维度 | Trae | Cursor |
|------|------|--------|
| 价格 | **免费**（含 Claude/GPT） | $20/月 |
| 中文支持 | ★★★（界面中文） | ★☆☆（纯英文） |
| 国内网络 | ★★★（直连） | ★☆☆（需要代理） |
| Agent 能力 | ★★☆ | ★★★ |
| Rules 系统 | ★★☆ | ★★★（globs 按需加载） |
| 插件生态 | ★★☆（VS Code 兼容） | ★★★（VS Code 兼容） |
| 适合 | 入门、预算敏感、国内团队 | 进阶、愿意付费、追求最强 |

---

## 常见陷阱

| 陷阱 | 说明 | 解决 |
|------|------|------|
| Builder 执行慢 | 免费模型可能排队 | 非紧急任务用 Builder，急的用 Chat |
| 模型切换 | 不同模型擅长不同事 | 复杂用 Claude，简单用 GPT |
| Rules 不生效 | 文件路径或格式不对 | 确保在 `.trae/rules/` 下，Markdown 格式 |

---

## 延伸阅读

- [Trae 官方网站](https://trae.ai)
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills 方法论（也支持 Trae）
