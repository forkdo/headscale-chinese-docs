# 在容器中运行 headscale

!!! warning "社区文档"

    本页面并非由 headscale 作者积极维护，而是由社区成员编写。headscale 开发者**并未**验证其内容。

    **内容可能已过时，且可能遗漏必要步骤**。

本文档旨在向用户展示如何在容器中设置和运行 headscale。需要容器运行时环境，例如 [Docker](https://www.docker.com) 或 [Podman](https://podman.io)。容器镜像可在 [Docker Hub](https://hub.docker.com/r/headscale/headscale) 和 [GitHub 容器镜像仓库](https://github.com/juanfont/headscale/pkgs/container/headscale) 上找到。容器镜像 URL 如下：

- [Docker Hub](https://hub.docker.com/r/headscale/headscale)：`docker.io/headscale/headscale:<VERSION>`
- [GitHub 容器镜像仓库](https://github.com/juanfont/headscale/pkgs/container/headscale)：
  `ghcr.io/juanfont/headscale:<VERSION>`

## 配置并运行 headscale

1.  在容器主机上创建一个目录，用于存储 headscale 的[配置](../../ref/configuration.md)和 [SQLite](https://www.sqlite.org/) 数据库：

    ```shell
    mkdir -p ./headscale/{config,lib}
    cd ./headscale
    ```

1.  下载适用于您所选版本的示例配置文件，并将其保存为：`$(pwd)/config/config.yaml`。根据您的本地环境调整配置。详细信息请参阅[配置](../../ref/configuration.md)。

1.  在先前创建的 `./headscale` 目录中启动 headscale：

    ```shell
    docker run \
      --name headscale \
      --detach \
      --read-only \
      --tmpfs /var/run/headscale \
      --volume "$(pwd)/config:/etc/headscale:ro" \
      --volume "$(pwd)/lib:/var/lib/headscale" \
      --publish 127.0.0.1:8080:8080 \
      --publish 127.0.0.1:9090:9090 \
      --health-cmd "CMD headscale health" \
      docker.io/headscale/headscale:<VERSION> \
      serve
    ```

    注意：如果您希望从外部访问容器，请使用 `0.0.0.0:8080:8080` 替代 `127.0.0.1:8080:8080`。

    此命令将本地目录挂载到容器内部，将端口 8080 和 9090 转发到容器外部，从而使 headscale 实例可用，然后分离容器以便 headscale 在后台运行。

    类似的 `docker-compose` 配置：

    ```yaml title="docker-compose.yaml"
    services:
      headscale:
        image: docker.io/headscale/headscale:<VERSION>
        restart: unless-stopped
        container_name: headscale
        read_only: true
        tmpfs:
          - /var/run/headscale
        ports:
          - "127.0.0.1:8080:8080"
          - "127.0.0.1:9090:9090"
        volumes:
          # 请将 <HEADSCALE_PATH> 设置为先前创建的 headscale 目录的绝对路径。
          - <HEADSCALE_PATH>/config:/etc/headscale:ro
          - <HEADSCALE_PATH>/lib:/var/lib/headscale
        command: serve
        healthcheck:
            test: ["CMD", "headscale", "health"]
    ```

1.  验证 headscale 是否正在运行：

    跟踪容器日志：

    ```shell
    docker logs --follow headscale
    ```

    验证正在运行的容器：

    ```shell
    docker ps
    ```

    验证 headscale 是否可用：

    ```shell
    curl http://127.0.0.1:8080/health
    ```

请继续阅读[入门指南](../../usage/getting-started.md)以注册您的第一台机器。

## 调试在 Docker 中运行的 headscale

Headscale 容器镜像基于“无发行版”（distroless）镜像，不包含 shell 或任何其他调试工具。如果您需要调试在 Docker 容器中运行的 headscale，可以使用 `-debug` 变体，例如 `docker.io/headscale/headscale:x.x.x-debug`。

### 运行调试版 Docker 容器

要运行调试版 Docker 容器，请使用与上述完全相同的命令，但将 `docker.io/headscale/headscale:x.x.x` 替换为 `docker.io/headscale/headscale:x.x.x-debug`（其中 `x.x.x` 是 headscale 的版本号）。这两个容器彼此兼容，因此您可以在它们之间交替使用。

### 在调试容器中执行命令

调试容器中的默认命令是运行 `headscale`，该程序位于容器内部的 `/ko-app/headscale` 路径。

此外，调试容器还包含一个极简的 Busybox shell。

要在容器中启动 shell，请使用：

```shell
docker run -it docker.io/headscale/headscale:x.x.x-debug sh
```

您也可以直接执行命令，例如本示例中的 `ls /ko-app`：

```shell
docker run docker.io/headscale/headscale:x.x.x-debug ls /ko-app
```

使用 `docker exec -it` 可以在现有容器中运行命令。