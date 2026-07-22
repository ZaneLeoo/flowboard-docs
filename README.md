# FlowBoard 文档中心

本仓库保存 FlowBoard 的跨端、跨环境文档，是需求、架构与交付决策的唯一来源。

## 文档索引

| 文档 | 用途 | 当前状态 |
| --- | --- | --- |
| [00-project-charter.md](00-project-charter.md) | 项目目标、边界与关键决策 | 已建立 |
| [01-prd.md](01-prd.md) | MVP 需求与用户流程 | 已建立 |
| [02-architecture.md](02-architecture.md) | 系统、仓库与部署架构 | 已建立 |
| [03-data-api.md](03-data-api.md) | 数据模型与 REST API 契约 | 已建立 |
| [04-ui-ux.md](04-ui-ux.md) | C 端视觉、交互与可访问性规范 | 已建立 |
| [05-delivery-devops.md](05-delivery-devops.md) | Git、Docker、Jenkins 与部署方案 | 已建立 |
| [06-quality-acceptance.md](06-quality-acceptance.md) | 测试策略、质量门禁与验收清单 | 已建立 |
| [07-backend-detailed-design.md](07-backend-detailed-design.md) | 后端分层、鉴权、Mapper 与迁移细节 | 已建立 |
| [08-frontend-detailed-design.md](08-frontend-detailed-design.md) | 前端路由、状态、组件和页面实现细节 | 已建立 |
| [09-visual-direction.md](09-visual-direction.md) | AI 视觉方向稿与界面决策 | 已建立 |
| [10-m1-project-bootstrap.md](10-m1-project-bootstrap.md) | M1 工程初始化记录与验收 | 已完成 |

## 维护规则

1. 实现前先更新相关设计文档；实现完成后补充验收结果和变更原因。
2. 用需求编号（如 `FR-TASK-001`）连接需求、测试和提交信息。
3. 文档只记录可验证的决定；尚未决定的内容放入“待确认”章节。
4. 密码、令牌、服务器 IP 和真实生产配置不得提交到本仓库。

## 仓库关系

```text
project/
├── flowboard-docs/  # 本仓库：共享文档与验收记录
├── flowboard-web/   # Vue C 端应用
└── flowboard-api/   # Spring Boot API
```

`flowboard-docs` 是文档仓库，不承载业务代码；前端和后端仍为两个独立 Git 仓库。
