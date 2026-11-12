# DSH Offline 0.2.1-rc.2 — 离线改造说明

基于 DeepSeek Harness 标签 `dsh-v0.2.1-rc.2`（提交 `687f282e`，MIT 许可）的**非官方本地构建**。全程在构建机（Linux x64）上从源码编译。
目标：不检查/下载更新、不能登录官方账号、无任何遥测/分析/崩溃上报、不连官方服务；模型仍可用你自己的 API Key / 任意兼容服务。
MIT 许可与 `THIRD_PARTY_NOTICES.md` 原样保留在每个包里。完整改动见同目录 `dsh-offline-0.2.1-rc.2.patch`（`git diff 687f282e..HEAD`）。
所有改动处源码均带 `OFFLINE BUILD` 注释，改动是“外科式”的：原实现保留为 `*Unused` 函数便于审计，实际调用路径被替换为空操作。

## 一、切断项（逐文件）

### 遥测 / 分析
| 文件 | 改动 | 切断了什么 |
|---|---|---|
| `packages/telemetry/otel/src/transport.ts` | `createLogExporter` 改为返回新增的 `createNullLogExporter()`（导出即成功、无网络） | OTLP 日志上报 |
| `packages/telemetry/otel/src/event-transport.ts` | `createEventLogExporter` 同样返回空导出器 | OTLP 事件上报 |
| `packages/session/session-telemetry-otel/src/index.ts` | 模式强制 `DISABLED`，默认值也改为 `DISABLED` | 会话遥测（原上报 `https://dsh-otel-collector.deepseeksvc.com/v1/logs`） |
| `packages/host/product-telemetry-otel/src/index.ts` | 默认 endpoint 改为 `http://127.0.0.1:9/offline-disabled` | 桌面端产品遥测收集器（deepseeksvc） |
| `packages/client/product-analytics/src/index.ts` | 默认关闭，`enabled()` 恒为 false，`report` 空操作 | 前端产品分析埋点 |
| `packages/session/session-log-deepseek/src/index.ts` | `OFFLINE_BUILD` 常量，`apply` 直接返回（打包后编译为空函数） | 把会话日志 `dsh_session_log` 夹带进官方 API 请求 |
| `packages/llm/plugin-package-inventory-deepseek/src/index.ts` | 同上 | 把已装插件清单 `dsh_plugin_packages` 夹带进官方 API 请求 |
| `packages/identity/anonymous-user-id/src/index.ts` | 改为进程内临时随机 UUID，不落盘 | 持久化匿名用户 ID（可跨会话关联） |
| `packages/llm/llm-deepseek/src/adapter.ts` | 删除 `x-deepseek-harness-user-id` / session-id / compact 头 | 请求头里的用户/会话标识 |
| `packages/llm/llm/src/attribution.ts` | 应用标识改为 `dsh-offline` / `local` | 归因信息 |
| `packages/web/web-fetch-http/src/index.ts` | 默认 UA 改为 `dsh-offline/...` | UA 中的官方产品标识 |
| `packages/bundle/base/cordis.patch.yml` | `otel`、`session-telemetry-otel`、`llm-deepseek-account`、`session-log-deepseek`、`plugin-package-inventory-deepseek` 设为 `disabled: true`；收集器 URL 改为本地死地址 | 插件层面不挂载上述功能 |
| `packages/bundle/web-app/cordis.patch.yml` | `desktop-product-telemetry`、`product-analytics` 禁用，endpoint 改本地死地址 | 同上（Web/桌面层） |
| `packages/bundle/sdk-minimal/cordis.patch.yml` | session-log / plugin-inventory 禁用 | 同上（SDK 层） |

