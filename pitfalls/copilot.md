# GitHub Copilot 陷阱合集

> Copilot 是最老牌的 AI 编程工具，也最容易被低估——以为就是补全，结果 Agent 模式和 MCP 支持都挺深。8 个常见坑。
>
> 踩过新坑？提 Issue 或 PR。

---

## 陷阱 1：补全用了过期的 API

**症状**
- Copilot 生成的代码用了某个库的旧 API
- 你拷进来跑：`Deprecated: use xxx instead` 或直接 `is not a function`
- 升级到库的新版本后，Copilot 还在按旧 API 补全

**根因**
Copilot 的补全基于训练数据 + 当前打开文件的上下文。训练数据截止日期前的库版本可能已经迭代过，而 Copilot 不会读 `package.json` 里的实际版本。

**出坑**
- 把新版文档的关键一页**作为 tab 打开**（Copilot 会读打开的 tab）
- 或在文件顶部加注释：

```typescript
// 使用 TanStack Query v5 API（useQuery 签名：useQuery({ queryKey, queryFn })）
import { useQuery } from '@tanstack/react-query';
```

**预防**
- 在 `.github/copilot-instructions.md` 里声明关键库的版本和 API 风格：

```markdown
## 关键依赖版本
- React 19（使用 useTransition、useOptimistic）
- TanStack Query v5（useQuery 新签名）
- Zod v3（不用 v2 的 object() 链式风格）
```

---

## 陷阱 2：Agent 模式改了 `.env` 或敏感文件

**症状**
- 让 Copilot Agent 改个配置，它顺手扫了 `.env` 把 key 读进上下文
- 在 Chat 窗口里看到你的 API key 明文显示
- 最坏：补全建议里带出了真实的 secret

**根因**
Copilot 的 exclude 配置不是默认启用的。`.gitignore` 里的文件 Copilot 依然会读。`.env`、`.aws/credentials`、`id_rsa` 这类敏感文件如果不显式排除，都在它的感知范围。

**出坑**
立刻在 `.vscode/settings.json` 加排除：

```json
{
  "github.copilot.enable": {
    "*": true,
    "env": false,
    "secrets": false
  },
  "github.copilot.advanced": {
    "exclude": ["**/.env*", "**/secrets/**", "**/*.pem", "**/*.key"]
  }
}
```

如果敏感信息已经进了 Chat 历史：清空 Chat（`Ctrl+L` 或面板里清除），**并轮换相关密钥**。

**预防**
- 项目初始化就配好 exclude，不要等出事
- 用 GitHub 的 Copilot Content Exclusion（企业版）做组织级兜底
- 敏感配置用 `direnv` / `1Password CLI` 等从非项目目录注入，不落地

---

## 陷阱 3：`.github/copilot-instructions.md` 太长被忽略

**症状**
- 你在 instructions 里写了 20 条规则
- Agent 只遵守前几条，后面的像没看过
- 尤其 Chat 窗口里提问时，细节规则几乎不生效

**根因**
copilot-instructions.md 超过 ~150 行时效果显著下降。Copilot 在注入上下文时会做截断，靠后的内容被牺牲。

**出坑**
拆文件：
- 核心不变规则 → `.github/copilot-instructions.md`（< 100 行）
- 场景化规则 → `.github/chatModes/` 下各自的 `.chatmode.md`
- 专项角色 → `.github/agents/` 下各自的 `.agent.md`

**预防**
Instructions 里**只放跨场景的全局规则**（技术栈、命名、禁止事项）。具体场景规则：

```
.github/
├── copilot-instructions.md     # 核心规则
├── chatModes/
│   ├── security-review.md      # 安全审查专用
│   └── write-tests.md          # 写测试专用
└── agents/
    └── migration-helper.md     # 迁移项目专用
```

---

## 陷阱 4：`#file` 引用的 vs `@workspace` 找到的不一致

**症状**
- 你说 `#file:src/api/user.ts 按这个改`
- Copilot 改得基本对，但不完全
- 另一次你用 `@workspace` 让它找 user.ts，它提到了 2 个 user.ts（项目有两处重名）

**根因**
- `#file` 精确引用你给的路径
- `@workspace` 会让 Copilot 自主搜索整个工作区
- 项目里有重名文件时，`@workspace` 可能选错那个

**出坑**
明确路径 > 模糊索引：

```
❌ @workspace 改一下 user.ts 的 register 方法
✅ #file:src/api/v2/user.ts 改一下这里的 register 方法
```

