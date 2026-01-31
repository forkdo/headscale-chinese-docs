# 通过 TLS 运行服务（可选）

## 使用自有证书

可以配置 Headscale 通过 TLS 暴露其 Web 服务。如需手动配置证书和密钥文件，请设置 `tls_cert_path` 和 `tls_key_path` 配置参数。如果路径是相对路径，则会被解释为相对于配置文件所在目录的路径。

```yaml title="config.yaml"
tls_cert_path: ""
tls_key_path: ""
```

证书应包含完整的证书链，否则某些客户端（如 Tailscale Android 客户端）将拒绝连接。

## Let's Encrypt / ACME

要通过 [Let's Encrypt](https://letsencrypt.org/) 自动获取证书，请将 `tls_letsencrypt_hostname` 设置为目标证书的主机名。该名称必须解析为 Headscale 可访问的 IP 地址（即必须与 `server_url` 配置参数对应）。证书和 Let's Encrypt 账户凭据将存储在 `tls_letsencrypt_cache_dir` 配置的目录中。如果路径是相对路径，则会被解释为相对于配置文件所在目录的路径。

```yaml title="config.yaml"
tls_letsencrypt_hostname: ""
tls_letsencrypt_listen: ":http"
tls_letsencrypt_cache_dir: ".cache"
tls_letsencrypt_challenge_type: HTTP-01
```

### 验证类型

Headscale 仅支持 `tls_letsencrypt_challenge_type` 的两个值：`HTTP-01`（默认）和 `TLS-ALPN-01`。

#### HTTP-01

对于 `HTTP-01`，除了 `listen_addr` 中配置的端口外，Headscale 还必须在 80 端口上可访问，以便 Let's Encrypt 进行自动验证。默认情况下，Headscale 会在所有本地 IP 的 80 端口上监听，以进行 Let's Encrypt 自动验证。

如果需要更改 Headscale 用于 Let's Encrypt 验证过程的 IP 和/或端口，请将 `tls_letsencrypt_listen` 设置为适当的值。这在以非 root 用户身份运行 Headscale（或无法运行 `setcap`）时非常有用。但请注意，Let's Encrypt 在验证回调时**只会**连接到 80 端口，因此如果更改了 `tls_letsencrypt_listen`，还需要配置其他内容（例如防火墙规则）将流量从 80 端口转发到 `tls_letsencrypt_listen` 中指定的 IP:端口组合。

#### TLS-ALPN-01

对于 `TLS-ALPN-01`，Headscale 会监听 `listen_addr` 中定义的 IP:端口组合。Let's Encrypt 在验证回调时**只会**连接到 443 端口，因此如果 `listen_addr` 未设置为 443 端口，则需要其他配置（例如防火墙规则）将流量从 443 端口转发到 `listen_addr` 中指定的 IP:端口组合。

### 技术说明

Headscale 使用 [autocert](https://pkg.go.dev/golang.org/x/crypto/acme/autocert)（一个提供 [ACME 协议](https://en.wikipedia.org/wiki/Automatic_Certificate_Management_Environment) 验证的 Golang 库）来通过 [Let's Encrypt](https://letsencrypt.org/about/) 实现证书续订。证书将自动续订，预期行为如下：

- 由 Let's Encrypt 提供的证书有效期为签发之日起 3 个月。
- 仅当距离证书到期日不足 30 天时，Headscale 才会尝试续订。
- autocert 的续订尝试会在 30-60 分钟的随机间隔内触发。
- 续订被跳过或成功时不会生成日志输出。

#### 检查证书到期时间

如需验证证书续订是否成功完成，可以手动或通过外部监控软件进行检查。以下是两种手动检查的示例：

1. 在浏览器中打开 Headscale 服务器的 URL，手动检查收到的证书的到期日期。
2. 或者，使用 `openssl` 从命令行远程检查：

```console
$ openssl s_client -servername [hostname] -connect [hostname]:443 | openssl x509 -noout -dates
(...)
notBefore=Feb  8 09:48:26 2024 GMT
notAfter=May  8 09:48:25 2024 GMT
```

#### autocert 库的日志输出

由于这些日志行来自 autocert 库，因此并非严格由 Headscale 本身生成。

```plaintext
acme/autocert: missing server name
```

可能由未指定主机名的传入连接引起，例如直接针对服务器 IP 的 `curl` 请求，或意外的主机名。

```plaintext
acme/autocert: host "[foo]" not configured in HostWhitelist
```

与上述情况类似，这通常表示针对错误主机名的无效传入请求，常见情况就是 IP 地址本身。

autocert 的源代码可在此处找到：[https://cs.opensource.google/go/x/crypto/+/refs/tags/v0.19.0:acme/autocert/autocert.go](https://cs.opensource.google/go/x/crypto/+/refs/tags/v0.19.0:acme/autocert/autocert.go)