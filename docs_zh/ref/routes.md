# 路由

Headscale 支持路由通告，可用于管理 tailnet 的[子网路由器](https://tailscale.com/kb/1019/subnets)和[出口节点](https://tailscale.com/kb/1103/exit-nodes)。

- [子网路由器](#subnet-router)可用于将现有网络（例如虚拟私有云或本地网络）连接到您的 tailnet。使用子网路由器可以访问无法安装 Tailscale 的设备，或逐步部署 Tailscale。
- [出口节点](#exit-node)可用于为另一个 Tailscale 节点路由所有互联网流量。使用它可以在不可信的 Wi-Fi 上安全地访问互联网，或访问期望来自特定 IP 地址的流量的在线服务。

## 子网路由器

子网路由器的设置需要双重确认，一次来自子网路由器本身，一次在控制服务器上允许其在 tailnet 中使用。可选地，使用 [`autoApprovers` 自动批准来自子网路由器的路由](#automatically-approve-routes-of-a-subnet-router)。

### 设置子网路由器

#### 将节点配置为子网路由器

注册一个节点并通告它应处理的以逗号分隔的路由列表：

```console
$ sudo tailscale up --login-server <YOUR_HEADSCALE_URL> --advertise-routes=10.0.0.0/8,192.168.0.0/24
```

如果节点已经注册，可以使用以下命令通告新路由或更新先前通告的路由：

```console
$ sudo tailscale set --advertise-routes=10.0.0.0/8,192.168.0.0/24
```

最后，[启用 IP 转发](#enable-ip-forwarding)以路由流量。

#### 在控制服务器上启用子网路由器

可以使用 `headscale nodes list-routes` 命令显示 tailnet 的路由。主机名为 `myrouter` 的子网路由器通告了 IPv4 网络 `10.0.0.0/8` 和 `192.168.0.0/24`。这些路由需要被批准才能使用。

```console
$ headscale nodes list-routes
ID | Hostname | Approved | Available      | Serving (Primary)
1  | myrouter |          | 10.0.0.0/8     |
   |          |          | 192.168.0.0/24 |
```

通过指定以逗号分隔的路由列表来批准子网路由器的所有所需路由：

```console
$ headscale nodes approve-routes --identifier 1 --routes 10.0.0.0/8,192.168.0.0/24
Node updated
```

现在，节点 `myrouter` 可以为 tailnet 路由 IPv4 网络 `10.0.0.0/8` 和 `192.168.0.0/24`。

```console
$ headscale nodes list-routes
ID | Hostname | Approved       | Available      | Serving (Primary)
1  | myrouter | 10.0.0.0/8     | 10.0.0.0/8     | 10.0.0.0/8
   |          | 192.168.0.0/24 | 192.168.0.0/24 | 192.168.0.0/24
```

#### 使用子网路由器

要在节点上接受子网路由器通告的路由：

```console
$ sudo tailscale set --accept-routes
```

请参考官方的 [Tailscale 文档](https://tailscale.com/kb/1019/subnets#use-your-subnet-routes-from-other-devices) 了解如何在不同操作系统上使用子网路由器。

### 使用 ACL 限制子网路由器的使用

子网路由器通告的路由对 tailnet 中的节点可用。默认情况下，如果没有启用 ACL，所有节点都可以接受和使用此类路由。配置 ACL 以明确管理谁可以使用路由。

下面的 ACL 片段定义了三个主机：子网路由器 `router`、常规节点 `node` 和可通过子网路由器 `router` 上的路由访问的内部服务 `service.example.net`。它允许节点 `node` 访问可通过子网路由器访问的 `service.example.net` 的 80 和 443 端口。对子网路由器本身的访问被拒绝。

```json title="Access the routes of a subnet router without the subnet router itself"
{
  "hosts": {
    // the router is not referenced but announces 192.168.0.0/24"
    "router": "100.64.0.1/32",
    "node": "100.64.0.2/32",
    "service.example.net": "192.168.0.1/32"
  },
  "acls": [
    {
      "action": "accept",
      "src": ["node"],
      "dst": ["service.example.net:80,443"]
    }
  ]
}
```

### 自动批准子网路由器的路由

子网路由器的初始设置通常需要在控制服务器上手动批准其通告的路由，然后 tailnet 中的节点才能使用它们。Headscale 支持 ACL 的 `autoApprovers` 部分来自动批准子网路由器提供的路由。

下面的 ACL 片段定义了由用户 `alice` 拥有的标签 `tag:router`。此标签用于 `autoApprovers` 部分的 `routes`。一旦通告了标签 `tag:router` 的子网路由器通告 IPv4 路由 `192.168.0.0/24`，该路由将自动被批准。

```json title="Subnet routers tagged with tag:router are automatically approved"
{
  "tagOwners": {
    "tag:router": ["alice@"]
  },
  "autoApprovers": {
    "routes": {
      "192.168.0.0/24": ["tag:router"]
    }
  },
  "acls": [
    // more rules
  ]
}
```

在加入 tailnet 时，从也通告标签 `tag:router` 的子网路由器通告路由 `192.168.0.0/24`：

```console
$ sudo tailscale up --login-server <YOUR_HEADSCALE_URL> --advertise-tags tag:router --advertise-routes 192.168.0.0/24
```

请参见[官方 Tailscale 文档](https://tailscale.com/kb/1337/acl-syntax#autoapprovers)以获取有关自动批准者的更多信息。

## 出口节点

出口节点的设置需要双重确认，一次来自出口节点本身，一次在控制服务器上允许其在 tailnet 中使用。可选地，使用 [`autoApprovers` 自动批准出口节点](#automatically-approve-an-exit-node-with-auto-approvers)。

### 设置出口节点

#### 将节点配置为出口节点

注册一个节点并使其通告自己为出口节点：

```console
$ sudo tailscale up --login-server <YOUR_HEADSCALE_URL> --advertise-exit-node
```

如果节点已经注册，可以像这样通告出口功能：

```console
$ sudo tailscale set --advertise-exit-node
```

最后，[启用 IP 转发](#enable-ip-forwarding)以路由流量。

#### 在控制服务器上启用出口节点

可以使用 `headscale nodes list-routes` 命令显示 tailnet 的路由。出口节点可以通过其通告的路由来识别：IPv4 为 `0.0.0.0/0`，IPv6 为 `::/0`。主机名为 `myexit` 的出口节点已经可用，但需要被批准：

```console
$ headscale nodes list-routes
ID | Hostname | Approved | Available | Serving (Primary)
1  | myexit   |          | 0.0.0.0/0 |
   |          |          | ::/0      |
```

对于出口节点，批准 IPv4 或 IPv6 路由中的任意一个就足够了。另一个将自动被批准。

```console
$ headscale nodes approve-routes --identifier 1 --routes 0.0.0.0/0
Node updated
```

现在，节点 `myexit` 已被批准为 tailnet 的出口节点：

```console
$ headscale nodes list-routes
ID | Hostname | Approved  | Available | Serving (Primary)
1  | myexit   | 0.0.0.0/0 | 0.0.0.0/0 | 0.0.0.0/0
   |          | ::/0      | ::/0      | ::/0
```

#### 使用出口节点

现在可以在节点上使用出口节点：

```console
$ sudo tailscale set --exit-node myexit
```

请参考官方的 [Tailscale 文档](https://tailscale.com/kb/1103/exit-nodes#use-the-exit-node) 了解如何在不同操作系统上使用出口节点。

### 使用 ACL 限制出口节点的使用

出口节点提供给 tailnet 中的所有节点。默认情况下，如果没有启用 ACL，tailnet 中的所有节点都可以选择和使用出口节点。在 ACL 规则中配置 `autogroup:internet` 以限制谁可以使用_任何_可用的出口节点。

```json title="Example use of autogroup:internet"
{
  "acls": [
    {
      "action": "accept",
      "src": ["..."],
      "dst": ["autogroup:internet:*"]
    }
  ]
}
```

### 按用户或组限制对出口节点的访问

用户可以使用 `autogroup:internet` 使用_任何_可用的出口节点。或者，下面的 ACL 片段为每个用户分配一个特定的出口节点，同时隐藏所有其他出口节点。用户 `alice` 只能使用出口节点 `exit1`，而用户 `bob` 只能使用出口节点 `exit2`。

```json title="Assign each user a dedicated exit node"
{
  "hosts": {
    "exit1": "100.64.0.1/32",
    "exit2": "100.64.0.2/32"
  },
  "acls": [
    {
      "action": "accept",
      "src": ["alice@"],
      "dst": ["exit1:*"]
    },
    {
      "action": "accept",
      "src": ["bob@"],
      "dst": ["exit2:*"]
    }
  ]
}
```

!!! warning

    - 上述实现是 Headscale 特有的，一旦[支持 `via`](https://github.com/juanfont/headscale/issues/2409) 可用，可能会被移除。
    - 请注意，用户也可以连接到出口节点本身的任何端口。

### 使用自动批准者自动批准出口节点

出口节点的初始设置通常需要在控制服务器上手动批准，然后 tailnet 中的节点才能使用它。Headscale 支持 ACL 的 `autoApprovers` 部分来自动批准新的出口节点，一旦它加入 tailnet。

下面的 ACL 片段定义了由用户 `alice` 拥有的标签 `tag:exit`。此标签用于 `autoApprovers` 部分的 `exitNode`。通告标签 `tag:exit` 的新出口节点将自动被批准：

```json title="Exit nodes tagged with tag:exit are automatically approved"
{
  "tagOwners": {
    "tag:exit": ["alice@"]
  },
  "autoApprovers": {
    "exitNode": ["tag:exit"]
  },
  "acls": [
    // more rules
  ]
}
```

在加入 tailnet 时，将节点通告为出口节点，同时也通告标签 `tag:exit`：

```console
$ sudo tailscale up --login-server <YOUR_HEADSCALE_URL> --advertise-tags tag:exit --advertise-exit-node
```

请参见[官方 Tailscale 文档](https://tailscale.com/kb/1337/acl-syntax#autoapprovers)以获取有关自动批准者的更多信息。

## 高可用性

Headscale 对高可用性路由的支持有限。用户可以使用多个具有重叠路由的子网路由器或多个出口节点来实现高可用性。如果一个路由器节点离线，另一个节点可以继续为客户端提供相同的路由。详情请参见官方的 [Tailscale 高可用性文档](https://tailscale.com/kb/1115/high-availability#subnet-router-high-availability)。

!!! bug

    在某些情况下，Headscale 可能需要长达 16 分钟才能检测到节点离线。如果将此类节点用作子网路由器或出口节点，故障转移节点可能无法及时被选中，从而导致客户端服务中断。更多信息请参见 [issue 2129](https://github.com/juanfont/headscale/issues/2129)。

## 故障排除

### 启用 IP 转发

子网路由器或出口节点代表其他节点路由流量，因此需要启用 IP 转发。请查看官方的 [Tailscale 文档](https://tailscale.com/kb/1019/subnets/?tab=linux#enable-ip-forwarding) 了解如何启用 IP 转发。