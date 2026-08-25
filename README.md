# Pi 配置

这套配置用于在多台电脑上复现当前 Pi 工作环境。配置仓库地址：

```text
https://github.com/jeffery-zhang/jingopi
```

## 版本

- Pi：`0.84.3`
- Node.js/npm：需要支持当前 Pi 版本
- 配置作用域：`~/.pi/agent/settings.json`

Pi 本身不由 `settings.json` 锁定。安装指定版本：

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent@0.84.3
```

## 已安装的 Pi 包

| 包 | 版本 | 作用 | 主要配置入口 |
| --- | --- | --- | --- |
| `pi-fff` | `0.1.12` | 模糊文件查找、索引内容搜索、增强 `read`/`grep`，并提供 `find_files`、`fff_grep` 等工具 | `/fff-features`、`/fff-status` |
| `pi-web-access` | `0.24.2` | 网页搜索、网页内容提取、GitHub 仓库读取、PDF、YouTube 和视频分析 | `~/.pi/web-search.json` |
| `pi-mcp-adapter` | `2.27.0` | 通过单个代理工具按需发现和调用 MCP 工具，支持 MCP OAuth 和 `mcpScript` | `~/.pi/agent/mcp.json`、`.mcp.json` |
| `pi-chrome` | `0.15.46` | 通过 Chrome 扩展操作现有 Chrome profile 中的标签页和网页 | `/chrome onboard`、`/chrome doctor` |

包版本在 `agent/settings.json` 的 `packages` 数组中精确锁定。精确版本不会被 `pi update --extensions` 自动升级。升级某个包时，显式指定新版本：

```bash
pi install npm:pi-chrome@0.15.47
```

## 全局设置

主要配置文件是 `agent/settings.json`，安装或启动 Pi 后对应于：

```text
~/.pi/agent/settings.json
```

当前重要设置包括：

- 默认 provider：`muryo`
- 默认模型：`deepseek-v4-flash-vision-exp`
- 默认思考等级：`max`
- 可循环模型：`muryo/*`
- 深色主题
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

当前配置包含 Cloudflare OAuth MCP server。项目级配置可以使用：

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

OAuth 凭据保存在操作系统凭据库，不会随 Git 仓库同步。新电脑需要重新执行 `/mcp-auth cloudflare`。

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
npm install -g --ignore-scripts @earendil-works/pi-coding-agent@0.84.3
git clone https://github.com/jeffery-zhang/jingopi.git ~/.pi
pi
```

Pi 启动时会根据 `agent/settings.json` 自动安装缺失的、已锁定版本的 Pi 包。启动后验证：

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
