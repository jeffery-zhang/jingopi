# Pi 配置

这套配置用于在多台电脑上复现当前 Pi 工作环境。配置仓库地址：

```text
https://github.com/jeffery-zhang/jingopi
```

## 版本

- Pi：`>=0.84.0`（本机当前参考版本：`0.85.1`）
- Node.js/npm：需要支持当前 Pi 版本
- 配置作用域：`~/.pi/agent/settings.json`

Pi 本身不由 `settings.json` 锁定。新电脑应使用 `0.84.0` 或更高版本：

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
pi --version
```

## 已安装的 Pi 包

| 包 | 本机当前版本（参考） | 作用 | 主要配置入口 |
| --- | --- | --- | --- |
| `pi-fff` | `0.1.12` | 模糊文件查找、索引内容搜索、增强 `read`/`grep`，并提供 `find_files`、`fff_grep` 等工具 | `/fff-features`、`/fff-status` |
| `pi-web-access` | `0.24.2` | 网页搜索、网页内容提取、GitHub 仓库读取、PDF、YouTube 和视频分析 | `~/.pi/web-search.json` |
| `pi-subagents` | `0.57.0` | 子 agent 委托、并行/后台任务、workflow 脚本和角色化模型路由 | `agent/settings.json` 的 `subagents` 块 |
| `pi-mcp-adapter` | `2.27.0` | 通过单个代理工具按需发现和调用 MCP 工具，支持 MCP OAuth 和 `mcpScript` | `~/.pi/agent/mcp.json`、`.mcp.json` |
| `pi-chrome` | `0.15.46` | 通过 Chrome 扩展操作现有 Chrome profile 中的标签页和网页 | `/chrome onboard`、`/chrome doctor` |
| `pi-rtk-optimizer` | `0.9.0` | 自动将 `bash` 重写为 `rtk` 等效命令并压缩工具输出（`bash`/`read`/`grep`），降低上下文占用 | `/rtk`、`/rtk stats` |
| `@jingoz/pi-questionnaire` | `0.1.0` | TUI 交互式问卷工具，支持单问题和多问题标签界面，用于收集用户偏好与决策 | 无（通过 `questionnaire` 工具参数配置） |
| `@jingoz/pi-image-input` | `0.2.0` | 将 Pi TUI 原生剪贴板图片路径转换为 `[Image]` 标记和图片附件；要求 Pi `>=0.84.3` 且当前模型支持图片 | 无（重启 Pi 或执行 `/reload`） |
| `pi-tool-display` | `0.5.0` | OpenCode 风格精简工具调用、隐藏/摘要工具输出、紧凑 diff 和用户消息框 | `/tool-display`、`~/.pi/agent/extensions/pi-tool-display/config.json` |
| `@victor-software-house/pi-curated-themes` | `0.2.1` | 65+ 款精选暗色终端主题大合集（适配自 iTerm2 配色，含 Catppuccin、Gruvbox、Kanagawa、Dracula+ 等） | `/settings`（选择主题） |
| `pi-hermes-memory` | `0.9.7` | 本地持久记忆、项目记忆、历史会话全文检索、自动复盘、纠错记录和 procedural skills | `/memory-insights`、`/memory-skills` |
| `pi-antigravity` | `0.7.0` | 使用 Google OAuth 直连 Antigravity / Cloud Code Assist 内部 API，提供 Gemini、Claude、GPT-OSS 模型和配额诊断 | `/login antigravity`、`/antigravity.models`、`/antigravity.doctor` |


`agent/settings.json` 只记录包名，不锁定扩展版本。新电脑启动时会安装当前可用版本；上表版本只是本机当前参考值。更新已安装的包：

```bash
pi update --extensions
```

也可以只更新一个包：

```bash
pi update npm:pi-chrome
```

## 全局设置

主要配置文件是 `agent/settings.json`，安装或启动 Pi 后对应于：

```text
~/.pi/agent/settings.json
```

当前重要设置包括：

- 默认 provider：`muryo`
- 默认模型：`deepseek-v4.1-flash`
- 默认思考等级：`high`
- 可循环模型：精选 `muryo` 系列模型（含 `muse-spark-1.3-contributor`）、`muryo/gemini-3.8-flash-high`、`muryo/deepseek-v4.1-flash`、`muryo/stealth/union-alpha` 与 `antigravity/gemini-3.8-flash`
- 主题：`catppuccin-mocha`
- 隐藏 thinking block
- HTTP 空闲超时：`300000` 毫秒

全局指令位于：

```text
~/.pi/agent/AGENTS.md
```

修改配置后，在 Pi 中执行：

```text
/reload
```

## pi-fff 配置

默认安装后即可使用。可在 Pi 中执行：

```text
/fff-features
```

启用或禁用 `read`、`grep` 的 FFF 增强功能。其他命令：

```text
/fff-status
/reindex-fff
```

功能开关状态保存在：

```text
~/.pi/agent/extensions/pi-fff.json
```

该文件是本机运行状态，不纳入配置仓库。新电脑使用默认设置即可，需要时重新执行 `/fff-features`。

## pi-web-access 配置

配置文件：

```text
~/.pi/web-search.json
```

当前机器配置了 Exa -> Tavily 的搜索回退顺序，并启用了 transient、quota、network 和 invalid-response 回退。建议使用环境变量，不要把真实 key 写入 Git：

```json
{
  "exaApiKey": "$EXA_API_KEY",
  "tavilyApiKey": "$TAVILY_API_KEY",
  "searchRouting": {
    "providers": ["exa", "tavily"],
    "fallbackOn": ["transient", "quota", "network", "invalid-response"]
  }
}
```

相关命令：

```text
/websearch
/curator
/google-account
```

`JINA_API_KEY` 等其他 provider key 可以通过环境变量按需配置。没有额外 key 时，仍可使用插件提供的零配置搜索能力。

## pi-mcp-adapter 配置

全局 MCP 配置位于：

```text
~/.pi/agent/mcp.json
```

当前配置包含 Cloudflare OAuth MCP server 和通过本机 `codegraph serve --mcp` 启动的 CodeGraph MCP server。项目级配置可以使用：

```text
.mcp.json
.pi/mcp.json
```

常用命令：

```text
/mcp
/mcp setup
/mcp-auth cloudflare
/mcp logout cloudflare
```

OAuth 凭据保存在操作系统凭据库，不会随 Git 仓库同步。新电脑需要重新执行 `/mcp-auth cloudflare`。CodeGraph server 要求本机已安装可执行的 `codegraph` 命令。

## Herdr 状态扩展

`agent/extensions/herdr-agent-state.ts` 由 Herdr 管理，用于向 Herdr pane 报告当前 Pi 会话及 `working`、`blocked`、`idle` 状态。仅在 Herdr 注入 `HERDR_ENV=1`、`HERDR_SOCKET_PATH` 和 `HERDR_PANE_ID` 时启用；普通 Pi 会话不会建立连接。Herdr 重新安装或升级集成时可能覆盖该文件。

## pi-hermes-memory 配置

当前不创建 `hermes-memory-config.json`，直接使用 `pi-hermes-memory` 的官方内置默认配置：

- `policy-only` 模式，注入完整 memory policy，需要时通过工具检索记忆。
- 每 10 轮或 15 次工具调用执行后台复盘。
- 启用 correction detection、压缩前 flush 和退出时 flush。
- 记忆达到上限时自动 consolidation。
- 启用 `skill_manage`，允许主 agent 在正常工作中创建和维护 procedural skills。
- 启用 standing instructions 和 SQLite FTS5 session search。

数据目录：

```text
~/.pi/agent/pi-hermes-memory/
~/.pi/agent/projects-memory/
```

常用命令：

```text
/memory-insights
/memory-skills
/memory-index-sessions
/memory-sync-markdown
/memory-preview-context
/memory-pin
```

扩展会在启动时有界回填历史 session；需要完整重建索引时执行 `/memory-index-sessions`。升级包或调整配置后执行 `/reload`。

## pi-chrome 配置

`pi-chrome` 需要 Pi 包和 Chrome unpacked extension 两部分同时存在。

安装 Pi 包后，在 Pi 中执行：

```text
/chrome onboard
```

在 Chrome 的 `chrome://extensions` 中开启开发者模式，选择“加载已解压的扩展程序”，加载目录：

