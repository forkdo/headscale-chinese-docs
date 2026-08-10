# 在反向代理后运行 Headscale

!!! warning "社区文档"

    此页面并非由 Headscale 作者积极维护，而是由社区成员编写。它**未经** Headscale 开发人员验证。

    **可能已过时，且可能遗漏必要步骤**。

在反向代理后运行 Headscale 非常有用，当您在同一服务器上运行多个应用程序，并希望
重复使用相同的外部 IP 和端口（通常是用于 HTTPS 的 tcp/443）时。

请参阅[限制](#limitations)了解已知问题和限制。

## 配置

配置取决于您打算使用的 Headscale 功能集合。请查看
[要求](../../setup/requirements.md)，尤其是[使用的端口](../../setup/requirements.md#ports-in-use)
部分，以了解 Tailscale 客户端的预期。

本文档中的配置示例是基础性的，仅涵盖 HTTP 和 HTTPS 流量。其他功能（例如 Headscale 的
[嵌入式 DERP 服务器](../derp.md)的 STUN）应直接暴露或仅对 localhost 可用。

### WebSocket

Tailscale 客户端使用自定义协议（Tailscale 控制协议）与控制服务器（如 Headscale）通信。反向代理**必须**
配置为支持 WebSocket，以便与 Tailscale 客户端通信，并且需要处理 Tailscale 控制协议的两个特性：

- 使用 POST 方法来升级 WebSocket 连接。
- `Upgrade` 标头的值为 `tailscale-control-protocol`。

### TLS

可以配置 Headscale 不使用 TLS，而由反向代理处理。将以下配置值添加到您的 Headscale
[配置文件](../configuration.md)：

```yaml title="config.yaml" hl_lines="1"
server_url: https://<SERVER_NAME>
tls_cert_path: ""
tls_key_path: ""
```

Headscale 在启动期间会记录 `WRN listening without TLS but ServerURL does not start with http://`。这是预期行为，
表明反向代理负责终止 TLS。

### 受信任的代理

除非请求的 TCP 对端匹配 `trusted_proxies` 配置选项，否则 Headscale 会忽略 `True-Client-IP`、`X-Real-IP` 和
`X-Forwarded-For` 标头。将其设置为您的反向代理所连接的 CIDR，以便真实客户端 IP 出现在访问日志中。

```yaml title="config.yaml"
trusted_proxies:
  - 127.0.0.1/32
  - ::1/128
```

反向代理负责将入站请求上任何客户端提供的 `True-Client-IP`、`X-Real-IP`、`X-Forwarded-For` 标头替换为经过
清理的值。Headscale 按以下顺序从标头中获取第一个有效的 IP 地址：

- `True-Client-IP`
- `X-Real-IP`
- `X-Forwarded-For`

## 限制

- 反向代理增加了另一层复杂性，需要能够正确处理 [Tailscale 控制协议](#websocket)。在提交 issue 之前，请务必
  先在没有反向代理的情况下测试您的设置。
- STUN（与[嵌入式 DERP 服务器](../derp.md)一起使用）需要将 udp/3478 公开提供服务。

## 反向代理特定配置

!!! warning "第三方软件和服务"

    本文档的这一部分针对第三方软件和服务。我们建议用户阅读第三方文档以获得安全的配置。

以下 Headscale 配置可作为下面各种反向代理示例的基础。假设[如下](../../setup/requirements.md)：

- 为 Tailscale 客户端提供的服务通过 HTTPS 在 443 端口上提供。
- 反向代理将 HTTP 重定向到 HTTPS，并终止 TLS。
- Headscale 和反向代理都运行在同一主机上。
- [指标](../debug.md#metrics-and-debug-endpoint)不被代理，仅通过 localhost 可用。

```yaml title="config.yaml" hl_lines="1"
server_url: https://<SERVER_NAME>
listen_addr: 127.0.0.1:8080
metrics_listen_addr: 127.0.0.1:9090
trusted_proxies:
  - 127.0.0.1/32
  - ::1/128
tls_cert_path: ""
tls_key_path: ""
```

### Apache

以下基础 Apache 配置可与[上述](#reverse-proxy-specific-configuration)所示的 Headscale 配置配合使用。请根据需要替换占位符并调整配置：

- `<SERVER_NAME>`：您实例的服务器名称，例如 `headscale.example.com`
- `<PATH_TO_TLS_CERT>`：您的 TLS 证书的绝对路径
- `<PATH_TO_TLS_KEY>`：您的 TLS 私钥的绝对路径

```apache title="apache.conf" hl_lines="2 7 11 14-15"
<VirtualHost *:80>
  ServerName <SERVER_NAME>

  # Tailscale 强制门户检测
  RedirectMatch 204 ^/generate_204$

  RedirectMatch permanent "^/(.*)$" "https://<SERVER_NAME>/$1"
</VirtualHost>

<VirtualHost *:443>
  ServerName <SERVER_NAME>

  SSLEngine On
  SSLCertificateFile <PATH_TO_TLS_CERT>
  SSLCertificateKeyFile <PATH_TO_TLS_KEY>

  RequestHeader set True-Client-IP "%{REMOTE_ADDR}s"
  RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"

  ProxyPreserveHost On
  ProxyPass / http://127.0.0.1:8080/ upgrade=any
</VirtualHost>
```

请注意，`upgrade=any` 是 `ProxyPass` 的必需参数，以便正确转发 `Upgrade` 标头值不等于 `WebSocket`（即 Tailscale 控制协议）的 WebSocket 流量。有关此内容的更多信息，请参阅 [Apache 文档](https://httpd.apache.org/docs/current/mod/mod_proxy.html#upgrade)。

### Caddy

以下基础 Caddyfile 可与[上述](#reverse-proxy-specific-configuration)所示的 Headscale 配置配合使用。请根据需要替换占位符并调整配置：

- `<SERVER_NAME>`：您实例的服务器名称，例如 `headscale.example.com`

```none title="Caddyfile" hl_lines="1 12"
http://<SERVER_NAME> {
	# Tailscale 强制门户检测
	handle /generate_204 {
		respond 204
	}

	handle * {
		redir https://{host}{uri}
	}
}

<SERVER_NAME> {
	reverse_proxy 127.0.0.1:8080 {
		header_up True-Client-IP {remote_host}
		header_up X-Real-IP {remote_host}
	}
}
```

Caddy 会[自动](https://caddyserver.com/docs/automatic-https)为您的域/子域配置证书，强制 HTTPS，并代理 WebSocket 连接。

### Cloudflare

不支持在 Cloudflare 代理或 Cloudflare 隧道后运行 Headscale，因为这无法正常工作，Cloudflare 不支持
[Tailscale 协议所需的 WebSocket POST](#websocket)。更多信息请参阅 [issue 1468](https://github.com/juanfont/headscale/issues/1468)。

### Envoy

您需要添加一个名为 `tailscale-control-protocol` 的新 upgrade_type。[详见此处](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-upgradeconfig)。

### Istio

与 [envoy](#envoy) 相同，我们可以使用 `EnvoyFilter` 添加一个名为 `tailscale-control-protocol` 的新 upgrade_type。

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

### Nginx

以下基础 Nginx 配置可与[上述](#reverse-proxy-specific-configuration)所示的 Headscale 配置配合使用。请根据需要替换占位符并调整配置：

- `<SERVER_NAME>`：您实例的服务器名称，例如 `headscale.example.com`
- `<PATH_TO_TLS_CERT>`：您的 TLS 证书的绝对路径
- `<PATH_TO_TLS_KEY>`：您的 TLS 私钥的绝对路径

```nginx title="nginx.conf" hl_lines="19 37 39-40"
# headscale
upstream headscale {
  zone upstreams 64K;
  server 127.0.0.1:8080 max_fails=1 fail_timeout=5s;
  keepalive 2;
}

# websocket
map $http_upgrade $connection_upgrade {
  default keep-alive;
  ''      close;
}

# http
server {
  listen 80;
  listen [::]:80;

  server_name <SERVER_NAME>;

  # Tailscale 强制门户检测
  location = /generate_204 {
    return 204;
  }

  location / {
    return 301 https://$server_name$request_uri;
  }
}

# https
server {
  listen 443 ssl;
  listen [::]:443 ssl;
  http2 on;

  server_name <SERVER_NAME>;

  ssl_certificate <PATH_TO_TLS_CERT>;
  ssl_certificate_key <PATH_TO_TLS_KEY>;

  location / {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_set_header True-Client-IP $remote_addr;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_buffering off;
    proxy_pass http://headscale;
  }
}
```
