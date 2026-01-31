# 注册方法

Headscale 支持多种节点注册方式。首选的注册方法取决于节点的身份以及您的使用场景。

## 身份模型

Tailscale 的身份模型区分个人节点和标签节点：

- 个人节点（或用户拥有节点）由个人拥有，通常指代终端用户设备，如笔记本电脑、工作站或手机。终端用户设备由单个用户管理。
- 标签节点（或服务型节点或非人类节点）为网络提供服务。常见示例包括 Web 服务器和数据库服务器。这些节点通常由一组用户管理。标签节点有一些额外的限制，例如，标签节点不允许通过 [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) 连接到个人节点。

Headscale 实现了 Tailscale 的身份模型，并区分个人节点和标签节点，其中个人节点由 Headscale 用户拥有，而标签节点由标签拥有。标签设备被归入特殊用户 `tagged-devices` 下。

## 注册方法

有两种主要的注册新节点的方法：[Web 认证](#web-authentication) 和 [使用预认证密钥注册](#pre-authenticated-key)。这两种方法都可以用于注册个人节点和标签节点。

### Web 认证

Web 认证是注册新节点的默认方法。它是交互式的，客户端发起注册，Headscale 管理员需要在节点被允许加入网络之前批准该新节点。节点可以通过以下方式批准：

- Headscale CLI（本文档中描述）
- [Headscale API](api.md)
- 或通过 [OpenID Connect](oidc.md) 委托给身份提供商

Web 认证依赖于 Headscale 用户的存在。使用 `headscale users` 命令创建一个新用户：

```console
headscale users create <USER>
```

=== "个人设备"

    运行 `tailscale up` 以登录您的个人设备：

    ```console
    tailscale up --login-server <YOUR_HEADSCALE_URL>
    ```

    通常，会打开一个包含进一步说明的浏览器窗口。该页面解释了如何在您的 Headscale 服务器上完成注册，并且还会打印批准节点所需的注册密钥：

    ```console
    headscale nodes register --user <USER> --key <REGISTRATION_KEY>
    ```

    恭喜，您的个人节点注册已完成，它应该在 `headscale nodes list` 的输出中显示为“在线”。"User" 列显示 `<USER>` 作为节点的所有者。

=== "标签设备"

    您的 Headscale 用户需要被授权注册标签设备。此授权在 [ACL](acls.md) 的 [`tagOwners`](https://tailscale.com/kb/1337/policy-syntax#tag-owners) 部分中指定。一个简单的示例如下：

    ```json title="用户 alice 可以注册带有 tag:server 标签的节点"
    {
      "tagOwners": {
        "tag:server": ["alice@"]
      },
      // 更多规则
    }
    ```

    运行 `tailscale up` 并提供至少一个标签以登录标签设备：

    ```console
    tailscale up --login-server <YOUR_HEADSCALE_URL> --advertise-tags tag:<TAG>
    ```

    通常，会打开一个包含进一步说明的浏览器窗口。该页面解释了如何在您的 Headscale 服务器上完成注册，并且还会打印批准节点所需的注册密钥：

    ```console
    headscale nodes register --user <USER> --key <REGISTRATION_KEY>
    ```

    Headscale 会检查 `<USER>` 是否被允许注册具有指定标签的节点，然后将新节点的所有权转移给特殊用户 `tagged-devices`。标签节点的注册已完成，它应该在 `headscale nodes list` 的输出中显示为“在线”。"User" 列显示 `tagged-devices` 作为节点的所有者。请参阅 "Tags" 列以查看分配的标签列表。

### 预认证密钥

使用预认证密钥（或认证密钥）注册是一种非交互式的注册新节点的方法。Headscale 管理员预先创建一个预认证密钥，然后可以使用此预认证密钥以非交互式方式注册节点。它最适合用于自动化。

=== "个人设备"

    个人节点总是分配给一个 Headscale 用户。使用 `headscale users` 命令创建一个新用户：

    ```console
    headscale users create <USER>
    ```

    使用 `headscale user list` 命令了解其 `<USER_ID>` 并为您的用户创建一个新的预认证密钥：

    ```console
    headscale preauthkeys create --user <USER_ID>
    ```

    上述命令会打印一个具有默认设置的预认证密钥（只能使用一次，有效期为一小时）。使用此认证密钥以非交互式方式注册节点：

    ```console
    tailscale up --login-server <YOUR_HEADSCALE_URL> --authkey <YOUR_AUTH_KEY>
    ```

    恭喜，您的个人节点注册已完成，它应该在 `headscale nodes list` 的输出中显示为“在线”。"User" 列显示 `<USER>` 作为节点的所有者。

=== "标签设备"

    创建一个新的预认证密钥并提供至少一个标签：

    ```console
    headscale preauthkeys create --tags tag:<TAG>
    ```

    上述命令会打印一个具有默认设置的预认证密钥（只能使用一次，有效期为一小时）。使用此认证密钥以非交互式方式注册节点。您不需要提供 `--advertise-tags` 参数，因为标签会自动从预认证密钥中读取：

    ```console
    tailscale up --login-server <YOUR_HEADSCALE_URL> --authkey <YOUR_AUTH_KEY>
    ```

    标签节点的注册已完成，它应该在 `headscale nodes list` 的输出中显示为“在线”。"User" 列显示 `tagged-devices` 作为节点的所有者。请参阅 "Tags" 列以查看分配的标签列表。