```text
~/.pi/agent/npm/node_modules/pi-chrome/extensions/chrome-profile-bridge/browser-extension
```

然后执行：

```text
/reload
/chrome doctor
/chrome authorize
```

`/chrome authorize` 默认授权当前 Pi 会话 15 分钟，也可以指定时长，例如：

```text
/chrome authorize 30m
/chrome authorize indefinite
```

Chrome 的 Cookie、登录状态、已打开标签页和 profile 不属于 Pi 配置，需要在每台电脑上使用或准备对应的 Chrome profile。

## pi-rtk-optimizer 配置

默认安装后即可生效。主要功能包括：

- **命令重写**：将开发工作流中的 `bash` 命令自动委托重写为 `rtk` 命令（需安装 `rtk` 二进制，若未安装会自动回退为原生命令）。
- **输出压缩**：管道化压缩工具输出（ANSI 剥离、测试/构建/Git/Linter 输出聚合、搜索结果分组、智能与硬截断等），大幅降低 Context Token 消耗。

可在 Pi 中执行：

```text
/rtk
```

打开交互式 TUI 设置面板。其他常用命令：

```text
/rtk show
/rtk verify
/rtk stats
/rtk reset
```

配置文件路径：

```text
~/.pi/agent/extensions/pi-rtk-optimizer/config.json
```

