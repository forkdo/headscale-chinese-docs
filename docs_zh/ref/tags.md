# 标签

Headscale 支持 Tailscale 标签。请阅读 [Tailscale 的标签文档](https://tailscale.com/kb/1068/tags) 以了解标签的工作原理以及如何使用它们。

标签可以在[节点注册](registration.md)期间应用：

- 使用 `--advertise-tags` 标志，参见[针对已标记设备的 Web 身份验证](registration.md#__tabbed_1_2)
- 使用带有标签的预认证密钥，参见[如何创建和使用它](registration.md#__tabbed_2_2)

管理员可以通过以下方式管理标签：

- Headscale CLI
- [Headscale API](api.md)

## 常见操作

### 管理节点的标签

运行 `headscale nodes list` 命令以列出节点的标签。

使用 `headscale nodes tag` 命令修改节点的标签。至少需要提供一个标签，多个标签可以用逗号分隔的列表形式提供。以下命令为 ID 为 1 的节点设置标签 `tag:server` 和 `tag:prod`：

```console
headscale nodes tag -i 1 -t tag:server,tag:prod
```

### 从个人节点转换为带标签的节点

使用 `headscale nodes tag` 命令将个人（用户拥有）节点转换为带标签的节点：

```console
headscale nodes tag -i <NODE_ID> -t <TAG>
```

现在该节点由特殊用户 `tagged-devices` 拥有，并被分配了指定的标签。

### 从带标签的节点转换为个人节点

带标签的节点可以通过以下命令重新认证，从而恢复为个人（用户拥有）节点：

```console
tailscale up --login-server <YOUR_HEADSCALE_URL> --advertise-tags= --force-reauth
```

通常，系统会打开一个浏览器窗口并显示进一步的操作说明。该页面说明了如何在 Headscale 服务器上完成注册，并会打印出批准节点所需的注册密钥：

```console
headscale nodes register --user <USER> --key <REGISTRATION_KEY>
```

所有之前分配的标签都会被移除，节点现在由上述命令中指定的用户拥有。