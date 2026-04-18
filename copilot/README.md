# GitHub Copilot 最佳实践

> GitHub Copilot 是 VS Code / JetBrains 内置的 AI 编程助手。它的核心优势是**无缝集成** — 不需要切换工具，在编辑器里自然地写代码就有补全和建议。Copilot Agent 模式让它从补全工具进化成了能独立完成任务的 Agent。

---

## 核心概念

| 概念 | 说明 | 用途 |
|------|------|------|
| **代码补全** | 行内灰色建议 | 日常编码，按 Tab 接受 |
| **Chat** | 侧边栏对话 `Cmd+Shift+I` | 问答、解释代码 |
| **Agent 模式** | 自主完成多步骤任务 | 复杂任务、跨文件修改 |
| **Copilot Instructions** | `.github/copilot-instructions.md` | 项目级配置 |
| **Chat Modes** | 自定义对话角色 | 安全审查、测试专家等 |
| **MCP Server** | 扩展 Copilot 的工具能力 | 连接数据库、API 等 |
| **#引用** | `#file` `#selection` `#terminal` | 精确指定上下文 |

---

## 快速上手

### Copilot Instructions — 项目配置

在项目根目录创建 `.github/copilot-instructions.md`：

```markdown
# 项目指引

## 技术栈
- Python 3.12 + FastAPI + SQLAlchemy 2.0
- 数据库：PostgreSQL
- 缓存：Redis
- 测试：pytest

## 代码规范
- 类型注解必须完整，使用 Python 3.10+ 语法
- async 函数统一加 `async_` 前缀
- 所有接口必须有 Pydantic 模型做入参校验
- 错误用自定义异常类，不要用裸 Exception

## 目录约定
- src/api/     — FastAPI 路由
- src/models/  — SQLAlchemy 模型
- src/schemas/ — Pydantic 模型
- src/services/ — 业务逻辑
- tests/       — 测试（镜像 src 结构）
```

### # 引用技巧

```
# 引用文件
#file:src/models/user.py 基于这个模型写一个用户注册接口

# 引用选中代码
#selection 这段代码有什么性能问题？

# 引用终端输出
#terminal 看看报错信息帮我修

# 引用 VS Code 问题面板
#problems 帮我修复这些类型错误
```

---

## Agent 模式

Copilot Agent 是 2025 年最大的更新。从对话框输入任务，Agent 会：

1. 分析需求 → 2. 搜索相关文件 → 3. 制定计划 → 4. 逐步执行 → 5. 运行测试验证

### 使用 Agent 模式

在 Chat 中选择 Agent 模式（或直接描述复杂任务）：

```
@workspace 给 src/api/ 下所有接口加上 rate limiting，
使用 Redis 做计数器，每个用户每分钟最多 60 次请求。
需要：
1. 一个可复用的 rate_limit 装饰器
2. Redis 连接配置
3. 超限时返回 429 状态码
4. 给每个接口加上测试
```

### Agent + MCP 扩展能力

通过 MCP Server 让 Agent 访问外部工具：

```json
// .vscode/settings.json
{
  "github.copilot.chat.mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-postgres", "postgresql://..."]
    }
  }
}
```

然后你可以说：

```
查一下数据库里 users 表的结构，
然后根据实际字段生成 SQLAlchemy 模型和 Pydantic schema。
```

---

## 提示词技巧

### 1. 写好注释让补全更准

```python
# 补全质量取决于上下文。写一行注释，补全就知道你要什么：

# 计算两个日期之间的工作日天数，排除周末和中国法定节假日
def count_business_days(start: date, end: date) -> int:
    # Copilot 会自动补全完整实现
```

### 2. 用自定义 Agent 做专项审查

创建 `.github/agents/security-reviewer.agent.md`：

```markdown
---
name: 安全审查员
description: 按 OWASP Top 10 审查代码安全
---

你是一名安全审查专家。审查代码时：
1. 按 OWASP Top 10 逐项检查
2. 关注 SQL 注入、XSS、CSRF、权限绕过
3. 检查敏感数据是否明文存储或日志泄露
4. 标注风险等级：🔴 严重 / 🟡 中等 / 🟢 低
```

在 Chat 中用 `@security-reviewer` 调用这个角色。

### 3. 利用 Workspace 全局理解

```
@workspace 项目里有没有硬编码的密钥或敏感信息？
帮我全部找出来，改成环境变量。
```

---

## 进阶技巧

### 自定义指令文件

Copilot 支持通过指令文件细化行为：

- **`.github/copilot-instructions.md`** — 全局项目指令
- **`.github/chatModes/`** — 自定义 Chat 角色（如安全审查员、测试专家）

如果你用 superpowers-zh，也可以在 `.claude/skills/` 下的 skill 文件里写方法论，Copilot Agent 的 `@workspace` 命令会读取项目中的 markdown 文件。

### VS Code 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Tab` | 接受补全 |
| `Esc` | 拒绝补全 |
| `Cmd+Shift+I` | 打开 Copilot Chat |
| `Cmd+I` | 行内编辑 |
| `Alt+]` / `Alt+[` | 切换不同补全建议 |

### 补全优化

让 Copilot 补全更准确的几个技巧：

1. **打开相关文件** — Copilot 会读取当前打开的 tab 作为上下文
2. **写好类型注解** — 类型越完整，补全越准
3. **函数签名先写好** — 先写好函数名、参数、返回类型，再让它补全函数体
4. **保持文件简短** — 大文件上下文噪音多，Copilot 容易跑偏

---

## 常见陷阱

| 陷阱 | 说明 | 解决 |
|------|------|------|
| 补全过时 API | Copilot 用了过期的库 API | 打开库的源码或文档作为 tab 上下文 |
| Agent 改错文件 | 多文件修改时动了不该动的 | 用 `#file` 限定范围 |
| 忽略 .gitignore | Copilot 可能读取 node_modules | 检查设置里的 exclude 配置 |
| Instructions 太长 | 超过模型上下文窗口 | 精简到关键规则，详细规则拆到 Chat Modes |

👉 **深度展开版**：[Copilot 陷阱合集](../pitfalls/copilot.md) — 8 个真实踩坑场景（Agent 读 .env / MCP 失效 / 免费限流 等），每个带症状 / 根因 / 出坑 / 预防

---

## 配置模板

直接复制到你的项目里用：

| 模板 | 用途 |
|------|------|
| [copilot-instructions.md](templates/copilot-instructions.md) | 项目指引模板，复制到 `.github/copilot-instructions.md` |
| [security-reviewer.agent.md](templates/security-reviewer.agent.md) | 安全审查角色，复制到 `.github/agents/` |

---

## 延伸阅读

- [Copilot 官方文档](https://docs.github.com/en/copilot)
- [awesome-copilot](https://github.com/github/awesome-copilot) — 官方资源集合（27k+ star）
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills 方法论（也支持 VS Code Copilot）
