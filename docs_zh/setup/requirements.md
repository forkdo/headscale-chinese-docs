# 要求

只要满足以下要求，Headscale 就可以正常运行：

- 一台具有公网 IP 地址的服务器用于运行 headscale。建议采用双栈配置，即同时拥有公网 IPv4 和公网 IPv6 地址。
- Headscale 通过 443 端口以 HTTPS 方式提供服务[^1]，并且[可能使用其他端口](#使用的端口)。
- 一个相当现代的 Linux 或基于 BSD 的操作系统。
- 一个专用的本地用户账户用于运行 headscale。
- 具备一定的命令行知识，用于配置和操作 headscale。

## 使用的端口

使用的端口因预期场景和启用的功能而异。虽然列出的某些端口可以通过[配置文件](../ref/configuration.md)进行修改，但我们建议保持默认值。

- tcp/80
    - 是否公开暴露：是
    - HTTP，Let's Encrypt 通过 HTTP-01 挑战验证所有权时使用。
    - 仅在使用内置 Let's Encrypt 客户端和 HTTP-01 挑战时才需要。详情请参见 [TLS](../ref/tls.md)。
- tcp/443
    - 是否公开暴露：是
    - HTTPS，用于使 Tailscale 客户端能够访问 Headscale[^1]
    - 如果启用了[内置 DERP 服务器](../ref/derp.md)，则需要此端口
- udp/3478
    - 是否公开暴露：是
    - STUN，如果启用了[内置 DERP 服务器](../ref/derp.md)，则需要此端口
- tcp/50443
    - 是否公开暴露：是
    - 仅在使用 gRPC 接口[远程控制 Headscale](../ref/api.md#grpc)时才需要。
- tcp/9090
    - 是否公开暴露：否
    - [指标和调试端点](../ref/debug.md#metrics-and-debug-endpoint)

## 假设

Headscale 文档和提供的示例基于以下几个假设编写：

- Headscale 作为系统服务运行，使用专用的本地用户 `headscale`。
- [配置](../ref/configuration.md)从 `/etc/headscale/config.yaml` 加载。
- 使用 SQLite 作为数据库。
- Headscale 的数据目录（用于存储私钥、ACL、SQLite 数据库等）位于 `/var/lib/headscale`。
- 需要用户替换的 URL 和值要么标记为 `<VALUE_TO_CHANGE>`，要么使用占位符值，例如 `headscale.example.com`。

请根据您的本地环境进行相应调整。

[^1]:
    在某些情况下，Tailscale 客户端会假定使用 443 端口的 HTTPS。虽然可以通过 HTTP 或在 443 以外的端口上通过 HTTPS 提供 headscale 服务，但对于生产环境，强烈建议坚持使用 443 端口的 HTTPS。更多信息请参见 [issue 2164](https://github.com/juanfont/headscale/issues/2164)。