该配置会随仓库同步，用于在新电脑复现当前扩展设置。

## pi-tool-display 配置

当前配置文件位于：

```text
~/.pi/agent/extensions/pi-tool-display/config.json
```

本机采用精简展示策略：

- `pi-fff` 继续拥有 `read`/`grep`，避免覆盖 FFF 的路径解析和搜索增强。
- `find`、`ls`、`bash`、`edit`、`write` 使用 `pi-tool-display` 的 renderer。
- 搜索结果和 MCP 结果隐藏，bash 使用 `opencode` 折叠展示；编辑和写入保留紧凑 diff，可用 `Ctrl+O` 展开。
- `hideThinkingBlock` 已在全局设置中启用，思考块不显示在主 transcript 中。

交互式设置和状态查看：

```text
/tool-display
/tool-display show
```

配置变更后执行：

```text
/reload
```

当前 Pi 为 `0.85.1`，而 `pi-tool-display@0.5.0` 的 peer 依赖声明最高到 `0.80.x`。本配置已在本机通过 Pi 启动、RPC 命令注册和多行 `bash`/`find` smoke test 验证。直接执行 `/tool-display preset opencode` 会把所有 ownership 重置为 `true`，与 `pi-fff` 的 `read`/`grep` 冲突；因此当前配置保留这两个工具为 `false`，`/tool-display show` 显示 `preset=custom` 是预期结果。

### 为什么 `read`/`grep` 必须保持 `false`

`pi-tool-display` 的 `registerToolOverrides.read`/`.grep` 默认为 `true`，会把这两个工具重新注册一遍。而 `pi-fff` 在 `builtInReadEnhancement`/`builtInGrepEnhancement` 开启时也注册 `read`/`grep`，两个扩展同时注册同名工具会被 Pi 判定为扩展加载错误（`Tool "read" conflicts with ...`）。任何 error 级诊断都会让启动以退出码 1 失败（`pi -p`、`--mode json`、`--mode rpc` 全部不可用），并打印 `Hint: Start without extensions using "pi -ne".`。因此 `config.json` 里这两个开关保持 `false`，由 `pi-fff` 拥有它们。

改动扩展后请验证（应只出现模型告警，退出码 0）：

```bash
pi -p --no-session "" < /dev/null; echo $?
```

### 与工具集相关的两点背景

- `agent/settings.json` 的 `defaultTools: ["read","bash","edit","write","grep","find","ls"]` 是显式声明默认激活的内置工具集。`defaultTools` 只控制内置工具，扩展/自定义工具不受影响（严格白名单是命令行 `--tools`）。
- `read` 不在系统提示的 `Available tools` 中：`pi-fff` 覆写 `read` 时没有提供 `promptSnippet`，而 Pi 不会把无 snippet 的工具列进该区域（也不会从内置定义继承）。这是 `pi-fff` 自身缺口；需要时可给 `pi-fff` 的 `read` 注册补上 `promptSnippet`/`promptGuidelines`。

