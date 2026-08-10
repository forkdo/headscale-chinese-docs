# 升级现有安装

!!! tip "必需的升级路径"

    必须从一个稳定版本依次升级到下一个稳定版本（例如 0.26.0 → 0.27.1 → 0.28.0），中间不能跳过次要版本。您应始终选择最新的可用补丁版本。

将现有的 Headscale 安装更新到新版本：

- 查看 [GitHub 发布页面](https://github.com/juanfont/headscale/releases) 上关于新版本的公告。该页面列出了此次发布的变更内容、可能的破坏性变更以及特定版本的升级说明。
- 停止 Headscale
- **[创建您的安装备份](#backup)**
- 将 Headscale 更新到新版本，建议采用与之前相同的安装方式进行升级。
- 对比并更新 [配置文件](../ref/configuration.md)。
- 启动 Headscale

## 备份 (Backup)

Headscale 在升级过程中会执行数据库迁移，因此我们强烈建议在升级前创建数据库备份。Headscale 的完整备份取决于您的具体部署方式，以下列举了一些典型场景。

=== "标准安装"

    遵循我们的[官方版本](install/official.md)安装指南的安装会使用以下路径：

    - [配置文件](../ref/configuration.md)：`/etc/headscale/config.yaml`
    - 数据目录：`/var/lib/headscale`
    - 以 SQLite 作为数据库：`/var/lib/headscale/db.sqlite`

    ```console
    TIMESTAMP=$(date +%Y%m%d%H%M%S)
    cp -aR /etc/headscale /etc/headscale.backup-$TIMESTAMP
    cp -aR /var/lib/headscale /var/lib/headscale.backup-$TIMESTAMP
    ```

=== "容器"

    遵循我们的[容器](install/container.md)安装指南的安装会使用单一的源卷目录，其中包含配置文件、数据目录和 SQLite 数据库。

    ```console
    cp -aR /path/to/headscale /path/to/headscale.backup-$(date +%Y%m%d%H%M%S)
    ```

=== "PostgreSQL"

    请遵循 PostgreSQL 的 [Backup and Restore](https://www.postgresql.org/docs/current/backup.html) 文档来创建您的 PostgreSQL 数据库备份。