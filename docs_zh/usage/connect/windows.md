# 连接 Windows 客户端

本文档旨在说明用户如何将官方 Windows [Tailscale](https://tailscale.com) 客户端与 headscale 配合使用。

!!! info "关于您的 headscale 实例的说明"

    您正在运行的实例上的 `/windows` 端点也提供了有关如何连接 Windows 设备的信息。

## 安装

下载 [官方 Windows 客户端](https://tailscale.com/download/windows) 并安装。

## 配置 headscale URL

打开命令提示符或 PowerShell，使用 Tailscale 的登录命令连接到您的 headscale 实例（例如 `https://headscale.example.com`）：

```
tailscale login --login-server <YOUR_HEADSCALE_URL>
```

按照打开的浏览器窗口中的说明完成配置。

## 故障排除

### 无人值守模式

默认情况下，Tailscale 的 Windows 客户端仅在用户登录时运行。如果您希望 Tailscale 始终保持运行状态，请启用“无人值守模式”：

- 单击 Tailscale 托盘图标，然后选择 `Preferences`
- 启用 `Run unattended`
- 确认“无人值守模式”消息

另请参阅 [在我未登录计算机时保持 Tailscale 运行](https://tailscale.com/kb/1088/run-unattended)

### 节点注册失败

如果您在 headscale 输出中看到如下重复消息：

```
[GIN] 2022/02/10 - 16:39:34 | 200 |    1.105306ms |       127.0.0.1 | POST     "/machine/redacted"
```

请启用 `DEBUG` 日志记录并查找：

```
2022-02-11T00:59:29Z DBG Machine registration has expired. Sending a authurl to register machine=redacted
```

这通常意味着上述注册表项未正确设置。

要重置并重新尝试，请务必执行以下操作：

1. 关闭 Tailscale 服务（或托盘中的客户端）
2. 删除 Tailscale 应用程序数据文件夹，该文件夹位于 `C:\Users\<USERNAME>\AppData\Local\Tailscale`，然后重新尝试连接。
3. 确保从 headscale 中删除 Windows 节点（以确保全新设置）
4. 在 Windows 机器上启动 Tailscale 并重新尝试登录。