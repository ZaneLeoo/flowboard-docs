# 07｜后端详细设计

## 1. 目标与边界

后端负责身份验证、用户数据隔离、任务业务规则、MySQL 持久化和可观测性。它使用 Java 17，采用模块化单体：模块可独立测试和演进，但以一个 Spring Boot 应用、一个镜像和一个数据库交付。

本设计不引入通用 CRUD 代码生成器、分布式事务或消息队列。接口只服务 FlowBoard Web；未来移动端仍复用同一 REST API。

## 2. 工程结构

```text
src/main/java/com/flowboard/api/
├── FlowBoardApplication.java
├── config/                 # Security、Jackson、MyBatis-Plus、OpenAPI 配置
├── shared/
│   ├── api/                # ApiException、错误码、分页响应
│   ├── security/           # JWT 过滤器、当前用户上下文
│   ├── persistence/        # BaseEntity、审计字段、逻辑删除约定
│   └── web/                # 全局异常处理、traceId 过滤器
├── auth/
│   ├── controller/ dto/ application/ persistence/
│   └── domain/
├── workspace/
├── project/
├── task/
└── insight/
```

每个业务模块遵循以下依赖方向：

```text
Controller → Application Service → Mapper / Domain Rule
                 ↓
             DTO / Command
```

- Controller 只负责 HTTP 输入输出、参数校验和返回状态码。
- Application Service 承担鉴权、事务边界和跨实体业务规则。
- Mapper 只执行数据访问；不包含当前用户、HTTP 或业务状态判断。
- Entity 仅对应表结构，绝不直接作为 API 请求或响应对象。

## 3. 模块职责

| 模块 | Application Service | 关键规则 |
| --- | --- | --- |
| `auth` | 注册、登录、刷新、退出 | 邮箱唯一；密码只接受明文输入并即时哈希；刷新令牌可撤销。 |
| `workspace` | 初始化、当前工作空间读取 | 注册即创建个人工作空间；MVP 一个用户只返回自己的工作空间。 |
| `project` | 项目 CRUD、归档 | 归档后禁止创建或编辑任务；访问必须验证工作空间归属。 |
| `task` | 任务 CRUD、完成、重开、排序、今日操作 | 写操作验证项目归属；完成时间与状态同步；删除使用逻辑删除。 |
| `insight` | 今日和项目统计 | 聚合查询只读，不回写任务数据。 |

## 4. 鉴权与数据隔离

1. 注册时使用 BCrypt 哈希密码；数据库从不保存明文或可逆密码。
2. 登录后返回短期 access token 与长期 refresh token；refresh token 仅保存其哈希及过期时间。
3. `JwtAuthenticationFilter` 解析 access token，并将 `userId` 写入安全上下文。
4. 每个资源读取/写入均以 `userId` + `workspaceId` 作为查询条件或在 Service 层验证所有权。
5. 无令牌返回 `401`；访问非本人资源统一返回 `404`，避免泄露资源存在性。

令牌有效期和密钥只来自环境变量。测试环境使用独立、可公开的测试密钥；生产密钥只能存于 Jenkins Credentials 与服务器环境文件。

## 5. MyBatis-Plus 约定

### 5.1 实体与 Mapper

- 实体位于各模块的 `persistence/entity`，使用明确的表名、主键和逻辑删除映射。
- 常规按主键读写使用 `BaseMapper<T>`；Service 不向 Controller 暴露 Entity。
- `TaskMapper`、`ProjectMapper` 等复杂查询接口配套 XML，置于 `src/main/resources/mapper/<module>/`。
- XML 仅保存 SQL；动态条件由明确参数对象表达，禁止拼接用户输入。

### 5.2 插件与数据规范

- 配置 `MybatisPlusInterceptor` 与 MySQL 分页拦截器。
- `deleted_at IS NULL` 由逻辑删除策略统一保证；管理员绕过逻辑删除必须显式命名。
- 所有列表查询显式指定排序和最大 `pageSize`（初期最大 100）。
- 使用 `version` 做任务排序和编辑的乐观锁保护；冲突时返回 `409 TASK_VERSION_CONFLICT`，前端提示刷新。
- 普通 CRUD 使用 MyBatis-Plus；统计、关联和批量排序使用 XML SQL，所有 SQL 避免 `SELECT *`。

## 6. 数据库迁移

Flyway 是唯一的数据库结构变更入口，迁移文件使用：

```text
V001__create_users_and_workspaces.sql
V002__create_projects.sql
V003__create_tasks_and_tags.sql
V004__add_refresh_tokens.sql
```

- 已进入任何共享环境的迁移不可修改；修复使用新的迁移文件。
- 每张业务表都有 `created_at`、`updated_at`；任务额外有 `deleted_at`、`completed_at` 和 `version`。
- 每个迁移在本地空库、升级库和 CI 集成测试中执行。

## 7. HTTP 设计细节

### 7.1 请求校验与错误码

| 场景 | HTTP | 错误码 |
| --- | --- | --- |
| 参数缺失或格式错误 | `400` | `VALIDATION_ERROR` |
| 未登录或令牌过期 | `401` | `UNAUTHENTICATED` |
| 邮箱已注册 | `409` | `EMAIL_ALREADY_EXISTS` |
| 资源不存在或不属于当前用户 | `404` | `RESOURCE_NOT_FOUND` |
| 项目已归档仍尝试改任务 | `409` | `PROJECT_ARCHIVED` |
| 乐观锁冲突 | `409` | `TASK_VERSION_CONFLICT` |

`@RestControllerAdvice` 统一转换异常；生产环境不返回堆栈。每次请求分配或透传 `X-Trace-Id`，同时写入响应和日志。

### 7.2 事务

- 注册（用户 + 工作空间）、项目归档、任务状态/排序修改在 Application Service 内使用事务。
- 统计和列表查询使用只读事务或不启事务。
- 外部 I/O 不放入数据库事务；MVP 暂无外部同步。

## 8. 测试与可观测性

- Service 单元测试覆盖所有权校验、归档限制、今日计算和完成/重开状态。
- Mapper 集成测试使用临时 MySQL，验证 Flyway 迁移和关键 XML SQL。
- Controller 测试覆盖认证、错误码和请求校验。
- 暴露 `/actuator/health`；生产只开放需要的 Actuator 端点，健康检查不泄露数据库凭据或版本以外的信息。

## 9. 后端完成定义

完成后端基础工程的标准：空数据库可通过 Flyway 启动；注册、登录、项目和任务核心接口可由 OpenAPI 测试；未授权访问被拒绝；`mvn verify` 在本机和 Jenkins 均可通过。
