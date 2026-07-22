# 03｜数据模型与 API 契约

## 1. 数据模型（MVP）

```mermaid
erDiagram
  USERS ||--|| WORKSPACES : owns
  WORKSPACES ||--o{ PROJECTS : contains
  PROJECTS ||--o{ TASKS : contains
  TASKS ||--o{ TASK_TAGS : has
  TAGS ||--o{ TASK_TAGS : labels

  USERS {
    uuid id PK
    string email UK
    string password_hash
    string display_name
    datetime created_at
  }
  WORKSPACES {
    uuid id PK
    uuid owner_id FK
    string name
  }
  PROJECTS {
    uuid id PK
    uuid workspace_id FK
    string name
    string color
    boolean archived
    datetime created_at
  }
  TASKS {
    uuid id PK
    uuid project_id FK
    string title
    text description
    string status
    string priority
    date due_date
    boolean in_today
    int sort_order
    datetime completed_at
    datetime deleted_at
  }
  TAGS {
    uuid id PK
    uuid workspace_id FK
    string name
    string color
  }
  TASK_TAGS {
    uuid task_id FK
    uuid tag_id FK
  }
```

所有主键使用 UUID。时间以 UTC 保存、由前端按用户时区显示；日期型截止日按用户本地日期解释。

## 2. REST 约定

- 基础路径：`/api/v1`。
- JSON 字段使用 `camelCase`；日期为 ISO 8601。
- 写操作必须带 `Authorization: Bearer <access-token>`。
- 成功响应直接返回资源；列表返回 `items`、`page`、`pageSize`、`total`。
- 错误统一返回 `code`、`message`、`traceId` 与可选 `fieldErrors`。

```json
{
  "code": "VALIDATION_ERROR",
  "message": "请求参数不合法",
  "traceId": "01J...",
  "fieldErrors": [{ "field": "title", "message": "任务标题不能为空" }]
}
```

## 3. 第一批接口

| 方法 | 路径 | 需求 | 说明 |
| --- | --- | --- | --- |
| `POST` | `/auth/register` | FR-AUTH-001 | 创建账号与默认工作空间。 |
| `POST` | `/auth/login` | FR-AUTH-002 | 返回 access / refresh token。 |
| `POST` | `/auth/refresh` | FR-AUTH-002 | 刷新 access token。 |
| `GET` | `/me` | FR-AUTH-002 | 获取当前用户与工作空间摘要。 |
| `GET, POST` | `/projects` | FR-PROJECT-001 | 列表与创建项目。 |
| `GET, PATCH, DELETE` | `/projects/{projectId}` | FR-PROJECT-001 | 查询、编辑、归档项目。 |
| `GET, POST` | `/projects/{projectId}/tasks` | FR-TASK-001 | 列表与创建任务。 |
| `GET, PATCH, DELETE` | `/tasks/{taskId}` | FR-TASK-001 | 查询、编辑、软删除任务。 |
| `POST` | `/tasks/{taskId}/complete` | FR-TASK-001 | 完成任务。 |
| `POST` | `/tasks/{taskId}/reopen` | FR-TASK-001 | 重新打开任务。 |
| `PATCH` | `/tasks/reorder` | FR-TASK-002 | 调整状态及排序。 |
| `GET` | `/today` | FR-TODAY-001 | 今日、逾期和手动置顶任务。 |
| `GET` | `/insights/overview` | FR-INSIGHT-001 | 今日与项目统计。 |

## 4. 跨仓库变更规则

1. 后端先发布向后兼容的字段或接口，再发布使用它的前端。
2. 删除字段至少经历“停止写入 → 前端不再读取 → 一个发布周期后删除”三步。
3. OpenAPI 定义由后端生成并发布；前端在 CI 中校验关键请求/响应模型。
4. 破坏性变化必须先更新本文件、创建迁移计划，并在 PR 中关联需求编号。
