# 08｜前端详细设计

## 1. 目标与边界

前端是面向个人用户的响应式 Web 应用。重点是“今天该做什么”的快速判断、低摩擦的任务操作和有层级但不过度装饰的界面。它不承担权限判断或业务真相，所有写操作以 API 结果为准。

## 2. 工程结构

```text
src/
├── app/                    # 根组件、路由、全局 Provider
├── assets/                 # 字体之外的本地静态资源
├── components/
│   ├── ui/                 # Button、Input、Dialog、Menu 等通用原语
│   ├── task/               # TaskCard、TaskList、TaskSheet
│   ├── project/            # ProjectCard、ProjectProgress
│   └── layout/             # AppShell、SideNav、MobileNav
├── features/
│   ├── auth/
│   ├── today/
│   ├── projects/
│   ├── tasks/
│   └── insights/
├── lib/                    # HTTP Client、日期、格式化、类型守卫
├── stores/                 # auth、ui 等跨页面客户端状态
├── styles/                 # Tailwind 入口、设计令牌、基础样式
└── views/                  # 路由页组合层
```

页面不直接请求 `fetch`。每个 feature 暴露类型化 API 函数、查询/变更 composable、视图组件和局部测试；共享组件不依赖任何业务 feature。

## 3. 路由与会话

| 路径 | 页面 | 访问控制 |
| --- | --- | --- |
| `/login` | 登录 | 未登录可访问；已登录跳转 `/today`。 |
| `/register` | 注册 | 未登录可访问；成功后进入引导。 |
| `/onboarding` | 首次项目/任务创建 | 需登录；完成后跳转 `/today`。 |
| `/today` | 今日任务与概览 | 需登录。 |
| `/projects` | 项目总览 | 需登录。 |
| `/projects/:projectId` | 项目详情 / 任务看板 | 需登录。 |
| `/settings` | 个人设置 | 需登录。 |

路由守卫只负责会话存在性和首屏恢复；后端仍是最终权限裁决者。access token 保存在内存，刷新会话依靠安全的 refresh token 策略；前端不得将长期令牌写入 `localStorage`。

## 4. 状态与数据获取

| 状态类型 | 存放位置 | 示例 |
| --- | --- | --- |
| 服务端状态 | feature composable + 可失效缓存 | 今日任务、项目列表、任务详情。 |
| 跨页面客户端状态 | Pinia | 当前会话摘要、导航折叠、主题、任务详情面板开关。 |
| 局部交互状态 | 组件 `ref` / `computed` | 编辑表单、Popover、拖拽中的临时位置。 |

HTTP Client 统一处理 `Authorization`、`X-Trace-Id`、`401` 刷新与失败重试。业务组件只处理成功、加载、空状态和错误状态，不能各自实现令牌刷新逻辑。

## 5. 国际化策略

- 使用 `vue-i18n`，默认 locale 为 `zh-CN`；MVP 只交付完整的简体中文资源。
- 所有系统固定文案使用语义键，例如 `task.create`、`task.complete`、`error.network`；组件模板不直接硬编码用户可见文案。
- 用户输入的项目名、任务名、备注等内容原样保存，不做机器翻译。
- 日期、数字和相对时间使用 `Intl` / `vue-i18n` 的 `zh-CN` 格式化能力。
- 英文资源 `en-US` 在全量翻译、术语校对和端到端测试完成后才开放语言切换；不展示半成品双语入口。

## 6. 页面与组件契约

### 6.1 Today

- 顶部：日期、今日聚焦数、创建任务按钮。
- 主列表：按焦点、计划、稍后、已完成分组；每项均可键盘完成、延期或打开详情。
- 下方：项目进度和轻量今日完成反馈；无任务时展示可执行的空状态。
- 桌面端任务详情在右侧面板打开；移动端使用底部抽屉。

### 6.2 Projects

- `ProjectCard` 显示名称、项目色、未完成数、进度和归档状态。
- 项目详情有列表与看板两种浏览方式；MVP 默认列表，拖拽排序作为渐进增强。
- 归档操作必须解释结果；成功后可短时间撤销。

### 6.3 TaskSheet

`TaskSheet` 只接受 `taskId`，自行通过 feature 层获取最新数据。它支持标题、描述、状态、优先级、截止日、标签和完成操作；保存采用乐观 UI，但 API 冲突时恢复并提示刷新。

## 7. 设计系统

### 7.1 令牌

使用 CSS Variables 定义语义 token，例如 `--surface-canvas`、`--surface-raised`、`--text-primary`、`--accent-primary`、`--border-subtle`、`--shadow-float`。Tailwind 只引用语义 token，不在业务组件直接使用任意色阶。

### 7.2 基础组件

| 组件 | 责任 |
| --- | --- |
| `AppButton` | 变体、加载、禁用、快捷键提示和按下反馈。 |
| `AppInput` | 标签、帮助文本、错误状态和可访问性关联。 |
| `AppDialog` / `AppPopover` | 基于 Radix Vue 的焦点管理、Esc 关闭和触发源定位。 |
| `AppToast` | 成功、警告、错误和撤销操作；不阻断主任务。 |
| `EmptyState` | 说明当前状态和唯一明确的下一步。 |

## 8. 响应式与动效

- 桌面端保留侧栏和右侧详情；小于 768px 时切换为底部导航和 `TaskSheet`。
- 点击反馈立即发生；hover 仅在 `hover: hover` 与精确指针下启用。
- 普通浮层只过渡 `transform` 和 `opacity`，时间短于 250ms；键盘高频动作不添加延迟动画。
- 手势抽屉与任务拖拽遵循 `apple-design`：Pointer Capture、1:1 跟随、阻尼边界、按释放速度决定归位或提交，并可随时反向。
- `prefers-reduced-motion` 下移除位移与弹簧，仅保留短淡入淡出；`prefers-reduced-transparency` 下取消玻璃模糊。

## 9. 前端测试

- Vitest：日期/今日规则、API 错误映射、Store 和关键组件状态。
- Vue Testing Library：键盘完成任务、打开/关闭详情、错误提示和表单校验。
- Playwright：注册、首次引导、创建任务、完成任务、项目归档的关键旅程。
- Chrome、Safari、移动端真实设备各做一次发布前冒烟检查。

## 10. 前端完成定义

完成前端基础工程的标准：所有路由可访问；认证守卫、空/加载/错误状态完善；按 PRD 的用户旅程在模拟 API 下可运行；桌面与手机断点下没有阻塞性布局问题。
