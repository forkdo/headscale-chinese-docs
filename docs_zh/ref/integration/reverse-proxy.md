# 在反向代理后运行 headscale

!!! warning "社区文档"

    此页面并非由 headscale 作者积极维护，而是由社区成员编写。它**未经** headscale 开发人员验证。

    **可能已过时，且可能遗漏必要步骤**。

当在同一服务器上运行多个应用程序，并希望重复使用相同的外部 IP 和端口（通常是用于 HTTPS 的 tcp/443）时，在反向代理后运行 headscale 非常有用。

### WebSockets

反向代理必须配置为支持 WebSockets，以便与 Tailscale 客户端通信。

使用 Headscale [嵌入式 DERP 服务器](../derp.md)时也需要 WebSockets 支持。在这种情况下，您还需要暴露用于 STUN 的 UDP 端口（默认为 udp/3478）。请查看我们的 [config-example.yaml](https://github.com/juanfont/headscale/blob/main/config-example.yaml)。

### Cloudflare

不支持在 cloudflare 代理或 cloudflare 隧道后运行 headscale，因为这无法正常工作，Cloudflare 不支持 Tailscale 协议所需的 WebSocket POST。请参阅[此问题](https://github.com/juanfont/headscale/issues/1468)

### TLS

可以配置 headscale 不使用 TLS，而由反向代理处理。将以下配置值添加到您的 headscale 配置文件中。

```yaml title="config.yaml"
server_url: https://<YOUR_SERVER_NAME> # 这应该是 headscale 将提供服务的 FQDN
listen_addr: 0.0.0.0:8080
metrics_listen_addr: 0.0.0.0:9090
tls_cert_path: ""
tls_key_path: ""
```

## nginx

以下示例配置可用于您的 nginx 设置，根据需要替换值。`<IP:PORT>` 应该是 headscale 运行的 IP 地址和端口。在大多数情况下，这将是 `http://localhost:8080`。

```nginx title="nginx.conf"
map $http_upgrade $connection_upgrade {
    default      upgrade;
    ''           close;
}

server {
    listen 80;
	listen [::]:80;

	listen 443      ssl http2;
	listen [::]:443 ssl http2;

    server_name <YOUR_SERVER_NAME>;

    ssl_certificate <PATH_TO_CERT>;
    ssl_certificate_key <PATH_CERT_KEY>;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://<IP:PORT>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $server_name;
        proxy_redirect http:// https://;
        proxy_buffering off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        add_header Strict-Transport-Security "max-age=15552000; includeSubDomains" always;
    }
}
```

## istio/envoy

如果您使用 [Istio](https://istio.io/) ingressgateway 或 [Envoy](https://www.envoyproxy.io/) 作为反向代理，这里有一些提示。如果未设置，您可能会在代理中看到如下调试日志：

```log
Sending local reply with details upgrade_failed
```

### Envoy

您需要添加一个名为 `tailscale-control-protocol` 的新 upgrade_type。[详见此处](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-upgradeconfig)

### Istio

与 envoy 相同，我们可以使用 `EnvoyFilter` 来添加 upgrade_type。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: headscale-behind-istio-ingress
  namespace: istio-system
spec:
  configPatches:
    - applyTo: NETWORK_FILTER
      match:
        listener:
          filterChain:
            filter:
              name: envoy.filters.network.http_connection_manager
      patch:
        operation: MERGE
        value:
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
            upgrade_configs:
              - upgrade_type: tailscale-control-protocol
```

## Caddy

以下 Caddyfile 是将 Caddy 用作 headscale 反向代理所需的全部内容，结合上述 `config.yaml` 规范来禁用 headscale 内置的 TLS。根据需要替换值 - `<YOUR_SERVER_NAME>` 应该是 headscale 将提供服务的 FQDN，`<IP:PORT>` 应该是 headscale 运行的 IP 地址和端口。在大多数情况下，这将是 `localhost:8080`。

```none title="Caddyfile"
<YOUR_SERVER_NAME> {
    reverse_proxy <IP:PORT>
}
```

Caddy v2 将[自动](https://caddyserver.com/docs/automatic-https)为您的域/子域提供证书，强制 HTTPS，并代理 websockets - 无需进一步配置。

对于使用 Docker 容器管理 Caddy、headscale 和 Headscale-UI 的更复杂配置，[Guru Computing 的指南](https://blog.gurucomputing.com.au/smart-vpns-with-headscale/)是一个很好的参考。

## Apache

以下最小 Apache 配置将代理流量到 `<IP:PORT>` 上的 headscale 实例。请注意，`upgrade=any` 是 `ProxyPass` 的必需参数，以便正确转发 `Upgrade` 标头值不等于 `WebSocket`（即 Tailscale 控制协议）的 WebSockets 流量。有关此内容的更多信息，请参阅 [Apache 文档](https://httpd.apache.org/docs/2.4/mod/mod_proxy_wstunnel.html)。

```apache title="apache.conf"
<VirtualHost *:443>
	ServerName <YOUR_SERVER_NAME>

	ProxyPreserveHost On
	ProxyPass / http://<IP:PORT>/ upgrade=any

	SSLEngine On
	SSLCertificateFile <PATH_TO_CERT>
	SSLCertificateKeyFile <PATH_CERT_KEY>
</VirtualHost>
```