### 曾评估但不采用的 `pi-compact-ui`

`pi-compact-ui` 会把 thinking 与连续工具调用合并成树状分组，观感更紧凑，但它的原版会无条件重新注册 `read`/`bash`/`edit`/`write`/`find`/`grep`/`ls` 七个内置工具，与 `pi-fff` 的 `read`/`grep` 直接冲突，导致 Pi 启动失败。若要使用，必须先让它跳过 `read`/`grep` 的注册（需 fork 上游 `index.ts`，因为该包没有对应配置项）；其占位定义也缺少 `promptSnippet`，会让工具从系统提示的 `Available tools` 中消失。当前选择继续使用 `pi-tool-display`。

## pi-questionnaire 配置

该工具仅在 TUI 模式下可用，非交互模式会返回 UI 不可用的结果，不会阻塞运行。

## Muryo Gemini 配置

通过 Muryo OpenAI Responses 兼容接口访问 Gemini 3.8 Flash High。模型配置位于 `agent/models.json`，凭据使用环境变量 `MURYO_API_KEY`。思考强度映射（`thinkingLevelMap`）仅开放 `low` 与 `high` 两个挡位。

## muryo 自定义模型

`agent/models.json` 的 `muryo` provider 下自定义了若干上游未在 Pi 内置目录中列出的模型。接口统一为 `openai-responses`，`baseUrl` 为 `https://cpa.muryorouter.cc/v1`。

| 模型 id | 上下文 | 最大输出 | thinking 映射 | 上游路由 |
| --- | --- | --- | --- | --- |
| `stealth/union-alpha` | 262144 (256K) | 64000 | `low`/`high`/`max` | 已连通（200 OK） |
| `muse-spark-1.3-contributor(-free)` | 1000000 | 64000 | 仅 `xhigh` | — |
| `gemini-3.8-flash-high` | 1000000 | 32000 | `low`/`high` | — |
| `deepseek-v4.1-flash` | 1000000 | 32000 | `low`/`high`/`max` | — |

### stealth/union-alpha

原 `union-alpha`（于 2026-09-17 更新模型名指向 `stealth/union-alpha`）。`text` + `image` 输入（多模态），262144 (256K) 上下文，`maxTokens: 64000`，`thinkingLevelMap` 开放 `low`/`high`/`max`（`off`/`minimal`/`medium` → `low`，`xhigh` → `high`，与同映射的 `deepseek-v4.1-flash` 一致）。

#### 连通状态

- 更新为 `stealth/union-alpha` 后重试连接，`/v1/responses` 现已成功连通（200 OK）。

## 已移除的 muryo 模型

### GPT 系列（2026-09-18 移除）

muryo 上游已不再提供 GPT 系列（原 `gpt-5.6-sol` / `gpt-5.6-terra` / `gpt-5.6-luna` / `gpt-5.5`，均为 `codex` 路由，353000 上下文）。已从 `agent/models.json` 与 `agent/settings.json` 的 `enabledModels` 中删除，`subagents` 块中引用它们的默认模型与回退模型一并改用现存模型（见下文 pi-subagents 配置）。

注：`nvidia` 的 `openai/gpt-oss-20b`、`antigravity` 的 `gpt-oss-120b` 属于其他 provider 的开放权重模型，与 muryo 无关，保留在 `agent/models-store.json` 缓存中。

### Atria-Dawn-Preview（2026-09-17 移除）

曾于 2026-09-16 接入（`text` 输入，262144 上下文，`thinkingLevelMap` 开放 `low`/`medium`/`high`）。移除原因：响应速度太慢，实际使用不划算。已从 `agent/models.json` 与 `agent/settings.json` 的 `enabledModels` 中删除。

保留两条当时验证过的上游事实，供后续遇到同族 codex 模型时参考：

