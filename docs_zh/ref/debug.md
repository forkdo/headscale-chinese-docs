# 调试和故障排除

Headscale 和 Tailscale 提供了调试和内省功能，当事情没有按预期工作时，这些功能会很有帮助。本页面解释了一些调试技术，以帮助定位问题。

请同时查看 [Tailscale 的故障排除指南](https://tailscale.com/docs/reference/troubleshooting)。它提供了许多提示和建议来排除常见问题。

## Tailscale

Tailscale 客户端本身提供了许多命令来检查其状态以及网络状态：

- [检查本地网络状况](https://tailscale.com/docs/reference/tailscale-cli#netcheck): `tailscale netcheck`
- [获取客户端状态](https://tailscale.com/docs/reference/tailscale-cli#status): `tailscale status --json`
- [获取 DNS 状态](https://tailscale.com/docs/reference/tailscale-cli#dns): `tailscale dns status --all`
- 客户端日志: `tailscale debug daemon-logs`
- 客户端网络图: `tailscale debug netmap`
- 测试 DERP 连接: `tailscale debug derp headscale`
- 以及更多，请参见: `tailscale debug --help`

许多命令在尝试理解 Headscale 和 Tailscale SaaS 之间的差异时很有帮助。

## Headscale

### 应用程序日志

日志级别 `debug` 和 `trace` 可以用于从 Headscale 获取更多信息。

```yaml hl_lines="3"
log:
  # 有效的日志级别: panic, fatal, error, warn, info, debug, trace
  level: debug
```

### 数据库日志

数据库调试模式记录所有数据库查询。启用它可以查看 Headscale 如何与其数据库交互。这也需要将应用程序日志级别设置为 `debug` 或 `trace`。

```yaml hl_lines="3 7"
database:
  # 启用调试模式。此设置要求 log.level 设置为 "debug" 或 "trace"。
  debug: false

log:
  # 有效的日志级别: panic, fatal, error, warn, info, debug, trace
  level: debug
```

### 指标和调试端点

Headscale 提供了一个指标和调试端点。它允许检查不同方面，例如：

- Go 运行时信息、内存使用情况和统计信息
- 已连接节点和待处理注册
- 活跃的策略、过滤器和 SSH 策略
- 当前 DERPMap
- Prometheus 指标

!!! warning "保持指标和调试端点私有"

    监听地址和端口可以通过 [配置文件](configuration.md) 中的 `metrics_listen_addr` 变量进行配置。默认情况下它监听在 localhost，端口 9090。

    将指标和调试端点保持在内部网络中私有，不要将其暴露到互联网。

    可以通过在 [配置文件](configuration.md) 中设置 `metrics_listen_addr: null` 来完全禁用指标和调试接口。

通过 <http://localhost:9090/metrics> 查询指标，并通过 <http://localhost:9090/debug/> 获取可用调试信息的概览。可以从 localhost 外部查询指标，但调试接口受到额外保护，即使监听在所有接口上。

=== "直接访问"

    在安装了 Headscale 的服务器上直接访问调试接口。

    ```console
    curl http://localhost:9090/debug/
    ```

=== "SSH 端口转发"

    使用 SSH 端口转发将 Headscale 的指标和调试端口转发到您的设备。

    ```console
    ssh <HEADSCALE_SERVER> -L 9090:localhost:9090
    ```

    在您的设备上通过在网页浏览器中打开 <http://localhost:9090/debug/> 来访问调试接口。

=== "通过调试密钥"

    调试接口的访问控制支持使用调试密钥。如果通过环境变量 `TS_DEBUG_KEY_PATH` 设置了调试密钥的路径，并且在每个请求中将调试密钥作为 `debugkey` 参数的值发送，则接受流量。

    ```console
    openssl rand -hex 32 | tee debugkey.txt
    export TS_DEBUG_KEY_PATH=debugkey.txt
    headscale serve
    ```

    在您的设备上通过在网页浏览器中打开 `http://<IP_OF_HEADSCALE>:9090/debug/?debugkey=<DEBUG_KEY>` 来访问调试接口。每个请求都必须发送 `debugkey` 参数。

=== "通过调试 IP 地址"

    调试端点期望来自 localhost 的流量。可以通过在启动 Headscale 之前设置 `TS_ALLOW_DEBUG_IP` 环境变量来配置不同的调试 IP 地址。当 HTTP 头 `X-Forwarded-For` 存在时，调试 IP 地址将被忽略。

    ```console
    export TS_ALLOW_DEBUG_IP=192.168.0.10       # 您的设备的 IP 地址
    headscale serve
    ```

    在您的设备上通过在网页浏览器中打开 `http://<IP_OF_HEADSCALE>:9090/debug/` 来访问调试接口。
