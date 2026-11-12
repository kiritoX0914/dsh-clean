# DSH 纯净版（DSH Clean Edition）

[English](README.md) | 中文

**基于 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) `0.2.1-rc.2` 的非官方社区构建：无遥测、无官方登录、无自动更新。**

DSH 纯净版是对 DeepSeek Harness 做了仔细补丁、并从源码编译的发行。它仍然是本地 agent 运行时——命令行、Web UI、以及可便携的 Windows 桌面端——但官方账号、自动更新、遥测、分析和静默上报路径已被切断。你只需自备 API Key，或把模型地址指到任意 OpenAI 兼容（或 Anthropic Messages 兼容）服务。默认启动时，它不会连接 DeepSeek。

> **非官方社区构建。** 本项目**与 DeepSeek 没有任何隶属、授权或支持关系**。"DeepSeek" 与 "DeepSeek Harness" 是其权利人的商标。上游项目采用 MIT 许可证；本仓库在保留原版权声明的前提下分发带有明确标记的修改源码，以及独立构建的二进制。使用风险自负。

上游标签：[`dsh-v0.2.1-rc.2`](https://github.com/deepseek-ai/deepseek-harness/releases/tag/dsh-v0.2.1-rc.2)（提交 `687f282e`）。官方 README 保存在 [docs/UPSTREAM_README.md](docs/UPSTREAM_README.md) / [docs/UPSTREAM_README.zh.md](docs/UPSTREAM_README.zh.md)。逐文件改动清单见 [docs/OFFLINE_CHANGES.md](docs/OFFLINE_CHANGES.md)。相对官方标签的完整 diff 作为 [`dsh-offline-0.2.1-rc.2.patch`](https://github.com/kiritoX0914/dsh-clean/releases/tag/v0.2.1-rc.2-clean) 随 Release 发布。

所有改动处都带有 `OFFLINE BUILD` 注释。原实现在可行的地方保留为 `*Unused` 函数，方便审计，而不是把代码改得无从对照。

## 切断了什么

即使残留配置试图重新打开，以下路径也无法再访问官方设施：

| 类别 | 具体切断内容 |
| --- | --- |
| **遥测 / 分析** | OTLP 日志与事件导出、会话遥测（原收集器 `dsh-otel-collector.deepseeksvc.com`）、桌面产品遥测、前端产品分析，以及夹带在官方 API 请求中的会话日志、插件清单字段。这些插件已在 bundle 补丁中禁用，启动器还会设置 `DSH_TELEMETRY_DISABLED=1`。 |
| **身份请求头** | 不再把匿名用户 ID 持久化到磁盘（改为进程内临时随机 UUID）。不再发送 `x-deepseek-harness-user-id`、session-id 以及 compact 头。产品归因标识为 `dsh-offline` / `local`。默认 User-Agent 为 `dsh-offline/...`。 |
| **账号登录** | 官方账号 / 推理源改为永不解析的 `.invalid` 域名。平台认证在任何 `fetch` 之前就会抛错。登录界面、「登录 / 联系我们」菜单、欢迎页认证按钮、未登录时的账号区均已去掉。引导文案改为：添加 API Key，或在「设置 → 模型」中配置 OpenAI 兼容服务。 |
| **自动更新** | 桌面更新协调器**从不调用** `electron-updater`。不嵌入强制更新策略。打包时 `publish: null`，发行包中**没有** `app-update.yml`。 |
| **插件市场启动探测** | 插件管理器启动时不再 ping npm / npmmirror。 |

## 什么情况下仍会访问网络（仅在你主动使用时）

纯净版不是网络沙箱。默认启动不会连接 DeepSeek，但当你**主动使用**相应功能时，运行时仍会上网：

- **模型 API。** 若你设置了 `DEEPSEEK_API_KEY` 并使用 DeepSeek 提供方，默认地址仍是 `https://api.deepseek.com/anthropic`（以及 `/v1/tools`）。可用 `DEEPSEEK_BASE_URL` 指向任意 Anthropic Messages 兼容服务，或在 **设置 → 模型** 中添加 OpenAI 兼容提供方。这是「用你自己的 Key 调模型」，不是遥测。
- **联网搜索 / 抓取提供方**，包括默认的 `deepseek-official`（使用你的 `DEEPSEEK_API_KEY`），以及 Exa、Perplexity 等可选提供方——仅在你启用时。
- **其他内置模型提供方**（OpenAI、Anthropic、Google 等）——仅在你选择它们时。
- **思维链翻译**（Google / Bing）和 **语音识别模型下载**（Hugging Face / hf-mirror）——仅在你手动打开这些功能时。
- **插件安装**，来自 npm（`registry.npmjs.org` / `registry.npmmirror.com`）或 GitHub——仅在你亲自安装插件时。

还有少量官方 URL 只作为文档字符串或打包脚本常量出现（`api-docs.deepseek.com`、`www.deepseek.com`、`download.deepseek.com`）。本免签构建的运行时不会去请求它们。

在构建机上，用 Node 网络审计钩子运行 `dsh web`：**没有任何外部连接**（只有 `dns.lookup 127.0.0.1`）。`dsh headless` 指向本地 mock 模型服务时，唯一连接是 `127.0.0.1:18080`，请求头中无 user-id / session-id，User-Agent 为 `dsh-offline/0.2.1-rc.2 (+local)`。

## 下载

预编译包见标签 [`v0.2.1-rc.2-clean`](https://github.com/kiritoX0914/dsh-clean/releases/tag/v0.2.1-rc.2-clean)：

| 文件 | 大小 | SHA-256 |
| --- | ---: | --- |
| [dsh-offline-0.2.1-rc.2-linux-x64-cli.tar.gz](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2-linux-x64-cli.tar.gz) | 164.2 MiB | `398d21ed097ad99b1ec5be8e1df45d50ae7d9004b55587d6d5239dd198e40349` |
| [dsh-offline-0.2.1-rc.2-win-x64-cli.zip](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2-win-x64-cli.zip) | 188.5 MiB | `e1df673b36e56442eabccf1858f32e6dbd65a64b43c42af5f6db0e4e3293f71a` |
| [dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip) | 390.9 MiB | `5e6ae3b463331542d64271d18b3deb4efc44e7ba78bf1c836913709a196f8696` |
| [dsh-offline-0.2.1-rc.2.patch](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2.patch) | 52.4 KiB | `c2e0c765ca7cfbf9fae7ea9a2d3b648df3251ce4a42680078db9844971b5f69a` |

同一 Release 还包含 `SHA256SUMS`、Ed25519 分离签名 `SHA256SUMS.sig`、公钥 [`dsh-offline-signing.pub.pem`](dsh-offline-signing.pub.pem)、[docs/VERIFY.md](docs/VERIFY.md) 和 [docs/OFFLINE_CHANGES.md](docs/OFFLINE_CHANGES.md)。

## 快速开始

先设置模型凭据（也可以启动后再在界面里添加提供方）：

```sh
export DEEPSEEK_API_KEY=sk-...
# 可选：任意 Anthropic Messages 兼容地址
# export DEEPSEEK_BASE_URL=https://your-server.example/anthropic
```

也可以先不设环境变量，启动后在 **设置 → 模型** 中填写 OpenAI 兼容服务的 Base URL 和 API Key。可用 `DSH_HOME` 覆盖数据目录。只是打开界面的话，没有任何必填项。

### Linux 命令行

```sh
tar -xzf dsh-offline-0.2.1-rc.2-linux-x64-cli.tar.gz
cd dsh-offline-0.2.1-rc.2-linux-x64
./dsh --version          # 0.2.1-rc.2
./dsh web                # Web UI，默认 http://127.0.0.1:3080
./dsh tui
./dsh headless "你的任务"
```

压缩包自带 Node.js 24 运行时，启动器是 `./dsh`。

### Windows 命令行

解压 `dsh-offline-0.2.1-rc.2-win-x64-cli.zip`，在 `dsh-offline-0.2.1-rc.2-win-x64\` 目录下运行 `dsh.cmd`（或 `dsh.ps1`）：

```bat
dsh.cmd --version
dsh.cmd web
```

### Windows 桌面（便携版）

这是免安装的解压目录，不是 NSIS 安装包（安装器界面无法在 Linux 交叉构建中生成）。

1. 把 `dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip` 解压到**较短的路径**，例如 `C:\DSH`。树内最长路径大约 210 个字符，解压位置过深可能碰到 `MAX_PATH`。
2. 运行 `DSH Offline.exe`。
3. Windows SmartScreen 会提示未知发布者——本构建**没有** Authenticode 签名。请先核对照下面的校验和，确认后再选 **更多信息 → 仍要运行**。
4. 添加 API Key，或打开 **设置 → 模型** 配置 OpenAI 兼容接口。

应用 ID 为 `local.dsh.offline`。窗口标题 / 产品名为 **DSH Offline**。

## 校验发布文件

需要 OpenSSL 3.x。Windows 上 Git for Windows 一般自带 `openssl`。

```sh
# 1）校验 SHA256SUMS 的 Ed25519 签名
openssl pkeyutl -verify -pubin -inkey dsh-offline-signing.pub.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
# 应输出：Signature Verified Successfully

# 2）校验每个文件的哈希
sha256sum -c SHA256SUMS
```

公钥指纹（DER 编码的 SHA-256）：

```
29932eaf6e032fc9be5cd8b7c0529762120618ee34f924850c061f378d63d326
```

同一把公钥也在仓库里：[`dsh-offline-signing.pub.pem`](dsh-offline-signing.pub.pem)。该签名只证明文件与本次构建一致，**不是** Windows Authenticode 代码签名，也**不代表**任何官方背书。

在 Windows PowerShell 里如果只想核对单个文件：

```powershell
Get-FileHash .\dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip -Algorithm SHA256
# 与 SHA256SUMS 中的对应行比较
```

## 从源码构建

环境要求：

- Node.js 24（发布包使用 **Node 24.22.1** 构建；`package.json` 的 engines 允许 `^22.19.0 || >=24.0.0`）
- **pnpm 11.28.5**（见 `packageManager` 字段；可用 Corepack 固定版本）

```sh
git clone https://github.com/kiritoX0914/dsh-clean.git
cd dsh-clean
git checkout v0.2.1-rc.2-clean
pnpm install
pnpm build          # 发布构建跑过完整 gate，全部通过
pnpm dsh web        # 直接从源码运行 Web UI
pnpm dsh --version
```

### Linux / Windows 命令行包装

已发布的 CLI 压缩包，是对 `@deepseek-ai/dsh` 做 injected + hoisted 的 `pnpm deploy`，再打入官方 Node 24.22.1 运行时：

```sh
pnpm --filter @deepseek-ai/dsh deploy \
  --config.inject-workspace-packages=true \
  --config.node-linker=hoisted \
  --config.ignore-scripts=true \
  ./dist/cli-linux   # Windows 则换成对应的暂存目录

# injected deploy 会漏掉 23 个工作区 @deepseek-ai/* peer 依赖；
# 发布构建用 pnpm pack 把这些工作区包补进 node_modules。
```

若需要 `landlock-run` 而又没有 `musl-gcc`，发布构建是在 Linux 主机上用 `gcc -static -Os` 从 `native/system/packages/entry`（`packages/entry/src/main.c`）编译的。

### 在 Linux 上交叉打包 Windows 桌面

```sh
export DSH_OFFLINE_CROSS_WIN=1
export DSH_DESKTOP_APP_ID=local.dsh.offline
pnpm --dir apps/desktop run package:win:x64:unsigned --dir
```

`DSH_OFFLINE_CROSS_WIN=1` 是本补丁提供的开关，允许在 Linux x64 主机上打出免签的 win-x64 目录版：跳过 Visual Studio / PowerShell 工具检查、无法在 Linux 上执行的 `electron.exe` 冒烟测试、`rcedit`，以及「official 构建记录」校验。产物是便携解压目录（发布时打成 `dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip`），不是 NSIS 安装器。

## 已知限制

- **Windows 二进制是在 Linux 上交叉编译的，尚未在真实 Windows 上运行过。** Windows CLI 的 JS 依赖图在 Linux 上可解析到原生模块加载一步（因为只带了 win32 原生件而停止，符合预期）。桌面压缩包已检查：`app.asar` 中只有 `dshDesktopAppId: local.dsh.offline`，没有 `dshMandatoryUpdatePolicy`；`resources/` 下没有 `app-update.yml`；`lib/main.js` 里更新器被硬性关闭；被改过的运行时文件与 Linux CLI 逐字节一致；原生附加组件为 win32-x64。
- **没有 Authenticode 签名。** SmartScreen 会提示未知发布者。桌面 exe 的图标和版本信息仍是 Electron 默认值——Linux 交叉构建没有 Wine / `rcedit` 步骤。
- **桌面压缩包里带了一些多余的 linux-x64 原生组件**（交叉安装时为了让安装脚本能在 Linux 构建机上运行）。Windows 上不会加载它们，只是多占体积。
- **自定义接口只对本地 mock 服务做过验证**，构建机没有拿这套构建去连生产环境的 OpenAI 或 DeepSeek 账号。
- 这是上游开发者预览版 `0.2.1-rc.2` 的**预发布快照**。上游已经在继续迭代，本仓库冻结在该标签。
- 在把任何 agent 运行时放到你自己的机器上、并授予工具权限之前，请先阅读上游的 [SAFETY.md](SAFETY.md)。

## 许可证

[MIT](LICENSE)。Copyright (c) 2026 DeepSeek。Modifications Copyright (c) 2026 kiritoX0914。

原版权声明与许可全文均予保留。第三方许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
