# 校验与签名说明 / How to verify

文件 / Files:
- `SHA256SUMS` — 所有发布文件的 SHA-256
- `SHA256SUMS.sig` — 对 `SHA256SUMS` 的 Ed25519 分离签名（本机 OpenSSL 生成的私钥签署，私钥不随包分发）
- `dsh-offline-signing.pub.pem` — 公钥

公钥指纹（DER 的 SHA-256）/ public key fingerprint:
`29932eaf6e032fc9be5cd8b7c0529762120618ee34f924850c061f378d63d326`

## Linux / macOS
```sh
openssl pkeyutl -verify -pubin -inkey dsh-offline-signing.pub.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
sha256sum -c SHA256SUMS
```
需要 OpenSSL 3.x。第一条应输出 `Signature Verified Successfully`。

## Windows (PowerShell)
```powershell
# 有 OpenSSL（如 Git for Windows 自带）时同上执行 openssl 命令；仅校验哈希：
Get-FileHash .\dsh-offline-0.2.1-rc.2-win-x64-desktop-portable.zip -Algorithm SHA256
# 与 SHA256SUMS 中对应行比较
```

## 重要 / Important
- 这是自签的发布校验签名，**不是 Windows Authenticode 代码签名**。Windows 运行 `DSH Offline.exe`、`node.exe` 时 SmartScreen 会提示“未知发布者 / Windows 已保护你的电脑”，点“更多信息 → 仍要运行”。
- 签名只证明文件与本次构建一致，不代表任何官方背书。