### 官方账号登录
| 文件 | 改动 |
|---|---|
| `packages/credentials/deepseek-account-platform/src/protocol.ts` | `platformRequest` 在任何 fetch 之前直接抛 `PlatformAuthError('no-response')` |
| `packages/credentials/deepseek-account-platform/src/index.ts` | 账号/推理 origin 默认改为 `https://account.offline.invalid` / `https://inference.offline.invalid`（`.invalid` 保留域名，永不解析） |
| `packages/llm/llm-deepseek-account/src/listing.ts` | 账号模型列表请求直接报 TRANSPORT 错误（且该插件已在 bundle 中禁用） |
| `packages/client/ui-settings-account/src/client/AccountMenu.tsx` | 移除“登录”“联系我们”菜单项和登录对话框 |
| `packages/client/ui-settings-account/src/client/SignInDialog.tsx` | 隐藏主“登录”按钮 |
| `packages/client/ui-settings-account/src/client/AccountSection.tsx` | 未登录时不渲染账号区 |
| `packages/client/ui-settings-account/src/client/locales.ts` | 中英文引导文案改为“添加 API Key 或在 设置 → 模型 中配置任意 OpenAI 兼容服务，本离线版本不提供账号登录” |
| `packages/client/ui-settings-account/src/contact-config.ts` | 飞书问卷链接替换为 `https://contact.offline.invalid/` |
| `apps/desktop/src/client/WelcomePage.tsx` | 桌面欢迎页隐藏登录/重试认证按钮，“开始”直接进入 API Key 页 |

### 更新
| 文件 | 改动 |
|---|---|
| `apps/desktop/src/update-coordinator.ts` | `OFFLINE_BUILD`：`enabled()` 为 false、检查恒为 idle、下载失败、安装抛错，**从不调用 electron-updater** |
| `apps/desktop/src/mandatory-update-policy.ts` | `resolveDesktopPolicyConfig` 恒返回 undefined（无远程强制更新策略） |
| `apps/desktop/scripts/electron-builder-config.mjs` | 不生成更新源（`publish: null`、无 `app-update.yml`）、不嵌入策略；产品名 `DSH Offline`，产物名 `dsh-offline-*`；appId 来自 `DSH_DESKTOP_APP_ID=local.dsh.offline` |
| `apps/desktop/scripts/package-target.ts` | 使用普通（非 official）前端构建配置 |

### 插件市场
| 文件 | 改动 |
|---|---|
| `packages/client/ui-plugin-manager/src/index.ts` | `registryProbeEnabled` 默认 false：启动时不再 ping npm / npmmirror |

### 仅构建用（不影响运行时行为）
- `apps/desktop/scripts/{package-target.ts,prepare-runtime.ts,prepare-dsh.ts,electron-builder-config.mjs,smoke-packaged-runtime.ts}`、`scripts/release/families.ts`：`DSH_OFFLINE_CROSS_WIN=1` 时允许在 Linux 上交叉打包 win-x64 免签目录版（跳过 VS/PowerShell 工具检查、无法在 Linux 执行的 electron.exe 冒烟测试、rcedit；跳过“official 构建记录”校验）。
- `native/system/packages/linux-x64/bin/landlock-run`：因缺 musl-gcc，用 `gcc -static -Os` 从 `packages/entry/src/main.c` 编译。

## 二、验证结果（在构建机上实测）
- Linux CLI：`./dsh --version` → `0.2.1-rc.2`。
- `dsh web` 在 Node 网络审计钩子（劫持 net/tls/dns，见构建机 `/workspace/dsh/audit/net-audit.cjs`）下运行并用 Chrome 打开 UI：**全部网络活动仅 `dns.lookup 127.0.0.1`**，浏览器侧零外部请求、零报错。
- `dsh headless` 指向本地 mock 模型服务（`DEEPSEEK_BASE_URL=http://127.0.0.1:18080/anthropic`）：返回正确，**唯一的连接是 127.0.0.1:18080**；请求头中无 user-id / session-id，UA 为 `dsh-offline/0.2.1-rc.2 (+local)`。
- Windows CLI / Desktop：在 Linux 上构建，**未在真实 Windows 上运行测试**。Windows CLI 的 JS 依赖图在 Linux 上可解析到原生模块加载一步（因只带 win32 原生件而停止，符合预期）。
- Windows Desktop 产物检查：`app.asar` 内 `package.json` 只有 `dshDesktopAppId: local.dsh.offline`，**无 `dshMandatoryUpdatePolicy`**；`resources/` 下**无 `app-update.yml`**（无更新源）；`lib/main.js` 中更新器 `enabled = () => false`、无任何 http(s) 地址；内置 dsh 运行时中上述被改文件与 Linux CLI 的逐字节一致；原生件为 win32-x64（koffi、node-pty ConPTY、sharp、sherpa-onnx 等），primary-runtime 为 win32 版 node.exe / python.exe。
- 桌面版附带了一些多余的 linux-x64 原生件（交叉安装时为了让安装脚本能在 Linux 上跑），在 Windows 上不会被加载，只占体积。

