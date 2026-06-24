# XuCodex

XuCodex 是给原生 Termux 用的 Codex CLI 安装与第三方 API 配置小工具。这个文件夹最终只需要三个文件：

- `xu`：交互式配置应用。在 Termux 输入 `xu` 启动。
- `TERMUX_INSTALL_CODEX_CLI_COMMAND.txt`：在 Termux 里复制执行的一键安装命令。
- `README_XuCodex.md`：当前说明书。

当前版本已经按原生 Termux 设计：`xu` 会安装到 `$PREFIX/bin/xu`，所以安装后直接输入 `xu` 就能启动。

## 1. 在 Termux 里一键安装

先把整个 `XuCodex` 文件夹放到 Android 下载目录，然后在 Termux 里执行：

```sh
termux-setup-storage
cd ~/storage/downloads/XuCodex
pkg update -y && pkg upgrade -y && pkg install -y nodejs git ripgrep curl jq openssh nano && npm install -g @openai/codex undici && mkdir -p ~/.codex && cp ./xu "$PREFIX/bin/xu" && chmod +x "$PREFIX/bin/xu" && xu
```

这条命令会安装 Codex CLI 和代理支持包，把 `xu` 放进 Termux 默认 PATH 里的 `$PREFIX/bin/xu`，最后自动启动 `xu`。

## 2. 菜单按键

在菜单页：

- `Tab` / `↑↓`：切换选项
- `Enter`：确认
- `Esc`：返回上一级；第一层按 `Esc` 退出
- 模型列表页：`Space` 可以勾选多个模型，直接 `Enter` 会选择当前高亮模型
- 最后确认页：`Alt+A` 或 `A` 进入高级设置

普通文字输入页也支持 `Esc` 返回上一级。

## 3. xu 能配置什么

启动后按菜单选择：

```text
xu
```

- 单供应商：从同一份供应商列表里选一个，生成 `~/.codex/config.toml`；之后直接运行 `codex`。
- 多供应商：从同一份供应商列表生成统一入口，并写入默认 `~/.codex/config.toml`；之后直接运行 `codex`。
- Responses API：供应商真正支持 `/v1/responses` 时选这个。
- Chat Completions 转 Responses API：供应商只有 `/v1/chat/completions` 时选这个，`xu` 会自动安装本地转换代理。

单供应商和多供应商共用一个供应商列表。区别只是 Codex 模型列表里显示一个供应商，还是显示多个供应商；模型名都会用后缀区分供应商：

```text
gpt-5.5_ranmeng
gpt-5.5_supernb
kimi-k2.7-code_dext
glm-5.2_dext
```

你在 Codex 里选择 `模型名_供应商`，本地代理会自动把请求分发到对应供应商，并把上游真实模型名还原成不带后缀的模型名。供应商管理页里用 `Tab` 或 `↑↓` 选中供应商，按 `Enter` 进入编辑/删除。

`xu` 不会写入或修改 `~/.config/opencode/opencode.json`，所以不会改变同一个 Termux 环境里的 opencode 默认模型或供应商。

输入 Base URL 和 API key 后，`xu` 会自动尝试请求：

```text
<Base URL>/models
```

如果 Base URL 没带 `/v1`，它会自动补成 `/v1/models`。如果 Base URL 写成了 `/v1/chat/completions`，它会自动换算到 `/v1/models`。获取失败时会提示你手动输入模型 ID。

## 4. 高级设置

最后确认页不操作高级设置时，会使用默认值：

- 上下文长度：`128000`
- 输出长度：`32768`
- 请求超时：`60s`
- 自动重试：`10` 次

按 `Alt+A` 或 `A` 可以修改这些值。Chat Completions 转 Responses 模式下，本地代理会按这里的输出长度、超时和重试次数请求上游；遇到连接失败、超时、`429` 或 `5xx` 等临时错误会自动重试。

## 5. 本地代理端口

首次使用多供应商，或者首次配置 Chat Completions 转 Responses 时，`xu` 会要求设置本地代理端口，默认是 `57321`。它会检测 `127.0.0.1:<端口>` 是否被占用：

- 没占用：写入 `~/.codex/xu-proxy.sh`
- 已占用：提示换一个端口并重试

多供应商管理页里也可以重新设置端口。端口变化后，`xu` 会重新生成配置，保证 Codex 连接到正确端口。

## 6. 关于 VPN 和代理

新版 XuCodex 会让 Codex 只请求本机 `127.0.0.1`，真正访问第三方 API 的工作由本地转发层完成。Responses API 和 Chat Completions 转 Responses API 都走这个本地入口，所以不会再因为 Codex CLI 自己不吃代理变量而被迫依赖 VPN。

在 `xu` 里可以进入“上游网络/代理设置”：

- 跟随 Termux 环境变量：读取当前 `HTTPS_PROXY` / `ALL_PROXY`，没有就直连。
- 强制直连：清掉代理变量，直接请求供应商。
- 使用代理地址：保存一个代理地址给本地转发层，例如 `http://127.0.0.1:7890`。

如果你的供应商可以直连，就在 `xu` 里选择“强制直连”。如果你要走 Clash/代理软件，就选择“使用代理地址”。Android 系统 VPN 仍然可用，但不再是 XuCodex 运行的必要条件。

## 7. xu 会写哪些文件

`xu` 会自动写入这些本机配置：

- `~/.codex/config.toml`
- `~/.codex/xucodex.config.toml`（备用 profile）
- `~/.codex/xu-env.sh`
- `~/.codex/xu-proxy.sh`
- `~/.codex/xu-upstream-proxy.sh`
- `~/.codex/xu-chat-provider.sh`
- `~/.codex/xu-chat-providers.json`
- `$PREFIX/bin/codex-with-proxy`
- `$PREFIX/bin/codex-chat2responses-proxy.mjs`
- `$PREFIX/bin/generate-codex-model-catalog.mjs`
- `~/bin/codex`（XuCodex 管理的启动器，会备份旧文件）

`xu` 文件本身没有内置任何真实 API key。你输入的 key 只会保存在你手机本机配置里，不要把生成后的配置文件发给别人。

## 8. 常用命令

```sh
xu
codex
codex-with-proxy
codex doctor --summary
```

想换供应商或换协议时，重新输入 `xu` 跑一遍即可。