**预防**
- 项目里避免重名文件（`user.ts` × 3 这种结构重构一下）
- 如果必须重名，引用时用完整路径而非文件名
- `@workspace` 只用于**探索**（"项目里有没有 XX"），不用于**指向**（"改 XX"）

---

## 陷阱 5：MCP Server 配置了但没生效

**症状**
- 在 `.vscode/settings.json` 配了 `github.copilot.chat.mcpServers`
- Chat 里该用 MCP 工具时它没用，走了别的路径
- 不报错，就是悄悄没用

**根因**
常见三种：
1. MCP server 启动命令写错（`npx` 路径、参数顺序）
2. MCP server 启动了但 Copilot 版本不支持（需要较新 VS Code + Copilot Chat）
3. server 启动成功但工具 schema 定义有问题，Copilot 认不出

**出坑**
```
# 在 Chat 里直接问
你当前可以调用哪些 MCP 工具？列出来。
```

如果它列不出你配的工具 → MCP 没加载。看 VS Code 的 Output 面板 → Copilot Chat，找错误日志。

**预防**
- 先用官方示例 server（比如 `@modelcontextprotocol/server-filesystem`）验证 MCP 能接通
- 再换自己的 server
- VS Code 和 GitHub Copilot 插件都保持最新版

---

## 陷阱 6：补全在 JetBrains 和 VS Code 行为不一致

**症状**
- 团队里 VS Code 用户的补全质量明显比 JetBrains 用户好
- 同一个 prompt 给的建议不一样
- 共享 `.github/copilot-instructions.md` 后，VS Code 生效，JetBrains 部分生效

**根因**
- VS Code 是 Copilot 的主战场，新功能优先 ship
- JetBrains 插件的 instructions 支持、MCP 支持、Agent 模式都滞后
- 底层模型也可能不一样（JetBrains 插件有时用老模型）

**出坑**
要么全团队统一 IDE，要么接受 JetBrains 体验差一截。

**预防**
- 团队 AI 工具选型时，把 IDE 一致性当硬约束
- JetBrains 用户补 instructions 时，在文件开头加上**复述式核心规则**（即使 instructions 不生效，开头几行总会被注入）

---

## 陷阱 7：免费用户遇到隐形限流

**症状**
- 免费额度（Copilot Free）用得好好的，某天开始补全延迟暴涨
- Chat 请求经常排队或直接拒绝（"已达使用上限"之类的提示）
- 补全变慢、建议数量减少

**根因**
Copilot Free 有每月补全数和 Chat 次数上限。接近上限时会排队/限速，超出后直接拒绝请求。VS Code 里提示不一定醒目，容易忽略。企业版用户也可能遇到组织级配额。

**出坑**
- 在 GitHub 账户 → Copilot 设置页查看用量
- 用量接近上限：升级 Pro / Pro+ / Business，或当月剩余时间依赖其他工具

**预防**
- 高频使用者直接上 Pro（$10/月），别在 Free 上省
- 团队用户和管理员对齐配额和使用策略
- 建立"Copilot 挂了用谁"的 fallback（比如切 Claude Code 或 Cursor）

---

## 陷阱 8：自定义 Chat Mode / Agent 不被发现

**症状**
- 你在 `.github/chatModes/security-review.md` 写了个专家角色
- 在 Chat 里输入 `@security-review`，Copilot 不识别
- 或者识别了但行为和普通 Chat 没区别

**根因**
- 文件名和 frontmatter 里的 `name` 字段不匹配
- 没有 frontmatter 或字段缺失
- VS Code / Copilot 版本太旧不支持自定义 Chat Mode

**出坑**
检查 frontmatter 必备字段：

```markdown
---
name: security-review       # 和文件名一致
description: Security review using OWASP Top 10
---

# 后面是角色内容
```

**预防**
- 拷一个官方示例 Chat Mode 文件，基于它改，别从零写
- 改完重启 VS Code（不是重启 Copilot）
- `.github/chatModes/*.md` 路径固定，别放错目录

---

## 贡献新陷阱

模板见 [claude-code.md 结尾](./claude-code.md#补充遇到新陷阱怎么办)。

---

## 相关方法论

- [Copilot 完整指南](../copilot/README.md)
- [common/security.md](../common/security.md) — AI 编程的安全风险和防护
- [common/context-management.md](../common/context-management.md) — 上下文管理
