# 10｜M1 工程初始化记录

## 1. 目标

建立可独立演进的前端与后端仓库，落地已确认的运行时、基础配置、中文国际化和项目级 Git 规范。本阶段不实现注册、登录或任务等业务功能。

## 2. 仓库与远程地址

| 仓库 | 本地目录 | GitHub 远程 |
| --- | --- | --- |
| 文档 | `D:\Dylan\project\flowboard-docs` | `ZaneLeoo/flowboard-docs` |
| API | `D:\Dylan\project\flowboard-api` | `ZaneLeoo/flowboard-api` |
| Web | `D:\Dylan\project\flowboard-web` | `ZaneLeoo/flowboard-web` |

所有仓库均使用项目级 Git 身份 `Dylan <zaneleoooooo@gmail.com>`，不依赖或修改全局提交人配置。

## 3. API 基线

- Java 17、Spring Boot 4.1.0、MyBatis-Plus 3.5.17。
- Spring Security 当前仅放行 `/actuator/health`；M2 再替换为 JWT 鉴权。
- MySQL/Flyway 的生产配置由环境变量提供，测试使用 H2 内存数据库。
- `.mvn/settings.xml` 固定 Maven Central，隔离本机旧的 Maven 镜像设置。
- `.editorconfig`、`.gitattributes`、`.env.example` 和中文 README 已建立。

## 4. Web 基线

- Vue 3、TypeScript、Vite、Tailwind CSS。
- 已接入 Vue Router、Pinia 和 `vue-i18n`；默认语言为 `zh-CN`。
- 已建立颜色/层级设计令牌、基础可访问焦点样式和中文启动页。
- `Radix Vue` 作为后续无样式交互原语依赖，M3 使用。
- `.editorconfig`、`.gitattributes`、`.env.example` 和中文 README 已建立。

## 5. 本地命令

```powershell
# API：当前终端需显式选择 JDK 17
$env:JAVA_HOME = 'D:\JDK17.0.12'
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
cd D:\Dylan\project\flowboard-api
.\mvnw.cmd verify

# Web
cd D:\Dylan\project\flowboard-web
npm install
npm run build
```

## 6. 验收清单

| 项目 | 标准 | 状态 |
| --- | --- | --- |
| API 脚手架 | Maven Wrapper、Spring Boot、MyBatis-Plus、H2 测试配置存在 | 已完成 |
| Web 脚手架 | Vue、Tailwind、路由、Pinia、中文国际化可编译 | 已完成 |
| Git | 两个仓库本地初始化、项目级提交身份和 GitHub `origin` 已配置 | 已完成 |
| 前端构建 | `npm run build` 通过 | 已通过 |
| 后端构建 | `mvn verify` 通过 | 已通过 |
| Maven Wrapper | `mvnw.cmd --version` 使用 Java 17 成功启动 | 已通过 |
| 首笔提交与推送 | 两个业务仓库均有中文首笔提交并跟踪 `origin/main` | 已完成 |

## 7. M1 完成后的下一步

进入 M2：以 `FR-AUTH-001` 和 `FR-AUTH-002` 为起点，创建 Flyway 初始表、注册/登录 API、令牌策略和测试。
