# Kiro 最佳实践

> Kiro 是 AWS 推出的 AI IDE，核心特色是 **Spec-driven Development**（规格驱动开发）。不是让 AI 直接写代码，而是先生成需求规格、设计文档，确认后再按规格实现。适合团队协作和需要高质量交付的场景。

---

## 核心概念

| 概念 | 说明 | 用途 |
|------|------|------|
| **Spec** | 需求规格文档 | AI 先写规格，确认后再写代码 |
| **Steering** | `.kiro/steering/*.md` | 项目级规则和指引 |
| **Hooks** | 自动化触发器 | 文件保存时自动验证/测试 |
| **Agent** | 后台自主执行 | 复杂任务自动完成 |

---

## 快速上手

### 安装

从 [kiro.dev](https://kiro.dev) 下载安装。需要 AWS 账号或 GitHub 账号登录。目前处于预览阶段。

### Steering 文件 — 项目配置

在 `.kiro/steering/` 下创建规则文件：

```markdown
---
mode: always
---

# 项目规则

## 技术栈
Java 17 + Spring Boot 3 + MyBatis Plus + MySQL 8

## 代码规范
- Controller 层只做参数校验和路由
- Service 层处理业务逻辑
- DAO 层只做数据库操作
- 统一用 Result<T> 包装返回值
- 异常用自定义 BusinessException

## 命名规范
- 包名全小写
- 类名 PascalCase
- 方法名 camelCase
- 常量 UPPER_SNAKE_CASE
```

Steering 支持三种模式：
- `always` — 每次对话都加载
- `globs: ["*.java"]` — 只在操作匹配文件时加载
- `manual` — 手动激活

### Spec 驱动开发

Kiro 的工作流和其他工具不同：

```
1. 你描述需求
2. Kiro 生成 Spec（需求规格 + 技术设计 + 测试用例）
3. 你审查和修改 Spec
4. 确认后 Kiro 按 Spec 逐步实现
5. 每步实现后自动运行相关测试
```

这比"直接写代码"慢，但产出质量更高，特别适合：
- 团队协作（Spec 可以 review）
- 复杂功能（先想清楚再动手）
- 需要文档的项目

---

## 提示词技巧

### 1. 让 Spec 更精确

```
给订单模块加一个退款功能。

业务规则：
- 订单支付后 7 天内可退款
- 部分退款和全额退款都支持
- 退款需要审批（金额 > 500 元）
- 退款到原支付渠道

请先生成 Spec，我确认后再实现。
```

### 2. 利用 Hooks 自动验证

```json
// .kiro/hooks.json
{
  "on-save": {
    "*.java": "mvn compile -q",
    "*.test.java": "mvn test -pl ${module} -q"
  }
}
```

每次保存 Java 文件自动编译，保存测试文件自动跑测试。

### 3. Steering 按模块配置

```
.kiro/steering/
├── always.md              # 全局规则（always）
├── api.md                 # API 规则（globs: src/controller/**）
├── database.md            # 数据库规则（globs: src/mapper/**）
└── testing.md             # 测试规则（globs: src/test/**）
```

---

## 与其他工具的区别

| 维度 | Kiro | Claude Code | Cursor |
|------|------|-------------|--------|
| 核心理念 | Spec 先行 | Agent 执行 | IDE 补全 |
| 适合 | 团队协作、高质量交付 | 个人高效开发 | 日常编码 |
| 规则系统 | Steering（三种模式） | CLAUDE.md + Skills | Rules（globs） |
| 开发流程 | 需求→规格→实现→验证 | 需求→实现→验证 | 需求→实现 |
| AWS 集成 | ★★★ | ★☆☆ | ★☆☆ |

---

## 配置模板

| 模板 | 用途 |
|------|------|
| [steering-always.md](templates/steering-always.md) | Steering 全局规则模板（Java + Spring Boot），复制到 `.kiro/steering/` |

---

## 延伸阅读

- [Kiro 官方文档](https://kiro.dev/docs)
- [superpowers-zh](https://github.com/jnMetaCode/superpowers-zh) — Skills 方法论（也支持 Kiro）
