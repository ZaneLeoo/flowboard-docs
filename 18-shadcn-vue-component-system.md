# 18｜shadcn-vue 组件体系与视觉改造记录

## 1. 决策

FlowBoard 当前所有已实现页面统一采用 shadcn-vue 的“组件源码归项目所有”模式，而非引入一套不可控的预制主题库。

选择原因：FlowBoard 是 C 端个人效率工具，需要可靠的可访问交互，但不能继承后台产品常见的密集、强边框、黑色主操作视觉。shadcn-vue 提供组件生成和源码维护模式；Reka UI（Radix Vue 的 v2 演进）提供无样式、可组合的底层交互能力。

## 2. 技术落地

- 新增 `components.json`，明确 `@/components/ui`、`@/lib/utils` 等 shadcn-vue 路径约定。
- `radix-vue` 替换为 `reka-ui`；新增 `class-variance-authority`、`clsx`、`tailwind-merge`、`tw-animate-css`、`lucide-vue-next` 与 `vue-sonner`。
- 组件源码位于 `flowboard-web/src/components/ui/`：`Button`、`Input`、`Textarea`、`Label`、`Checkbox`、`Dialog`、`Select`、`Sonner` 与确认对话框组合件。
- 登录、注册、Today、项目编辑、任务编辑、任务详情和删除确认均已接入该组件层。
- 原手写 `AppButton`、手写 Teleport 弹窗、原生 `select`/`checkbox`、浏览器 `confirm` 和页面内临时 Toast 均已移除。

## 3. FlowBoard 视觉约束

- 组件结构来自 shadcn-vue，视觉令牌仍使用 FlowBoard：蓝色主操作 `#2463EB`、暖灰画布、克制白色材料和圆角。
- `Button` 提供即时 `scale(0.97)` 按压反馈；弹窗、Select 以 `transform + opacity` 短过渡出现，并响应减少动效偏好。
- `Dialog` 与 `Select` 由 Reka 管理焦点、Esc 关闭、键盘导航和 Portal；`Sonner` 负责全局成功/失败反馈。
- 不使用 shadcn-vue 默认黑色主色，也不引入 Element Plus、Ant Design Vue、PrimeVue 等预制主题。

## 4. 验证

| 检查 | 结果 |
| --- | --- |
| `npm run build` | 通过；Vue 类型检查和 Vite 生产构建均成功。 |
| 前端依赖 | `radix-vue` 已移除；新组件依赖安装完成。 |
| 交互替换 | 项目/任务 Dialog、Select、Checkbox、删除确认和 Toast 已使用新组件层。 |

## 5. 后续原则

后续新增界面不得再手写新的基础按钮、输入框、弹窗、选择器或通知组件；优先从 `src/components/ui/` 复用。如需新交互，先通过 shadcn-vue/ Reka 的原语生成或组合，再应用 FlowBoard 的令牌与视觉约束。
