---
globs: ["**/*"]
---

# 全局规则

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
- 不要在组件里写业务逻辑，抽到 hooks 或 services
