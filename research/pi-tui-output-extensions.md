# Pi TUI 输出精简扩展调研

> 调研快照：2026-08-26。当前本机 Pi：`0.84.3`。

## 结论

严格同时满足“独立扩展、非废弃、受欢迎、默认只突出用户输入和最终答复”的项目，目前没有完全匹配者。

- **最值得先试的独立扩展：** [`MasuRii/pi-tool-display`](https://github.com/MasuRii/pi-tool-display)。它最成熟、最受欢迎，主要压缩工具结果；已在本机 Pi `0.84.3` 通过启动、命令注册和多行工具输出 smoke test 验证。已发布 `0.5.0` 的 peer 依赖只声明到 Pi `0.80.x`，且标准 `opencode` 预设会与 `pi-fff` 的 `read/grep` ownership 冲突，需要使用自定义 ownership 配置。
- **行为最完全匹配、且仓库很受欢迎：** Firstmate 的 [`/calm`](https://github.com/kunchenguid/firstmate/blob/main/docs/calm.md)。它不是独立 npm 扩展，而是 Firstmate 仓库内的项目级 Pi 扩展；不建议为了 UI 单独引入整个 Firstmate。
- **最接近独立“一键 Zen”：** [`aleclarson/pi-zen`](https://github.com/aleclarson/pi-zen)，但仅 2★、未发布 npm，属于小众实验性候选。

Pi 核心本身已经支持 [`hideThinkingBlock`](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/settings.md)、`Ctrl+O` 折叠工具输出和 `Ctrl+T` 折叠思考；`/tree` 还有 `user-only` 过滤，但这些不能把主 transcript 默认变成“只显示用户输入和最终答复”。

## 候选对比

| 候选 | 热度/维护证据 | 能做什么 | 判断 |
| --- | --- | --- | --- |
| [`pi-tool-display`](https://github.com/MasuRii/pi-tool-display) | 263★、41 forks；npm 最近一个月约 3,107 下载；未归档；MIT；发布 `0.5.0` | `read/grep/find/ls/MCP` 可设 `hidden/summary/preview`；bash 可折叠或只显示行数；edit/write 提供紧凑 diff；支持 `/tool-display`、预设和工具所有权开关。执行逻辑仍委托原工具，主要改 renderer。 | **已在本机 Pi 0.84.3 验证。** 标准 `opencode` 预设会把所有 ownership 设为 `true`，与 `pi-fff` 的 `read/grep` 冲突；当前配置将这两个工具设为 `false`，其余工具保留 opencode renderer，`preset=custom` 是预期状态。思考只做标签/上下文清理，不负责隐藏。 |
| Firstmate [`/calm`](https://github.com/kunchenguid/firstmate/blob/main/docs/calm.md) | Firstmate 仓库 4,138★、1,373 forks；持续更新；未归档 | 隐藏 Pi 内置工具 shell/result、折叠 thinking、隐藏 mid-turn assistant working note；用户输入和最终 agent response 保留；只改展示层，模型上下文、session、export 保留。 | **最贴合目标，但不是独立包。** 只覆盖它分类的内置/运营行；任意自定义工具、展开 reasoning、部分状态行仍可见。依赖 Firstmate 的目录和项目级 `.pi/extensions`。 |
| [`pi-zen`](https://github.com/aleclarson/pi-zen) | 2★；最近提交 2026-08-15；未归档；peer 使用 `@earendil-works/pi-coding-agent`；未发布 npm | `/zen` 时隐藏 `ToolExecutionComponent` 的工具调用历史和 transcript thinking；执行期间在 working 行显示最新 thinking preview，结束后清除；关闭后恢复。仅改 renderer。 | **功能很接近但不满足“受欢迎”。** 还会显示最新思考预览，不是完全无 thinking；通过原型 patch，需关注 Pi 内部变更。 |
| [`pi-context-prune`](https://github.com/championswimmer/pi-context-prune) | 220★、19 forks；npm 最近一个月约 1,189 下载；npm `1.3.0`；未归档 | 在 `context` 事件中把已完成工具批次总结后从未来 LLM context 中裁剪；原始输出保留在 session，可用 `context_tree_query` 恢复；有 `agent-message` 等触发模式。 | **不是 TUI 扩展。** 适合“不要把旧工具输出继续喂给模型”，不会隐藏当前屏幕内容；可与 UI 扩展组合。 |
| [`pi-caveman`](https://github.com/jonjonrankin/pi-caveman) | 94★、18 forks；npm 最近一个月约 7,212 下载；npm `1.0.8`；未归档 | 通过 `before_agent_start` 注入简洁回答规则，压缩最终文本输出 token；有 lite/full/ultra 等级。 | **不是 TUI 扩展。** 不影响 thinking、输入 token、工具调用或工具结果；只能作为最终答复变短的可选层。 |
| [`pi-thinking-steps`](https://github.com/crustyhacker/pi-thinking-steps) | 133★、15 forks；npm 最近一个月约 252 下载；未归档 | thinking 的 `collapsed/summary/expanded` 三种展示，支持 `Alt+T` 和持久化。 | **不符合“隐藏思考”，且兼容性风险高。** 当前包仍依赖已迁移/弃用的 `@mariozechner/pi-* 0.69.0`，不是本机 `@earendil-works` 0.84.3 的优先选择。 |

## 建议组合

### 稳妥的成熟路线

1. 在 `~/.pi/agent/settings.json` 保持：

   ```json
   { "hideThinkingBlock": true }
   ```

2. 试用 `pi-tool-display` 的 `opencode` 预设：

   ```text
   /tool-display preset opencode
   ```

   结果是工具结果尽量隐藏/压缩，edit/write 仍保留有用的 diff 摘要；最终答复保持正常 Markdown。当前仓库的自定义配置关闭了 `read/grep` ownership 以兼容 `pi-fff`；标准预设会覆盖这两个开关，导致 Pi `0.84.3` 加载冲突。

### 追求“绝对安静”

- Firstmate 用户直接使用 `/calm`，这是当前最接近“用户输入 + 最终答复”的高热度实现。
- 普通 Pi 用户可试 `pi-zen`，但它不满足受欢迎标准，且默认仍在 working 行显示最新 thinking preview；不建议作为生产配置的首选。

### 同时减少模型上下文

若“整合工具输出”也包含不希望反复发送给模型的旧结果，再叠加 `pi-context-prune`；注意它会调用总结模型，带来额外延迟/成本，且不改变屏幕显示。

## 不纳入正式推荐

- [`mbufkin/pi-hide-tools`](https://github.com/mbufkin/pi-hide-tools)：0★，功能直接但影响范围和维护/采用证据不足。
- [`yuritoledo/pi-compact-output`](https://github.com/yuritoledo/pi-compact-output)：0★，默认一行工具输出的方向正确，但项目很新且缺少受欢迎度证据。
- [`zackerydev/pi-minimalist-ui`](https://github.com/zackerydev/pi-minimalist-ui)：2★，功能与目标接近，但热度低；它也通过工具定义覆盖和原型 patch 改 UI。
- [`@zigai/pi-response-renderer`](https://github.com/zigai/pi-tweaks/tree/master/packages/pi-response-renderer)：只压缩 assistant Markdown 的 fence、空行和 italic，不处理工具输出/思考隐藏。
