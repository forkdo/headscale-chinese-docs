Headscale 实现了与 Tailscale.com 相同的策略 ACL，但已适配到自托管环境。

例如，在定义组时，不能引用用户，而必须使用用户（这等同于 Tailscale.com 中的用户/登录账户）。

如需更多信息，请查看 https://tailscale.com/kb/1018/acls/。

使用 ACL 时，用户边界规则不再适用。只要 ACL 允许，所有机器（无论属于哪个用户）都可以与其他主机通信。

## ACL 配置

要在 Headscale 中启用并配置 ACL，请在 `config.yaml` 中的 `policy.path` 键中指定 ACL 策略文件的路径。

ACL 策略文件必须使用 [huJSON](https://github.com/tailscale/hujson) 格式编写。

关于如何编写这些策略的信息可在 [此处](https://tailscale.com/kb/1018/acls/) 找到。

更新 ACL 文件后，请重新加载或重启 Headscale。Headscale 可通过其 systemd 服务重新加载
（`sudo systemctl reload headscale`）或通过向主进程发送 SIGHUP 信号（`sudo kill -HUP $(pidof headscale)`）来重新加载。Headscale 在每次重新加载后都会记录 ACL 策略处理的结果。

## 简单示例

- [**允许所有**](https://tailscale.com/kb/1192/acl-samples#allow-all-default-acl)：如果您定义了 ACL 文件但完全省略了其内容中的 `"acls"` 字段，Headscale 将默认为“允许所有”策略。这意味着连接到您的 tailnet 的所有设备都可以自由相互通信。

    ```json
    {}
    ```

- [**拒绝所有**](https://tailscale.com/kb/1192/acl-samples#deny-all)：要阻止 tailnet 内的所有通信，您可以在策略文件中的 `"acls"` 字段包含一个空数组。

    ```json
    {
      "acls": []
    }
    ```

## 复杂示例

让我们为一家小型企业构建一个更复杂的 ACL 使用示例（ACL 在此处可能最有用）。

我们有一家小型公司，包括老板、管理员、两名开发人员和一名实习生。

老板应能访问所有服务器，但不能访问用户主机。管理员
也应能访问所有主机，但其权限应被限制为维护主机（仅作示例）。开发人员可以在开发主机上执行任何操作，但在生产主机上只能“观察”。实习生只能与开发服务器交互。

还有一个额外的服务器充当路由器，将 VPN 用户连接到内部网络 `10.20.0.0/16`。开发人员必须能够访问这些内部资源。

每位用户至少有一台连接到网络的设备，还有一些服务器。

- database.prod
- database.dev
- app-server1.prod
- app-server1.dev
- billing.internal
- router.internal

![ACL 实现示例](../assets/images/headscale-acl-network.png)

在[注册服务器](../usage/getting-started.md#register-a-node)时，
需要添加标志 `--advertise-tags=tag:<tag1>,tag:<tag2>`，注册服务器的用户
应被允许执行此操作。由于任何人均可为服务器添加标签，因此标签检查在 headscale 服务器上进行，仅应用有效的标签。如果注册标签的用户被允许执行此操作，则该标签是有效的。

以下 ACL 用于实现上述相同的权限：

```json title="acl.json"
{
  // groups 是具有共同作用域的用户集合。用户可以属于多个组
  // 组不能由其他组组成
  "groups": {
    "group:boss": ["boss@"],
    "group:dev": ["dev1@", "dev2@"],
    "group:admin": ["admin1@"],
    "group:intern": ["intern1@"]
  },
  // tailscale 中的 tagOwners 是标签与允许在服务器上设置此标签的人员之间的关联。
  // 这在[此处](https://tailscale.com/kb/1068/acl-tags#defining-a-tag)有文档说明
  // 在[此处](https://tailscale.com/blog/rbac-like-it-was-meant-to-be/)有解释
  "tagOwners": {
    // 管理员可以将服务器标记为生产环境
    "tag:prod-databases": ["group:admin"],
    "tag:prod-app-servers": ["group:admin"],

    // 老板可以将任何服务器标记为内部
    "tag:internal": ["group:boss"],

    // 开发人员和管理员可以为开发用途添加服务器
    "tag:dev-databases": ["group:admin", "group:dev"],
    "tag:dev-app-servers": ["group:admin", "group:dev"]

    // 实习生不能添加服务器
  },
  // 主机应使用其 IP 地址和子网掩码定义。
  // 要定义单个主机，请使用 /32 掩码。此处不能使用 DNS 条目，
  // 因为它们容易被通过替换其 IP 地址来劫持。
  // 有关更多信息，请参见 https://github.com/tailscale/tailscale/issues/3800。
  "hosts": {
    "postgresql.internal": "10.20.0.2/32",
    "webservers.internal": "10.20.10.1/29"
  },
  "acls": [
    // 老板可以访问所有服务器
    {
      "action": "accept",
      "src": ["group:boss"],
      "dst": [
        "tag:prod-databases:*",
        "tag:prod-app-servers:*",
        "tag:internal:*",
        "tag:dev-databases:*",
        "tag:dev-app-servers:*"
      ]
    },

    // 管理员只能访问服务器的管理端口，即 tcp/22
    {
      "action": "accept",
      "src": ["group:admin"],
      "proto": "tcp",
      "dst": [
        "tag:prod-databases:22",
        "tag:prod-app-servers:22",
        "tag:internal:22",
        "tag:dev-databases:22",
        "tag:dev-app-servers:22"
      ]
    },

    // 我们还允许管理员 ping 服务器
    {
      "action": "accept",
      "src": ["group:admin"],
      "proto": "icmp",
      "dst": [
        "tag:prod-databases:*",
        "tag:prod-app-servers:*",
        "tag:internal:*",
        "tag:dev-databases:*",
        "tag:dev-app-servers:*"
      ]
    },

    // 开发人员可以访问数据库服务器和应用服务器的所有端口
    // 他们只能查看生产环境中的应用服务器，且无法访问生产环境中的数据库服务器
    {
      "action": "accept",
      "src": ["group:dev"],
      "dst": [
        "tag:dev-databases:*",
        "tag:dev-app-servers:*",
        "tag:prod-app-servers:80,443"
      ]
    },
    // 开发人员可以通过路由器访问内部网络。
    // 内部网络由 HTTPS 端点和 Postgresql
    // 数据库服务器组成。
    {
      "action": "accept",
      "src": ["group:dev"],
      "dst": ["10.20.0.0/16:443,5432"]
    },

    // 服务器应能通过 tcp/5432 与数据库通信。数据库不应能主动连接到
    // 应用服务器
    {
      "action": "accept",
      "src": ["tag:dev-app-servers"],
      "proto": "tcp",
      "dst": ["tag:dev-databases:5432"]
    },
    {
      "action": "accept",
      "src": ["tag:prod-app-servers"],
      "dst": ["tag:prod-databases:5432"]
    },

    // 实习生只能以只读模式访问 dev-app-servers
    {
      "action": "accept",
      "src": ["group:intern"],
      "dst": ["tag:dev-app-servers:80,443"]
    },

    // 允许用户通过 autogroup:self 访问其自己的设备（下面会更详细地介绍性能影响）
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["autogroup:self:*"]
    }
  ]
}
```

## 自动分组

Headscale 支持多个自动分组，这些分组会自动包含具有特定属性的用户、目标或设备。自动分组提供了一种便捷的方式来编写 ACL 规则，无需手动列出单个用户或设备。

### `autogroup:internet`

允许通过[退出节点](routes.md#exit-node)访问互联网。只能在 ACL 目标中使用。

```json
{
  "action": "accept",
  "src": ["group:users"],
  "dst": ["autogroup:internet:*"]
}
```

### `autogroup:member`

包含所有[个人（未标记）设备](registration.md/#identity-model)。

```json
{
  "action": "accept",
  "src": ["autogroup:member"],
  "dst": ["tag:prod-app-servers:80,443"]
}
```

### `autogroup:tagged`

包含所有[至少有一个标签](registration.md/#identity-model)的设备。

```json
{
  "action": "accept",
  "src": ["autogroup:tagged"],
  "dst": ["tag:monitoring:9090"]
}
```

### `autogroup:self`
**(EXPERIMENTAL)**

!!! warning "当前 `autogroup:self` 的实现效率不高"

包含源和目标设备上的用户相同的设备。不包括已标记的设备。只能在 ACL 目标中使用。

```json
{
  "action": "accept",
  "src": ["autogroup:member"],
  "dst": ["autogroup:self:*"]
}
```
*在大型部署中使用 `autogroup:self` 可能会导致 Headscale 协调器服务器的性能下降，因为过滤规则必须按节点编译而不是全局编译，且当前实现效率不高。*

如果遇到性能问题，请考虑使用更具体的 ACL 规则或限制 `autogroup:self` 的使用。
```json
{
  // 以下规则允许内部用户与其自己的节点通信，在 autogroup:self 导致性能问题时使用。
  { "action": "accept", "src": ["boss@"], "dst": ["boss@:*"] },
  { "action": "accept", "src": ["dev1@"], "dst": ["dev1@:*"] },
  { "action": "accept", "src": ["dev2@"], "dst": ["dev2@:*"] },
  { "action": "accept", "src": ["admin1@"], "dst": ["admin1@:*"] },
  { "action": "accept", "src": ["intern1@"], "dst": ["intern1@:*"] }
}
```

### `autogroup:nonroot`

在 Tailscale SSH 规则中使用，允许访问任何非 root 用户。只能在 SSH 规则的 `users` 字段中使用。

```json
{
  "action": "accept",
  "src": ["autogroup:member"],
  "dst": ["autogroup:self"],
  "users": ["autogroup:nonroot"]
}
```
