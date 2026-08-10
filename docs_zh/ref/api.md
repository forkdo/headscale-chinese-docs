# API

Headscale 提供了一个 [HTTP REST API](#rest-api)，可用于集成 [web 界面](integration/web-ui.md)、[远程控制 Headscale](#remote-control) 或为自定义集成和工具提供基础。

使用该 API 之前需要一个有效的 API 密钥。要创建一个 API 密钥，请登录到您的 Headscale 服务器并生成一个默认有效期为 90 天的密钥：

```shell
headscale apikeys create
```

复制命令的输出并保存以备后用。请注意，您无法再次检索 API 密钥。如果 API 密钥丢失了，请过期旧密钥并创建一个新的密钥。

要列出当前与服务器关联的 API 密钥：

```shell
headscale apikeys list
```

要过期一个 API 密钥：

```shell
headscale apikeys expire --prefix <PREFIX>
```

## REST API

- API 端点：`/api/v1`，例如 `https://headscale.example.com/api/v1`
- 文档：`/api/v1/docs`，例如 `https://headscale.example.com/api/v1/docs`
- Headscale 版本：`/version`，例如 `https://headscale.example.com/version`
- 通过在 HTTP 请求中发送 [API 密钥](#api) 和 HTTP `Authorization: Bearer <API_KEY>` 头部来使用 HTTP Bearer 认证进行认证。

首先 [创建一个 API 密钥](#api)，然后使用下面的示例进行测试。请在 `/api/v1/docs` 阅读 Headscale 服务器提供的 API 文档以获取详细信息。

=== "获取所有用户的详细信息"

    ```console
    curl -H "Authorization: Bearer <API_KEY>" \
        https://headscale.example.com/api/v1/user
    ```

=== "获取用户 'bob' 的详细信息"

    ```console
    curl -H "Authorization: Bearer <API_KEY>" \
        https://headscale.example.com/api/v1/user?name=bob
    ```

=== "注册一个节点"

    ```console
    curl -H "Authorization: Bearer <API_KEY>" \
        --json '{"user": "<USER>", "authId": "<AUTH_ID>"}' \
        https://headscale.example.com/api/v1/auth/register
    ```

## Remote control

`headscale` 二进制文件可以通过 HTTP API 从远程机器控制一个 Headscale 实例。

### 前置条件

- 一台运行 `headscale` 的工作站（支持任何平台，例如 Linux）。
- 可通过 HTTP(S) 访问的 Headscale 服务器。
- 一个 [API 密钥](#api) 用于向 Headscale 服务器进行认证。

### 设置远程控制

1. 从 GitHub 发布页面下载 [`headscale` 二进制文件](https://github.com/juanfont/headscale/releases)。确保使用与服务器相同版本的二进制文件。

1. 将二进制文件放到您的 `PATH` 中的某个位置，例如 `/usr/local/bin/headscale`

1. 使 `headscale` 可执行：`chmod +x /usr/local/bin/headscale`

1. 在 Headscale 服务器上 [创建一个 API 密钥](#api)。

1. 通过最小化的 YAML 配置文件或环境变量为远程 Headscale 服务器提供连接参数：

    === "最小化的 YAML 配置文件"

        ```yaml title="config.yaml"
        cli:
            address: <HEADSCALE_URL>
            api_key: <API_KEY>
        ```

    === "环境变量"

        ```shell
        export HEADSCALE_CLI_ADDRESS="<HEADSCALE_URL>"
        export HEADSCALE_CLI_API_KEY="<API_KEY>"
        ```

    这告诉 `headscale` 二进制文件连接到 `<HEADSCALE_URL>`（例如
    `https://headscale.example.com`）上的远程实例，而不是连接到本地实例。不带协议的裸主机名将被视为 `https`。

1. 通过列出所有节点来测试连接：

    ```shell
    headscale nodes list
    ```

    您现在应该能够在工作站上看到节点列表，并且现在可以从工作站控制 Headscale 服务器。

### 通过代理

远程 CLI 使用的是与其他所有功能相同的 HTTP API，因此它可以无缝通过 Headscale 前端已有的反向代理工作，无需额外配置。

### 故障排除

- 确保您的服务器和工作站上运行的 Headscale 版本相同。
- 验证您的 TLS 证书是否有效且受信任。
- 如果您没有受信任的证书（例如来自 Let's Encrypt），请：
    - 将您的自签名证书添加到操作系统的信任存储中，或者
    - 通过在配置文件中设置 `cli.insecure: true` 或通过环境变量设置 `HEADSCALE_CLI_INSECURE=1` 来禁用证书验证。我们不推荐禁用证书验证。