- **不接受图片输入**：`/v1/responses` 与 `/v1/chat/completions` 传图片均返回 `400 {"type":"atria_api_error","code":"upstream_error"}`（data URI 与公开 https URL 都被拒，纯文本对照 200）。所以 `input` 必须为 `["text"]`；声明 `image` 会让图片直发、整个请求被 400 拒掉，连文本也拿不到。声明为纯文本后 `read` 一张图会降级为 `(tool image omitted: model does not support images)`。
- **thinking 三挡真实生效**：回环 echo server 抓到 `off`/`minimal`/`low` → `low`，`medium` → `medium`，`high`/`xhigh`/`max` → `high`；reasoning 正文长度随挡位变化。上游把推理放在 `output[].type="reasoning"` 的 `content[].type="reasoning_text"`，pi 能正确还原为独立 `thinking` 块。

## pi-antigravity 配置

通过 Google OAuth 授权直接访问 Antigravity / Cloud Code Assist 内部 API，提供 Gemini（含 3.8 Flash）、Claude 及 GPT-OSS 等模型。

首次使用或凭据失效时在 Pi 中执行：

```text
/login antigravity
```

完成浏览器 Google 账号授权（凭据自动保存于 `agent/auth.json`）。常用命令：

```text
/antigravity.models
/antigravity.usage
/antigravity.doctor
/antigravity.image
```


## pi-subagents 配置

`pi-subagents` 为内置角色配置了模型分层与回退策略，配置位于 `agent/settings.json` 的 `subagents` 块。

| 角色 | 模型 | thinking | 回退 |
| --- | --- | --- | --- |
| `scout` | `muryo/gemini-3.8-flash-high` | `high` | `muryo/deepseek-v4.1-flash:high` |
| `researcher` | `muryo/deepseek-v4.1-flash` | `high` | `muryo/gemini-3.8-flash-high:high` |
| `worker` | `muryo/gemini-3.8-flash-high` | `high` | `muryo/deepseek-v4.1-flash:max` |
| `reviewer` | `muryo/deepseek-v4.1-flash` | `max` | `muryo/stealth/union-alpha:max` |
| `oracle` | `muryo/stealth/union-alpha` | `max` | `muryo/deepseek-v4.1-flash:max` |
| `delegate` | `muryo/gemini-3.8-flash-high` | `high` | `muryo/deepseek-v4.1-flash:high` |

全局默认子 agent 模型为 `muryo/deepseek-v4.1-flash`，默认 thinking 为 `high`，`maxThinking` 为 `max`。

2026-09-18 起 muryo 不再提供 GPT 系列模型，原先承担强推理层的 `gpt-5.6-sol`/`terra`/`luna` 由 `deepseek-v4.1-flash`（1M 上下文、`low`/`high`/`max`）与 `stealth/union-alpha`（256K、`low`/`high`/`max`）替代；快速层继续使用 `gemini-3.8-flash-high`。thinking 只写模型实际支持的挡位（`gemini-3.8-flash-high` 无 `max`、`muse-spark-1.3-contributor` 仅 `xhigh`）。

常用命令：

```text
/subagents-models
/subagents-models reviewer
/subagents-doctor
/subagents-guide
```

## 自定义模型和凭据

当前默认 provider `muryo` 的详细模型配置位于：

```text
~/.pi/agent/models.json
```

该文件被 `.gitignore` 排除。新电脑需要安全复制或重新创建，并设置：

```text
MURYO_API_KEY
```

Pi 登录凭据位于 `~/.pi/agent/auth.json` 或操作系统凭据库，应该在新电脑重新登录，不要提交到仓库。

## 新电脑安装流程

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
git clone https://github.com/jeffery-zhang/jingopi.git ~/.pi
pi
```

Pi 启动时会根据 `agent/settings.json` 自动安装缺失的 Pi 包，扩展版本不锁定。启动后验证：

```bash
pi list
```

随后补齐 `models.json`、`web-search.json` 和环境变量，重新授权 MCP，并按上面的步骤加载 Chrome 扩展。

## 安全说明

以下内容包含凭据、缓存或本机状态，已被 `.gitignore` 排除，不应提交：

- `agent/auth.json`
- `agent/models.json`
- `agent/models-store.json`
- `web-search.json`
- `agent/mcp-cache.json`
- `agent/npm/`
- `agent/sessions/`
