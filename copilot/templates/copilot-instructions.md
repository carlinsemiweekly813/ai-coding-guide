# 项目指引

## 技术栈
- Python 3.12 + FastAPI + SQLAlchemy 2.0
- 数据库：PostgreSQL
- 缓存：Redis
- 测试：pytest + httpx

## 代码规范
- 类型注解必须完整，使用 Python 3.10+ 语法（X | Y 而非 Union[X, Y]）
- async 函数用于所有 IO 操作
- 所有接口必须有 Pydantic 模型做入参校验
- 错误用自定义异常类，不要用裸 Exception

## 目录约定
- src/api/      — FastAPI 路由
- src/models/   — SQLAlchemy 模型
- src/schemas/  — Pydantic 模型
- src/services/ — 业务逻辑
- tests/        — 测试（镜像 src 结构）

## 禁止
- 不要用 sync 方式操作数据库
- 不要在路由函数里写业务逻辑
- 不要硬编码配置，用环境变量
