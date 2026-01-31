# API

Headscale 提供了一个 [HTTP REST API](#rest-api) 和一个 [gRPC 接口](#grpc)，可用于集成 [web 界面](integration/web-ui.md)、[远程控制 Headscale](#setup-remote-control) 或为自定义集成和工具提供基础。

在使用这两个接口之前都需要一个有效的 API 密钥。要创建一个 API 密钥，请登录到您的 Headscale 服务器并生成一个默认有效期为 90 天的密钥：

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
- 文档：`/swagger`，例如 `https://headscale.example.com/swagger`
- Headscale 版本：`/version`，例如 `https://headscale.example.com/version`
- 通过在 HTTP 请求中发送 `Authorization: Bearer <API_KEY>` 头部来使用 HTTP Bearer 认证进行认证。

首先 [创建一个 API 密钥](#api)，然后使用下面的示例进行测试。请在 `/swagger` 阅读 Headscale 服务器提供的 API 文档以获取详细信息。

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
        -d user=<USER> -d key=<REGISTRATION_KEY> \
        https://headscale.example.com/api/v1/node/register
    ```

## gRPC

gRPC 接口可用于通过 `headscale` 二进制文件从远程机器控制一个 Headscale 实例。

### 前置条件

- 一台运行 `headscale` 的工作站（支持任何平台，例如 Linux）。
- 一台启用了 gRPC 的 Headscale 服务器。
- 允许连接到 gRPC 端口（默认：`50443`）。
- 远程访问需要通过 TLS 进行加密连接。
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
            address: <HEADSCALE_ADDRESS>:<PORT>
            api_key: <API_KEY>
        ```

    === "环境变量"

        ```shell
        export HEADSCALE_CLI_ADDRESS="<HEADSCALE_ADDRESS>:<PORT>"
        export HEADSCALE_CLI_API_KEY="<API_KEY>"
        ```

    这告诉 `headscale` 二进制文件连接到 `<HEADSCALE_ADDRESS>:<PORT>` 上的远程实例，而不是连接到本地实例。

1. 通过列出所有节点来测试连接：

    ```shell
    headscale nodes list
    ```

    您现在应该能够在工作站上看到节点列表，并且现在可以从工作站控制 Headscale 服务器。

### 通过代理

可以在反向代理（如 Nginx）后面运行 gRPC 远程端点，并让它在与 Headscale 相同的端口上运行。

虽然这不是一个支持的功能，但下面展示了一个在 NixOS 上如何设置的示例：
[NixOS 示例](https://github.com/kradalby/dotfiles/blob/4489cdbb19cddfbfae82cd70448a38fde5a76711/machines/headscale.oracldn/headscale.nix#L61-L91)。

### 故障排除

- 确保您的服务器和工作站上运行的 Headscale 版本相同。
- 确保允许连接到 gRPC 端口。
- 验证您的 TLS 证书是否有效且受信任。
- 如果您没有受信任的证书（例如来自 Let's Encrypt），请：
    - 将您的自签名证书添加到操作系统的信任存储中，或者
    - 通过在配置文件中设置 `cli.insecure: true` 或通过环境变量设置 `HEADSCALE_CLI_INSECURE=1` 来禁用证书验证。我们不推荐禁用证书验证。