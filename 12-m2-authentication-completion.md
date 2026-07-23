# 12｜M2：认证与 MySQL 基础验收记录

## 1. 结论

M2 已完成。FlowBoard API 已拥有基于 MySQL 的账号、个人工作区和会话基础；H2 已从工程与测试中移除。

对应实现提交：`flowboard-api` 的 `078ec5c 功能：实现认证与 MySQL 集成测试`。

## 2. 交付内容

### 数据与迁移

- Flyway `V001` 创建 `users` 和 `workspaces`；注册在一个事务中创建账号与“我的工作区”。
- Flyway `V002` 创建 `refresh_tokens`；数据库仅存储 refresh token 的 SHA-256 哈希。
- 所有测试数据库均为 MySQL 8.4 临时容器，不使用 H2。

### 认证接口

| 接口 | 结果 |
| --- | --- |
| `POST /api/v1/auth/register` | 创建账号、工作区与会话，返回 `201` |
| `POST /api/v1/auth/login` | 校验邮箱密码，返回新的会话 |
| `POST /api/v1/auth/refresh` | 轮换 refresh token 并返回新的 access token |
| `POST /api/v1/auth/logout` | 撤销当前 refresh token 并清除 Cookie |
| `GET /api/v1/me` | 返回当前登录用户和个人工作区 |

access token 为短期 JWT；refresh token 只通过 `HttpOnly` Cookie 传递，不进入 JSON 响应或前端状态。

### 基础安全与可观测性

- 使用 BCrypt 保存密码哈希。
- 认证失败、参数校验与业务冲突统一返回 `code`、`message`、`traceId` 和可选字段错误。
- 每次请求返回 `X-Trace-Id`，便于以后在服务器与 Jenkins 日志中定位问题。
- CORS 允许来源、JWT 密钥、数据库账号均通过运行环境配置，不写入 Git。

## 3. 验收结果

在 Windows Docker Desktop 环境中完成真实 MySQL 8.4 集成测试：

| 验证项 | 结果 |
| --- | --- |
| 空 MySQL 执行 V001、V002 | 通过 |
| 注册创建工作区 | 通过 |
| 登录正确/错误密码处理 | 通过 |
| Bearer access token 访问 `/me` | 通过 |
| refresh token 轮换、旧 token 失效 | 通过 |
| 重复注册与匿名访问错误契约 | 通过 |
| Maven `verify`（编译、测试、打包） | 通过，3 个测试、0 失败 |

没有 Docker 的机器会跳过这组容器集成测试，不会悄悄降级为 H2；后续 Jenkins 必须配置 Docker 并实际执行这组测试。

## 4. 下一阶段

进入 M3：在 `flowboard-web` 实现中文优先的登录、注册与主工作区界面，并接入 M2 API。完成界面联调后，再实现项目和任务的第一批业务能力。
