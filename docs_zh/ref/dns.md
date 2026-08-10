# DNS

Headscale 支持 [Tailscale 的大多数 DNS 功能](../about/features.md)。DNS 相关设置可在 [配置文件](configuration.md) 的 `dns` 部分中进行配置。

## 设置额外的 DNS 记录

Headscale 允许设置额外的 DNS 记录，这些记录可通过 [MagicDNS](https://tailscale.com/docs/features/magicdns) 提供。额外的 DNS 记录可通过配置文件中的静态条目或 Headscale 持续监控更改的 JSON 文件进行配置：

- 对于在 Headscale 运行期间静态不变的条目，使用 [配置文件](configuration.md) 中的 `dns.extra_records` 选项。这些条目在 Headscale 启动时进行处理，配置更改需要重启 Headscale。
- 对于在 Headscale 运行期间可能被添加、更新或删除的动态 DNS 记录，或由脚本生成的 DNS 记录，[配置文件](configuration.md) 中的 `dns.extra_records_path` 选项很有用。将其设置为包含 DNS 记录的 JSON 文件的绝对路径，Headscale 会在检测到更改时处理此文件。

一个典型的使用场景是通过反向代理（如 NGINX）在同一主机上托管多个应用，例如一个 Prometheus 监控堆栈。这允许您以 "http://grafana.myvpn.example.com" 的形式访问服务，而不是使用主机名和端口的组合 "http://hostname-in-magic-dns.myvpn.example.com:3000"。

!!! warning "限制"

    目前，[Tailscale 仅处理 A 和 AAAA 记录](https://github.com/tailscale/tailscale/blob/v1.86.5/ipn/ipnlocal/node_backend.go#L662)。

1. 使用以下任一可用的配置选项配置额外的 DNS 记录：

    === "静态条目，通过 `dns.extra_records`"

        ```yaml title="config.yaml"
        dns:
          ...
          extra_records:
            - name: "grafana.myvpn.example.com"
              type: "A"
              value: "100.64.0.3"

            - name: "prometheus.myvpn.example.com"
              type: "A"
              value: "100.64.0.3"
          ...
        ```

        重启您的 headscale 实例。

    === "动态条目，通过 `dns.extra_records_path`"

        ```json title="extra-records.json"
        [
          {
            "name": "grafana.myvpn.example.com",
            "type": "A",
            "value": "100.64.0.3"
          },
          {
            "name": "prometheus.myvpn.example.com",
            "type": "A",
            "value": "100.64.0.3"
          }
        ]
        ```

        Headscale 会自动检测上述 JSON 文件的更改。

        !!! tip "提示"

            - [配置文件](configuration.md) 中的 `dns.extra_records_path` 选项需要引用包含额外 DNS 记录的 JSON 文件。
            - 如果您使用脚本生成 JSON 文件，请确保对键进行“排序”并生成稳定的输出。Headscale 使用校验和来检测文件更改，稳定的输出可以避免不必要的处理。

1. 使用您选择的 DNS 查询工具验证 DNS 记录是否正确设置：

    === "使用 dig 查询"

        ```console
        dig +short grafana.myvpn.example.com
        100.64.0.3
        ```

    === "使用 drill 查询"

        ```console
        drill -Q grafana.myvpn.example.com
        100.64.0.3
        ```

1. 可选：设置反向代理

    此示例的动机是能够在同一主机上访问内部监控服务而无需指定端口，以下为 NGINX 配置片段：

    ```nginx title="nginx.conf"
    server {
        listen 80;
        listen [::]:80;

        server_name grafana.myvpn.example.com;

        location / {
            proxy_pass http://localhost:3000;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

    }
    ```
