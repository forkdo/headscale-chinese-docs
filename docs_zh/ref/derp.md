# DERP

一个 [DERP (Designated Encrypted Relay for Packets) 服务器](https://tailscale.com/kb/1232/derp-servers) 主要用于在两个节点之间中继流量，当无法建立直接连接时。Headscale 提供了一个嵌入式 DERP 服务器，以确保节点之间的无缝连接。

## 配置

DERP 相关设置在 [配置文件](./configuration.md) 的 `derp` 部分中进行配置。以下部分仅使用了可用设置中的一小部分，查看 [示例配置](./configuration.md) 以了解所有可用的配置选项。

### 启用嵌入式 DERP

Headscale 自带一个嵌入式 DERP 服务器，允许您轻松运行自己的自托管 DERP 服务器。嵌入式 DERP 服务器默认禁用，需要手动启用。此外，为了提高连接稳定性，建议配置您的 Headscale 服务器的公网 IPv4 和公网 IPv6 地址：

```yaml title="config.yaml" hl_lines="3-5"
derp:
  server:
    enabled: true
    ipv4: 198.51.100.1
    ipv6: 2001:db8::1
```

请注意，[运行 DERP 服务器需要额外的端口](../setup/requirements.md#ports-in-use)。除了中继流量，它还使用 STUN (udp/3478) 来帮助客户端发现其公网 IP 地址并执行 NAT 穿透。[检查 DERP 服务器连接性](#check-derp-server-connectivity) 以确认一切正常工作。

### 移除 Tailscale 的 DERP 服务器

启用后，Headscale 的嵌入式 DERP 会被添加到 Tailscale Inc. 提供的免费 [DERP 服务器](https://tailscale.com/kb/1232/derp-servers) 列表中。若只想使用 Headscale 的嵌入式 DERP 服务器，请禁用默认 DERP 地图的加载：

```yaml title="config.yaml" hl_lines="6"
derp:
  server:
    enabled: true
    ipv4: 198.51.100.1
    ipv6: 2001:db8::1
  urls: []
```

!!! warning "单点故障"

    移除 Tailscale 的 DERP 服务器意味着现在只有一个 DERP 服务器可供客户端使用。这是一个单点故障，可能会影响连接性。

    在移除 Tailscale 的 DERP 服务器之前，请先使用 [检查 DERP 服务器连接性](#check-derp-server-connectivity) 确认您的嵌入式 DERP 服务器工作正常。

### 自定义 DERP 地图

提供给客户端的 DERP 地图可以通过 [专用 YAML 配置文件](https://github.com/juanfont/headscale/blob/main/derp-example.yaml) 进行自定义。这允许修改通过 URL 获取的先前加载的 DERP 地图，或向节点提供您自己的自定义 DERP 服务器。

=== "移除特定 DERP 区域"

    免费的 [DERP 服务器](https://tailscale.com/kb/1232/derp-servers) 通过区域 ID 组织为区域。您可以通过将特定区域的区域 ID 设为 `null` 来明确禁用该区域。以下示例 `derp.yaml` 禁用了纽约 DERP 区域（区域 ID 为 1）：

     ```yaml title="derp.yaml"
     regions:
       1: null
     ```

    使用以下配置向节点提供默认 DERP 地图（排除纽约）：

    ```yaml title="config.yaml" hl_lines="6 7"
    derp:
      server:
        enabled: false
      urls:
        - https://controlplane.tailscale.com/derpmap/default
      paths:
        - /etc/headscale/derp.yaml
    ```

=== "提供自定义 DERP 服务器"

    以下示例 `derp.yaml` 引用了两个自定义区域（`custom-east`，ID 为 900 和 `custom-west`，ID 为 901），每个区域各有一个自定义 DERP 服务器。每个 DERP 服务器通过 HTTPS 在 tcp/443 上提供 DERP 中继服务，通过 HTTP 在 tcp/80 上支持 captive portal 检查，并通过 udp/3478 提供 STUN 服务。有关所有可用选项，请参阅
    [DERPMap](https://pkg.go.dev/tailscale.com/tailcfg#DERPMap)、
    [DERPRegion](https://pkg.go.dev/tailscale.com/tailcfg#DERPRegion) 和
    [DERPNode](https://pkg.go.dev/tailscale.com/tailcfg#DERPNode) 的定义。

    ```yaml title="derp.yaml"
    regions:
      900:
        regionid: 900
        regioncode: custom-east
        regionname: My region (east)
        nodes:
          - name: 900a
            regionid: 900
            hostname: derp900a.example.com
            ipv4: 198.51.100.1
            ipv6: 2001:db8::1
            canport80: true
      901:
        regionid: 901
        regioncode: custom-west
        regionname: My Region (west)
        nodes:
          - name: 901a
            regionid: 901
            hostname: derp901a.example.com
            ipv4: 198.51.100.2
            ipv6: 2001:db8::2
            canport80: true
    ```

    使用以下配置仅提供上述 `derp.yaml` 中的两个 DERP 服务器：

    ```yaml title="config.yaml" hl_lines="5 6"
    derp:
      server:
        enabled: false
      urls: []
      paths:
        - /etc/headscale/derp.yaml
    ```

无论是否使用自定义 DERP 地图，您都可以选择 [启用嵌入式 DERP 服务器并将其自动添加到自定义 DERP 地图中](#enable-embedded-derp)。

### 验证客户端

DERP 服务器的访问可以限制为只允许属于您的 Tailnet 的节点。对于未知客户端，中继访问将被拒绝。

=== "嵌入式 DERP"

    客户端验证默认启用。

    ```yaml title="config.yaml" hl_lines="3"
    derp:
      server:
        verify_clients: true
    ```

=== "第三方 DERP"

    Tailscale 的 `derper` 提供两个参数来配置客户端验证：

    - 使用 `derper` 的 `-verify-client-url` 参数，将其指向您的 Headscale 服务器的 `/verify` 端点（例如 `https://headscale.example.com/verify`）。当客户端连接到 DERP 服务器时，DERP 服务器会查询您的 Headscale 实例，询问是否允许或拒绝访问。如果 Headscale 知道该连接客户端，则允许访问，否则拒绝。
    - 参数 `-verify-client-url-fail-open` 控制当 DERP 服务器无法连接到 Headscale 实例时应如何处理。默认情况下，如果无法连接到 Headscale，将允许访问。

## 检查 DERP 服务器连接性

任何 Tailscale 客户端都可以用来检查 DERP 地图并检测 DERP 服务器的连接问题。

- 显示 DERP 地图：`tailscale debug derp-map`
- 检查与嵌入式 DERP[^1] 的连接性：`tailscale debug derp headscale`

更多 DERP 相关指标和信息可通过 [指标和调试端点](./debug.md#metrics-and-debug-endpoint) 获取。

[^1]:
    这假设使用了 [配置文件](./configuration.md) 中的默认区域代码。

## 限制

- 内置 DERP 服务器无法用于 Tailscale 的 captive portal 检查，因为它不支持通过 HTTP 在 tcp/80 端口上的 `/generate_204` 端点。
- 没有速度或吞吐量优化，主要目的是协助节点连接。