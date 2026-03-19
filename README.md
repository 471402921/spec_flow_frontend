# SpecFlow Frontend

SpecFlow 框架的前端部分，支持 **Flutter** 和 **React** 双选型。

## 快速开始

1. 查看 `CLAUDE.md` 确认当前选型
2. 阅读 `doc/README.md` 了解开发 SOP
3. 使用 Skills 驱动开发流程：
   - `/sf-frontend-gen` — 生成业务模块
   - `/sf-frontend-review` — 代码审核
   - `/sf-frontend-test` — 生成测试

## 项目结构

```
├── CLAUDE.md                  # 框架契约（含当前选型声明）
├── doc/
│   ├── README.md              # 前端开发 SOP
│   ├── deploy.md              # 部署指南
│   ├── requirements/          # PRD 和需求文档
│   └── design/                # Tech Pack
├── .claude/skills/
│   ├── sf-frontend-gen/       # 模块生成 Skill
│   ├── sf-frontend-review/    # 代码审核 Skill
│   └── sf-frontend-test/      # 测试生成 Skill
└── src/                       # 源码（由 Skill 生成）
    ├── pages/                 # 页面 / 路由入口
    ├── features/              # 业务模块（按领域划分，Skill 生成时填充）
    ├── services/
    │   └── http/              # HTTP client、拦截器、Token 注入
    ├── shared/                # 跨模块共享组件、工具函数
    └── models/                # 全局数据类型
```

## 关联项目

- 后端：SpecFlow Backend（Java DDD Light 架构）
