# 2026-11-18 Fork 独立发布配置

- 版本：v1.0.3
- 仓库：l1905/cursor-byok-go（fork 自 leookun/cursor-byok）

## 需求

fork 独立维护后，需要在本仓库通过 GitHub Actions 构建并发布 Release，已安装的应用也要从本仓库获取更新，不再跟随上游。

## 改动

```
apps/desktop/src-tauri/
├── tauri.conf.json        # updater.pubkey 换成本仓库的签名公钥；endpoints 指向 l1905/cursor-byok-go
└── src/update/mod.rs      # Windows 便携版更新地址指向 l1905/cursor-byok-go
apps/desktop/              # 桌面端版本号 1.0.2 → 1.0.3
```

仓库配置（不在代码中）：

| 项 | 值 |
|---|---|
| Actions secret `TAURI_SIGNING_PRIVATE_KEY` | 本地 `.tauri/cursor-byok.local.key` 的内容（无密码，所以不需要配置 `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`） |
| 公钥 ID | `717064538CFC6B58` |

## 发布流程

```
1. 同步改动 3 个 manifest 里的版本号（tauri.conf.json / package.json / src-tauri/Cargo.toml）和 2 个 lock 文件
2. 提交并 push 到 main
3. git tag -a vX.Y.Z && git push origin vX.Y.Z
   └─▶ .github/workflows/release.yml
         prepare → publish（Linux / Windows / macOS arm64 / macOS x64）→ finalize（发布为 Latest）
```

## 注意

- **私钥不能丢**：`.tauri/cursor-byok.local.key` 已被 gitignore，必须另外备份。丢失后，已安装的应用不会接受任何新签名的更新。
- **与上游断开**：安装 1.0.3 及以后的版本只从本仓库更新。要用上游的修复，需要先合并到本仓库再发布新版本。
- `v1.0.2` 只是一个 tag，没有对应的 Release（当时 fork 还没配置签名密钥）。
