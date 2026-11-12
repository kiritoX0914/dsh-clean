# DSH Clean Edition

English | [中文](README.zh-CN.md)

**An unofficial, telemetry-free, login-free, update-free community build of [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) `0.2.1-rc.2`.**

DSH Clean Edition is a carefully patched, from-source build of DeepSeek Harness. The product still works as a local agent harness — CLI, Web UI, and a portable Windows desktop — but the official account, auto-update, telemetry, analytics, and silent phone-home paths have been cut. Bring your own API key, or point it at any OpenAI-compatible (or Anthropic Messages-compatible) endpoint. On a default launch it does not talk to DeepSeek.

> **Unofficial community build.** This project is **not affiliated with, endorsed by, or supported by DeepSeek**. "DeepSeek" and "DeepSeek Harness" are trademarks of their respective owners. The upstream project is MIT-licensed; this repository redistributes the original source with clearly marked modifications, plus independently built binaries. Use at your own risk.

Upstream tag: [`dsh-v0.2.1-rc.2`](https://github.com/deepseek-ai/deepseek-harness/releases/tag/dsh-v0.2.1-rc.2) (commit `687f282e`). The official README is preserved at [docs/UPSTREAM_README.md](docs/UPSTREAM_README.md). A file-by-file change list (Chinese) is in [docs/OFFLINE_CHANGES.md](docs/OFFLINE_CHANGES.md). The full diff against the official tag is published as [`dsh-offline-0.2.1-rc.2.patch`](https://github.com/kiritoX0914/dsh-clean/releases/tag/v0.2.1-rc.2-clean) with the release.

Every patched call site is marked with an `OFFLINE BUILD` comment. Original implementations are kept as `*Unused` functions where practical, so you can audit exactly what was disabled instead of hunting through a rewrite.

## What was removed

These paths cannot reach official infrastructure, even if a leftover config tries to turn them back on:

| Area | What was cut |
| --- | --- |
| **Telemetry / analytics** | OTLP log and event export, session telemetry (the official collector at `dsh-otel-collector.deepseeksvc.com`), desktop product telemetry, frontend product analytics, session-log and plugin-inventory fields that were attached to official API requests. Telemetry plugins are disabled in the bundle patches, and the launcher also sets `DSH_TELEMETRY_DISABLED=1`. |
| **Identity headers** | Persistent anonymous user ID no longer written to disk (a process-local random UUID is used instead). Request headers `x-deepseek-harness-user-id`, session-id, and compact headers are not sent. Product attribution identifies as `dsh-offline` / `local`. Default User-Agent is `dsh-offline/...`. |
| **Account login** | Official account / inference origins resolve to `.invalid` hosts that never resolve. Platform auth throws before any `fetch`. The sign-in UI, "Log in" / "Contact us" menu entries, welcome-page auth buttons, and the unauthenticated account section are gone. Onboarding copy tells you to add an API key or an OpenAI-compatible provider. |
| **Auto-update** | The desktop update coordinator never calls `electron-updater`. Forced-update policy is not embedded. Packages are built with `publish: null` and ship **without** `app-update.yml`. |
| **Plugin-market startup ping** | The plugin manager no longer probes npm / npmmirror at startup. |

## What can still reach the network (only if you opt in)

A clean edition is not a network sandbox. Default startup does not contact DeepSeek, but the harness will use the network when **you** ask it to:

- **Model APIs.** The DeepSeek provider still defaults to `https://api.deepseek.com/anthropic` (and `/v1/tools`) when you set `DEEPSEEK_API_KEY` and actually use that provider. Set `DEEPSEEK_BASE_URL` to any Anthropic Messages-compatible server, or add an OpenAI-compatible provider in **Settings → Models**. This is your key, your endpoint — not telemetry.
- **Web search / fetch providers**, including the default `deepseek-official` provider (uses your `DEEPSEEK_API_KEY`) and optional ones such as Exa or Perplexity, only when you enable them.
- **Other built-in model providers** (OpenAI, Anthropic, Google, and so on), only when you select them.
- **Chain-of-thought translation** (Google / Bing) and **speech-model downloads** (Hugging Face / hf-mirror), only when you turn those features on.
- **Plugin installs** from npm (`registry.npmjs.org` / `registry.npmmirror.com`) or GitHub, only when you install a plugin yourself.

A few official URLs remain as documentation strings or packaging-script constants (`api-docs.deepseek.com`, `www.deepseek.com`, `download.deepseek.com`). They are not contacted at runtime by this unsigned build.

On the build machine, `dsh web` under a Node network-audit hook made **no outbound connections** (only `dns.lookup 127.0.0.1`). A `dsh headless` run against a local mock model server talked only to `127.0.0.1:18080`, with no user-id / session-id headers and User-Agent `dsh-offline/0.2.1-rc.2 (+local)`.

## Downloads

Prebuilt packages for tag [`v0.2.1-rc.2-clean`](https://github.com/kiritoX0914/dsh-clean/releases/tag/v0.2.1-rc.2-clean):

| File | Size | SHA-256 |
| --- | ---: | --- |
| [dsh-offline-0.2.1-rc.2-linux-x64-cli.tar.gz](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2-linux-x64-cli.tar.gz) | 164.2 MiB | `398d21ed097ad99b1ec5be8e1df45d50ae7d9004b55587d6d5239dd198e40349` |
| [dsh-offline-0.2.1-rc.2-win-x64-cli.zip](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2-win-x64-cli.zip) | 188.5 MiB | `e1df673b36e56442eabccf1858f32e6dbd65a64b43c42af5f6db0e4e3293f71a` |
| [dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip) | 390.9 MiB | `5e6ae3b463331542d64271d18b3deb4efc44e7ba78bf1c836913709a196f8696` |
| [dsh-offline-0.2.1-rc.2.patch](https://github.com/kiritoX0914/dsh-clean/releases/download/v0.2.1-rc.2-clean/dsh-offline-0.2.1-rc.2.patch) | 52.4 KiB | `c2e0c765ca7cfbf9fae7ea9a2d3b648df3251ce4a42680078db9844971b5f69a` |

The same release also ships `SHA256SUMS`, the Ed25519 detached signature `SHA256SUMS.sig`, the public key [`dsh-offline-signing.pub.pem`](dsh-offline-signing.pub.pem), [docs/VERIFY.md](docs/VERIFY.md), and [docs/OFFLINE_CHANGES.md](docs/OFFLINE_CHANGES.md).

## Quick start

Set a model credential first (or add a provider in the UI after launch):

```sh
export DEEPSEEK_API_KEY=sk-...
# optional: any Anthropic Messages-compatible endpoint
# export DEEPSEEK_BASE_URL=https://your-server.example/anthropic
```

You can also leave the environment empty and add an OpenAI-compatible provider later under **Settings → Models** (Base URL + API key). `DSH_HOME` optionally overrides the data directory. Nothing is required just to start the UI.

### Linux CLI

```sh
tar -xzf dsh-offline-0.2.1-rc.2-linux-x64-cli.tar.gz
cd dsh-offline-0.2.1-rc.2-linux-x64
./dsh --version          # 0.2.1-rc.2
./dsh web                # Web UI, default http://127.0.0.1:3080
./dsh tui
./dsh headless "your task"
```

The archive includes its own Node.js 24 runtime. `./dsh` is the launcher.

### Windows CLI

Unzip `dsh-offline-0.2.1-rc.2-win-x64-cli.zip` and run `dsh.cmd` (or `dsh.ps1`) from `dsh-offline-0.2.1-rc.2-win-x64\`:

```bat
dsh.cmd --version
dsh.cmd web
```

### Windows Desktop (portable)

This is an unpacked portable directory, not an NSIS installer (the installer UI cannot be built in the Linux cross-build).

1. Unzip `dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip` to a **short path**, for example `C:\DSH`. Some nested paths inside the tree are about 210 characters long; a deep unzip location can hit `MAX_PATH`.
2. Run `DSH Offline.exe`.
3. Windows SmartScreen will warn about an unknown publisher — this build is **not Authenticode-signed**. Choose **More info → Run anyway** only if you trust the checksums below.
4. Add your API key, or open **Settings → Models** and configure an OpenAI-compatible endpoint.

App ID is `local.dsh.offline`. The window title / product name is **DSH Offline**.

## Verify the release

You need OpenSSL 3.x. On Windows, Git for Windows typically provides `openssl`.

```sh
# 1) Verify the Ed25519 signature on SHA256SUMS
openssl pkeyutl -verify -pubin -inkey dsh-offline-signing.pub.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
# expect: Signature Verified Successfully

# 2) Verify every artifact hash
sha256sum -c SHA256SUMS
```

Public key fingerprint (SHA-256 of the DER encoding):

```
29932eaf6e032fc9be5cd8b7c0529762120618ee34f924850c061f378d63d326
```

The same key is in this repository as [`dsh-offline-signing.pub.pem`](dsh-offline-signing.pub.pem). The signature only proves the files match this build; it is **not** an Authenticode signature and is **not** an official endorsement.

On Windows PowerShell, if you only want to check one hash:

```powershell
Get-FileHash .\dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip -Algorithm SHA256
# compare with the corresponding line in SHA256SUMS
```

## Build from source

Requirements:

- Node.js 24 (the release binaries were built with **Node 24.22.1**; `package.json` engines allow `^22.19.0 || >=24.0.0`)
- **pnpm 11.28.5** (`packageManager` field; Corepack can pin this)

```sh
git clone https://github.com/kiritoX0914/dsh-clean.git
cd dsh-clean
git checkout v0.2.1-rc.2-clean
pnpm install
pnpm build          # the release build ran the full gate set and it passed
pnpm dsh web        # run the Web UI from the checkout
pnpm dsh --version
```

### Linux / Windows CLI packages

The published CLI archives were assembled with an injected, hoisted `pnpm deploy` of `@deepseek-ai/dsh`, then the official Node 24.22.1 runtime was copied in:

```sh
pnpm --filter @deepseek-ai/dsh deploy \
  --config.inject-workspace-packages=true \
  --config.node-linker=hoisted \
  --config.ignore-scripts=true \
  ./dist/cli-linux   # or a Windows staging directory

# pnpm's injected deploy skipped 23 workspace @deepseek-ai/* peer
# dependencies; the release build filled those from the workspace
# packages with `pnpm pack` and unpacked them into node_modules.
```

If you need `landlock-run` and `musl-gcc` is not available, the release build compiled it on the Linux host with `gcc -static -Os` from `native/system/packages/entry` (`packages/entry/src/main.c`).

### Windows Desktop, cross-built from Linux

```sh
export DSH_OFFLINE_CROSS_WIN=1
export DSH_DESKTOP_APP_ID=local.dsh.offline
pnpm --dir apps/desktop run package:win:x64:unsigned --dir
```

`DSH_OFFLINE_CROSS_WIN=1` is the patch that allows an unsigned win-x64 directory build on a Linux x64 host: it skips Visual Studio / PowerShell toolchain checks, Linux-impossible `electron.exe` smoke tests, `rcedit`, and the "official build record" check. The output is the portable unpacked directory (zipped for the release as `dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip`), not an NSIS installer.

## Known limitations

- **Windows binaries were cross-compiled on Linux and have not been run on a real Windows machine.** The Windows CLI JavaScript graph was resolved on Linux up to native-module load (it stops there because only win32 natives are shipped, which is expected). The desktop zip was inspected: `app.asar` has `dshDesktopAppId: local.dsh.offline` and no `dshMandatoryUpdatePolicy`; `resources/` has no `app-update.yml`; the updater in `lib/main.js` is hard-disabled; patched runtime files match the Linux CLI byte-for-byte; native addons are win32-x64.
- **Not Authenticode-signed.** SmartScreen will show an unknown publisher. The desktop exe icon and version resources are still the Electron defaults — the Linux cross-build has no Wine/`rcedit` step.
- **The desktop zip includes some leftover linux-x64 native addons** needed so install scripts could run on the Linux build host. Windows will not load them; they only add size.
- **Custom endpoints were tested against a local mock server**, not against a production OpenAI or DeepSeek account from this build machine.
- This is a **prerelease** of a developer-preview upstream (`0.2.1-rc.2`). Breaking changes exist upstream already; this snapshot is frozen on that tag.
- Please review upstream [SAFETY.md](SAFETY.md) before running any agent harness with tool access to your machine.

## License

[MIT](LICENSE). Copyright (c) 2026 DeepSeek. Modifications Copyright (c) 2026 kiritoX0914.

The original copyright notice and permission text are preserved. Third-party licenses remain in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
