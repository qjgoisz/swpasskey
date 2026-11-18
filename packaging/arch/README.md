# 在 Arch Linux 上构建 swpasskey

本目录提供 x86_64 的 PKGBUILD 和 .SRCINFO。按 Arch 的惯例先准备 base-devel。
在本目录以普通用户运行：

```sh
makepkg -s
```

makepkg 会校验固定提交的源码归档和 OpenSSL 兼容补丁。CMake 使用 Arch 提供的
Catch2、OpenSSL、tpm2-tss、libsecret 和 libnotify。Catch2 只用于构建与测试。

安装需要管理员权限。在同一目录运行：

```sh
pacman -U ./swpasskey-0.2.0.r2.g68a40f5-1-x86_64.pkg.tar.zst
```

首次安装后重启并登录图形桌面，使 UHID 和 udev 设备权限生效。以普通用户检查：

```sh
systemctl --user status swpasskeyd.service
swpasskeyctl stats
journalctl --user -u swpasskeyd.service -b -n 30
```

确认后端为 `tpm2`，日志出现 `presence_notify_ready`。通知服务需支持操作按钮。
如需手册，安装 Arch 的 man-db 后运行 `man swpasskeyctl` 或
`man -L zh_CN swpasskeyctl`。

若迁移已有凭据，保留原有 `$XDG_DATA_HOME/swpasskey`（默认
`~/.local/share/swpasskey`）和对应的桌面 Secret Service 加密密钥。
TPM 凭据也需要原来的 TPM 和未重置的 TPM 状态。安装包不会重置 PIN。
已有凭据却遇到密钥环或 TPM 错误时，先停服务并检查原数据和日志。
卸载前可用 `systemctl --user stop swpasskeyd.service` 停止服务，再以管理员身份
执行 `pacman -R swpasskey`。用户数据由用户自行保留。
