# 官方版本

headscale 的官方版本提供了适用于各种平台的二进制文件以及适用于 Debian 和 Ubuntu 的 DEB 软件包。  
两者均可在 [GitHub 发布页面](https://github.com/juanfont/headscale/releases) 获取。

## 使用 Debian/Ubuntu 软件包（推荐）

建议在基于 Debian 的系统上使用我们的 DEB 软件包来安装 headscale，因为这些软件包会配置一个本地用户来运行 headscale，提供默认配置，并附带 systemd 服务文件。  
支持的发行版包括 Ubuntu 22.04 或更高版本、Debian 12 或更高版本。

1.  为您的平台下载[最新的 headscale 软件包](https://github.com/juanfont/headscale/releases/latest)（Ubuntu 和 Debian 使用 `.deb` 文件）。

    ```shell
    HEADSCALE_VERSION="" # 请参考上述 URL 获取最新版本，例如 "X.Y.Z"（注意：不要添加 "v" 前缀！）
    HEADSCALE_ARCH="" # 您的系统架构，例如 "amd64"
    wget --output-document=headscale.deb \
     "https://github.com/juanfont/headscale/releases/download/v${HEADSCALE_VERSION}/headscale_${HEADSCALE_VERSION}_linux_${HEADSCALE_ARCH}.deb"
    ```

1.  安装 headscale：

    ```shell
    sudo apt install ./headscale.deb
    ```

1.  [通过编辑配置文件来配置 headscale](../../ref/configuration.md)：

    ```shell
    sudo nano /etc/headscale/config.yaml
    ```

1.  启用并启动 headscale 服务：

    ```shell
    sudo systemctl enable --now headscale
    ```

1.  验证 headscale 是否按预期运行：

    ```shell
    sudo systemctl status headscale
    ```

请继续阅读[入门指南](../../usage/getting-started.md)以注册您的第一台机器。

## 使用独立二进制文件（高级）

!!! warning "高级"

    这种安装方法被视为高级操作，因为您需要自行处理本地用户和 systemd 服务。如果可能，请改用 [DEB 软件包](#using-packages-for-debianubuntu-recommended) 或[社区软件包](./community.md)。

本节介绍根据[要求和假设](../requirements.md#assumptions)安装 headscale 的方法。headscale 由专用的本地用户运行，服务本身由 systemd 管理。

1.  从 [GitHub 发布页面下载最新的 `headscale` 二进制文件](https://github.com/juanfont/headscale/releases)：

    ```shell
    sudo wget --output-document=/usr/bin/headscale \
    https://github.com/juanfont/headscale/releases/download/v<HEADSCALE VERSION>/headscale_<HEADSCALE VERSION>_linux_<ARCH>
    ```

1.  使 `headscale` 可执行：

    ```shell
    sudo chmod +x /usr/bin/headscale
    ```

1.  添加一个专用的本地用户来运行 headscale：

    ```shell
    sudo useradd \
     --create-home \
     --home-dir /var/lib/headscale/ \
     --system \
     --user-group \
     --shell /usr/sbin/nologin \
     headscale
    ```

1.  下载您所选版本的示例配置文件，并将其保存为：`/etc/headscale/config.yaml`。根据您的本地环境调整配置。详情请参阅[配置](../../ref/configuration.md)。

    ```shell
    sudo mkdir -p /etc/headscale
    sudo nano /etc/headscale/config.yaml
    ```

1.  将 [headscale 的 systemd 服务文件](https://github.com/juanfont/headscale/blob/main/packaging/systemd/headscale.service) 复制到 `/etc/systemd/system/headscale.service`，并根据您的本地设置进行调整。以下参数可能需要修改：`ExecStart`、`WorkingDirectory`、`ReadWritePaths`。

1.  在 `/etc/headscale/config.yaml` 中，将默认的 `headscale` Unix 套接字覆盖为一个可由 `headscale` 用户或组写入的路径：

    ```yaml title="config.yaml"
    unix_socket: /var/run/headscale/headscale.sock
    ```

1.  重新加载 systemd 以加载新的配置文件：

    ```shell
    systemctl daemon-reload
    ```

1.  启用并启动新的 headscale 服务：

    ```shell
    systemctl enable --now headscale
    ```

1.  验证 headscale 是否按预期运行：

    ```shell
    systemctl status headscale
    ```

请继续阅读[入门指南](../../usage/getting-started.md)以注册您的第一台机器。