# 连接 Apple 客户端

本文档旨在说明用户如何将官方的 iOS 和 macOS [Tailscale](https://tailscale.com) 客户端与 headscale 配合使用。

!!! info "关于您的 headscale 实例的说明"

    有关如何连接 Apple 设备的信息端点也可在运行中的实例上的 `/apple` 路径下获取。

## iOS

### 安装

从 [App Store](https://apps.apple.com/app/tailscale/id1470499037) 安装官方的 Tailscale iOS 客户端。

### 配置 headscale URL

- 打开 Tailscale 应用
- 点击右上角的账户图标，选择 `登录…`。
- 点击右上角的选项菜单按钮，选择 `使用自定义协调服务器`。
- 输入您的实例 URL（例如 `https://headscale.example.com`）
- 输入您的凭据并登录。现在 headscale 应该已在您的 iOS 设备上运行。

## macOS

### 安装

选择适用于 macOS 的任意一款可用的 [Tailscale 客户端](https://tailscale.com/docs/concepts/macos-variants) 并安装。

### 配置 headscale URL

#### 命令行

使用 Tailscale 的登录命令连接到您的 headscale 实例（例如 `https://headscale.example.com`）：

```
tailscale login --login-server <YOUR_HEADSCALE_URL>
```

#### 图形界面

- 按住 Option 键并点击菜单栏中的 Tailscale 图标，悬停在“调试”菜单上
- 在 `自定义登录服务器` 下，选择 `添加账户...`
- 输入您的 headscale 实例 URL（例如 `https://headscale.example.com`），然后点击 `添加账户`
- 按照浏览器中的登录流程操作

## tvOS

### 安装

从 [App Store](https://apps.apple.com/app/tailscale/id1470499037) 安装官方的 Tailscale tvOS 客户端。

!!! danger

    安装后**不要**打开 Tailscale 应用！

### 配置 headscale URL

- 打开“设置”（Apple tvOS 设置）> “应用” > “Tailscale”
- 在 `备用协调服务器 URL` 下，选择 `URL`
- 输入您的 headscale 实例 URL（例如 `https://headscale.example.com`），然后点击 `确定`
- 返回 tvOS 主屏幕
- 打开 Tailscale
- 点击 `安装 VPN 配置` 按钮，并通过点击 `允许` 按钮确认弹出的提示
- 扫描二维码并按照登录流程操作