# 入门指南

本页面帮助您开始使用 headscale，并提供一些 headscale 命令行工具 `headscale` 的使用示例。

!!! note "先决条件"

    * Headscale 已安装并以系统服务运行。请阅读[设置部分](../setup/requirements.md)了解安装说明。
    * 配置文件已存在并根据您的环境进行了调整，详情请参见[配置](../ref/configuration.md)。
    * Headscale 可从互联网访问。请通过访问健康检查端点来验证：https://headscale.example.com/health
    * 已安装 Tailscale 客户端，更多信息请参见[客户端和操作系统支持](../about/clients.md)。

## 获取帮助

`headscale` 命令行工具提供了内置帮助。要显示可用命令及其参数和选项，请运行：

=== "原生方式"

    ```shell
    # 显示帮助
    headscale help

    # 显示特定命令的帮助
    headscale <COMMAND> --help
    ```

=== "容器方式"

    ```shell
    # 显示帮助
    docker exec -it headscale \
      headscale help

    # 显示特定命令的帮助
    docker exec -it headscale \
      headscale <COMMAND> --help
    ```

!!! note "从其他本地用户管理 headscale"

    默认情况下，只有 `headscale` 用户或 `root` 用户才具有访问用于与服务通信的 unix 套接字 (`/var/run/headscale/headscale.sock`) 的必要权限。为了能够与 headscale 服务通信，您必须确保运行命令的用户可以访问该 unix 套接字。通常，您可以通过以下任一方法实现：

      * 使用 `sudo`
      * 以 `headscale` 用户身份运行命令
      * 将您的用户添加到 `headscale` 组

    要验证，您可以使用首选方法运行以下命令：

    ```shell
    headscale users list
    ```

## 管理 headscale 用户

在 headscale 中，节点（也称为机器或设备）[通常会被分配给一个 headscale 用户](../ref/registration.md#identity-model)。这样的 headscale 用户可以拥有多个分配给他们的节点，并可通过 `headscale users` 命令进行管理。调用内置帮助以获取更多信息：`headscale users --help`。

### 创建 headscale 用户

=== "原生方式"

    ```shell
    headscale users create <USER>
    ```

=== "容器方式"

    ```shell
    docker exec -it headscale \
      headscale users create <USER>
    ```

### 列出已有的 headscale 用户

=== "原生方式"

    ```shell
    headscale users list
    ```

=== "容器方式"

    ```shell
    docker exec -it headscale \
      headscale users list
    ```

## 注册节点

要使用 headscale 作为 Tailscale 的协调服务器，必须首先[注册节点](../ref/registration.md)。以下示例适用于 Linux/BSD 操作系统上的 Tailscale 客户端。或者，请按照连接 [Android](connect/android.md)、[Apple](connect/apple.md) 或 [Windows](connect/windows.md) 设备的说明进行操作。请阅读[注册方法](../ref/registration.md)以了解可用的注册方法概览。

### [Web 认证](../ref/registration.md#web-authentication)

在客户端机器上，运行 `tailscale up` 命令并提供您的 headscale 实例的 FQDN 作为参数：

```shell
tailscale up --login-server <YOUR_HEADSCALE_URL>
```

通常，会打开一个包含进一步说明的浏览器窗口。此页面说明了如何在您的 headscale 服务器上完成注册，并显示了批准节点所需的注册密钥：

=== "原生方式"

    ```shell
    headscale nodes register --user <USER> --key <REGISTRATION_KEY>
    ```

=== "容器方式"

    ```shell
    docker exec -it headscale \
      headscale nodes register --user <USER> --key <REGISTRATION_KEY>
    ```

### [预认证密钥](../ref/registration.md#pre-authenticated-key)

也可以生成预认证密钥并以非交互方式注册节点。首先，在 headscale 实例上生成预认证密钥。默认情况下，该密钥有效期为一小时，且只能使用一次（有关其他选项，请参见 `headscale preauthkeys --help`）：

=== "原生方式"

    ```shell
    headscale preauthkeys create --user <USER_ID>
    ```

=== "容器方式"

    ```shell
    docker exec -it headscale \
      headscale preauthkeys create --user <USER_ID>
    ```

该命令成功时会返回预认证密钥，用于通过 `tailscale up` 命令将节点连接到 headscale 实例：

```shell
tailscale up --login-server <YOUR_HEADSCALE_URL> --authkey <YOUR_AUTH_KEY>
```