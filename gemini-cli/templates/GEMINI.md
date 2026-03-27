# 项目背景
这是一个 Go 微服务项目。

# 目录结构
- cmd/         — 各服务入口
- internal/    — 内部包（不对外暴露）
- pkg/         — 公共包（可被外部引用）
- api/         — Protobuf / OpenAPI 定义
- deployments/ — Docker / K8s 配置
- scripts/     — 构建和部署脚本

# 常用命令
- 编译所有服务：make build-all
- 跑测试：make test
- 生成 proto：make proto-gen
- 本地启动：docker compose up
- Lint：golangci-lint run

# 代码规范
- 服务间通信用 gRPC，不要用 HTTP
- 配置统一走 Viper，不要硬编码
- 错误处理用 fmt.Errorf("xxx: %w", err) 包装
- 日志用 slog，不要用 fmt.Println
