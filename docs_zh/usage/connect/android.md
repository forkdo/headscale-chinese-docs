# 连接 Android 客户端

本文档旨在展示用户如何将官方的 Android [Tailscale](https://tailscale.com) 客户端与 headscale 配合使用。

## 安装

从 [Google Play Store](https://play.google.com/store/apps/details?id=com.tailscale.ipn) 或 [F-Droid](https://f-droid.org/packages/com.tailscale.ipn/) 安装官方 Tailscale Android 客户端。

## 通过网页认证连接

- 打开应用并选择右上角的设置菜单
- 点击 `Accounts`
- 在右上角的菜单图标（三个点）中选择 `Use an alternate server`
- 输入您的服务器 URL（例如 `https://headscale.example.com`）并按照说明操作
- 一旦 headscale 上的节点注册完成，客户端会自动连接。在此之前，服务器日志中不会显示任何内容。

## 使用预认证密钥连接

- 打开应用并选择右上角的设置菜单
- 点击 `Accounts`
- 在右上角的菜单图标（三个点）中选择 `Use an alternate server`
- 输入您的服务器 URL（例如 `https://headscale.example.com`）。如果出现登录提示，请关闭它并继续
- 打开右上角的设置菜单
- 点击 `Accounts`
- 在右上角的菜单图标（三个点）中选择 `Use an auth key`
- 输入您从 headscale [生成的预认证密钥](../../ref/registration.md#pre-authenticated-key)
- 如需要，请在主屏幕上点击 `Log in`。现在您应该已连接到您的 headscale。