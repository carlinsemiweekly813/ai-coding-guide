# Aider 最佳实践

> Aider 是一个开源的 CLI AI 编程工具，核心特色是 **Git 原生** — 每次修改自动提交，天然支持版本控制。支持几乎所有主流 LLM（Claude、GPT、DeepSeek、Qwen、本地模型），是最灵活的 AI 编程 CLI。

---

## 核心概念

| 概念 | 说明 | 用途 |
|------|------|------|
| **Chat 模式** | `code` / `ask` / `architect` | 不同任务用不同模式 |
| **自动 Git 提交** | 每次修改自动 commit | 随时可以回滚 |
| **Map 模式** | 自动索引项目结构 | 理解大代码库 |
| **多模型支持** | Claude/GPT/DeepSeek/Ollama 等 | 灵活选择，控制成本 |
| **Lint & Test** | 内置代码检查和测试 | 修改后自动验证 |

---

## 快速上手

### 安装

```bash
pip install aider-chat

# 设置 API Key（选一个）
export ANTHROPIC_API_KEY=sk-xxx    # Claude
export OPENAI_API_KEY=sk-xxx       # GPT
export DEEPSEEK_API_KEY=sk-xxx     # DeepSeek
```

### 基本使用

```bash
cd /your/project
aider

# 指定模型
aider --model claude-3-5-sonnet

# 用 DeepSeek（便宜）
aider --model deepseek/deepseek-chat

# 用本地模型（免费）
aider --model ollama/qwen2.5-coder
```

### 三种 Chat 模式

```bash
# code 模式（默认）— 直接修改代码
/code 给 utils.py 加一个 retry 装饰器

# ask 模式 — 只问不改
/ask 这个函数的时间复杂度是多少？

# architect 模式 — 先设计再实现
/architect 设计一个任务队列系统，先给方案不要写代码
```

---

## 提示词技巧

### 1. 添加文件到上下文

```bash
# 手动添加文件
/add src/models/user.py src/api/auth.py

# 添加整个目录
/add src/services/

# 移除不需要的文件
/drop src/legacy/old_auth.py
```

Aider 只会修改已添加的文件。这是精确控制范围的方式。

### 2. Architect 模式做大任务

```
/architect

我要给项目加一个 WebSocket 实时通知系统。
要求：
1. 用户上线/下线通知
2. 新消息实时推送
3. 支持频道订阅/退订
4. 需要考虑断线重连

先给出技术方案，包括：
- 用什么库
- 数据流设计
- 需要新建哪些文件
- 需要修改哪些现有文件

确认后切到 code 模式实现。
```

### 3. 利用自动 Git 提交

```bash
# 每次修改都有独立 commit，随时回滚
git log --oneline  # 看 Aider 的提交记录
git diff HEAD~1    # 看最后一次修改
git revert HEAD    # 不满意？一键回滚

# 设置自定义提交前缀
aider --commit-prefix "[ai] "
```

### 4. 多模型策略

```bash
# 复杂架构设计 — 用最强模型
aider --model claude-3-5-sonnet
/architect 设计微服务拆分方案

# 日常编码 — 用性价比模型
aider --model deepseek/deepseek-chat
/code 按方案实现 user-service

# 代码审查 — 用免费本地模型
aider --model ollama/qwen2.5-coder
/ask 看看这段代码有没有问题
```

---

## 进阶技巧

### .aider.conf.yml — 项目配置

```yaml
# .aider.conf.yml
model: claude-3-5-sonnet
auto-commits: true
auto-lint: true
auto-test: true
test-cmd: pytest
lint-cmd: ruff check
```

### Lint + Test 自动化

```yaml
# 每次修改后自动：
# 1. 跑 linter 检查
# 2. 跑相关测试
# 3. 如果失败，自动修复再重试
auto-lint: true
auto-test: true
lint-cmd: "ruff check --fix"
test-cmd: "pytest -x"
```

### 与 Git 工作流集成

```bash
# 在 feature 分支上工作
git checkout -b feature/add-notifications
aider

# Aider 的每次修改都在这个分支上
# 完成后正常走 PR 流程
git push -u origin feature/add-notifications
```

---

## 与其他 CLI 工具的区别

| 维度 | Aider | Claude Code | Gemini CLI |
|------|-------|-------------|------------|
| Git 集成 | ★★★（自动提交） | ★★☆（手动） | ★☆☆ |
| 模型灵活性 | ★★★（几乎所有 LLM） | ★☆☆（仅 Claude） | ★☆☆（仅 Gemini） |
| Agent 能力 | ★★☆ | ★★★ | ★★☆ |
| 上下文管理 | 手动 /add /drop | 自动 | 自动 |
| 开源 | ✅ 完全开源 | ❌ | ✅ 开源 |
| 成本控制 | ★★★（可用免费模型） | ★☆☆ | ★★★ |
| 适合 | 灵活、省钱、Git 重度用户 | 复杂 Agent 任务 | 大代码库分析 |

---

## 配置模板

| 模板 | 用途 |
|------|------|
| [.aider.conf.yml](templates/aider.conf.yml) | 项目配置模板（含多模型、lint、test 配置），复制到项目根目录 |

---

## 延伸阅读

- [Aider 官方文档](https://aider.chat/docs/)
- [Aider GitHub](https://github.com/Aider-AI/aider)（42k+ star）
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills 方法论（也支持 Aider）
