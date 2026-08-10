# 社区软件包

多个 Linux 发行版和社区成员为 headscale 提供了软件包。这些软件包可以替代 headscale 维护者提供的[官方发行版](official.md)。此类软件包针对其目标操作系统提供了更好的集成，通常具有以下特点：

- 设置专用本地用户账户来运行 headscale
- 提供默认配置
- 将 headscale 安装为系统服务

!!! warning "社区软件包可能已过时"

    此页面上提到的软件包可能已过时或无人维护。请使用[官方发行版](official.md)获取当前稳定版本或[测试预发行版本](main.md)。

    [![Packaging status](https://repology.org/badge/vertical-allrepos/headscale.svg)](https://repology.org/project/headscale/versions)

## Arch Linux

Arch Linux 提供了 headscale 软件包，可通过以下命令安装：

```shell
pacman -S headscale
```

## Fedora、RHEL、CentOS

一个适用于各种基于 RPM 发行版的第三方仓库位于：
<https://copr.fedorainfracloud.org/coprs/jonathanspw/headscale/>。该网站提供了详细的设置和安装说明。

## Nix、NixOS

Nix 软件包可通过以下方式获取：`headscale`。有关安装详情，请参阅 [NixOS 软件包网站](https://search.nixos.org/packages?show=headscale)。

## Gentoo

```shell
emerge --ask net-vpn/headscale
```

Gentoo 专用文档可在[此处](https://wiki.gentoo.org/wiki/User:Maffblaster/Drafts/Headscale)获取。

## OpenBSD

Headscale 在 ports 中可用。该 port 使用 `rc.d` 将 headscale 安装为系统服务，并在安装时提供使用说明。

```shell
pkg_add headscale
```