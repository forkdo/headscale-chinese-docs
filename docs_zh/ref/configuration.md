# 配置

- Headscale 从 YAML 文件加载其配置
- 它会在以下路径中搜索 `config.yaml`：
    - `/etc/headscale`
    - `$HOME/.headscale`
    - 当前工作目录
- 要从其他路径加载配置，请使用：
    - 命令行标志 `-c`、`--config`
    - 环境变量 `HEADSCALE_CONFIG`
- 使用以下命令验证配置文件：`headscale configtest`

!!! example "从 [GitHub 仓库获取示例配置](https://github.com/juanfont/headscale/blob/main/config-example.yaml)"

    始终选择与您使用的发布版本[相同的 GitHub 标签](https://github.com/juanfont/headscale/tags)，
    以确保您获得正确的示例配置。`main` 分支可能包含尚未发布的更改。

    === "在 GitHub 上查看"

        * 开发版本：<https://github.com/juanfont/headscale/blob/main/config-example.yaml>
        * 版本 {{ headscale.version }}：<https://github.com/juanfont/headscale/blob/v{{ headscale.version }}/config-example.yaml>

    === "使用 `wget` 下载"

        ```shell
        # 开发版本
        wget -O config.yaml https://raw.githubusercontent.com/juanfont/headscale/main/config-example.yaml

        # 版本 {{ headscale.version }}
        wget -O config.yaml https://raw.githubusercontent.com/juanfont/headscale/v{{ headscale.version }}/config-example.yaml
        ```

    === "使用 `curl` 下载"

        ```shell
        # 开发版本
        curl -o config.yaml https://raw.githubusercontent.com/juanfont/headscale/main/config-example.yaml

        # 版本 {{ headscale.version }}
        curl -o config.yaml https://raw.githubusercontent.com/juanfont/headscale/v{{ headscale.version }}/config-example.yaml
        ```