## 二点五、桌面版说明
- 交付的是**免安装便携版**（`win-unpacked` 目录打 zip），没有 NSIS 安装器：安装器 UI 需要 PowerShell + VS C++ 编译 `window-frame.dll`，Linux 上做不了。解压后直接运行 `DSH Offline.exe`。建议解压到较短路径（如 `C:\DSH`），最长内部路径约 210 字符。
- exe 的图标/版本信息仍是 Electron 默认（Linux 无 Wine，无法 rcedit），窗口标题/产品名为 DSH Offline。

## 三、仍然存在的官方地址（未切断的原因）
只有在**你主动配置/使用**时才会访问，默认启动不会连接：
- `https://api.deepseek.com/anthropic`（及 `/v1/tools`）：DeepSeek 模型提供方的默认地址。只有设置了 `DEEPSEEK_API_KEY` 并使用 deepseek 提供方时才会请求；可用 `DEEPSEEK_BASE_URL` 指向任意 Anthropic-Messages 兼容服务，或在 设置 → 模型 里改用其他 OpenAI 兼容提供方。这是“用你自己的 Key 调模型”，不是遥测。
- 联网搜索/抓取默认提供方 `deepseek-official`（同样使用你的 `DEEPSEEK_API_KEY`）；其余可选提供方：exa、perplexity 等。
- pi-ai 内置的各家模型默认地址（OpenAI、Anthropic、Google…），仅在你选择该提供方时使用。
- 思维链翻译（Google/Bing）、语音识别模型下载（HuggingFace / hf-mirror）：仅在你手动启用该功能时。
- 插件安装：从 npm（`registry.npmjs.org` / `registry.npmmirror.com`）或 GitHub 下载，仅在你手动安装插件时。
- `api-docs.deepseek.com`、`www.deepseek.com`（OpenTelemetry 语义常量里的示例字符串）：只是文档/常量文本，不会发起请求。
- 打包脚本中的 `download.deepseek.com` 等发布地址：仅官方签名发布流程使用，本免签离线构建不会走到，也未打进运行时。

## 四、使用所需环境变量
- 必需：无。启动器默认设置 `DSH_TELEMETRY_DISABLED=1`（双保险）。
- 模型：`DEEPSEEK_API_KEY=...`，可选 `DEEPSEEK_BASE_URL=http(s)://你的服务/anthropic`；或在 Web/桌面 UI 的 设置 → 模型 中配置 OpenAI 兼容服务（Base URL + Key）。
- 数据目录：`DSH_HOME`（可选）。

## 五、签名说明
- `SHA256SUMS` 用本机生成的 **Ed25519** OpenSSL 密钥做了分离签名 `SHA256SUMS.sig`，公钥 `dsh-offline-signing.pub.pem`（私钥不在发布目录中）。验证方法见 `VERIFY.md`。
- 这**不是** Windows Authenticode 代码签名。Windows 运行 exe 时 SmartScreen 会提示“未知发布者”，需点“更多信息 → 仍要运行”。桌面版 exe 的图标/版本资源保持 Electron 原样（Linux 上无 Wine，无法执行 rcedit）。

## 六、构建命令记录（构建机）
```sh
export COREPACK_HOME=/workspace/dsh/.corepack PATH=/workspace/dsh/toolchain/node-v24.22.1-linux-x64/bin:$PATH
cd /workspace/dsh/src && pnpm install && pnpm build            # 9 道 gate 全部通过
# CLI（Linux / Windows）：pnpm deploy 注入式、hoisted、ignore-scripts，补齐 workspace peer 依赖，配官方 Node 24.22.1 运行时
pnpm --filter @deepseek-ai/dsh deploy --config.inject-workspace-packages=true --config.node-linker=hoisted --config.ignore-scripts=true <dir>
/workspace/dsh/audit/fill-peers.sh <dir>                        # 补齐 23 个缺失的 @deepseek-ai/* peer
# Windows 桌面（Linux 交叉、免签、目录版）
DSH_OFFLINE_CROSS_WIN=1 pnpm --dir apps/desktop run package:win:x64:unsigned --dir
```
