# 功能特性

Headscale 致力于提供一个自托管的开源替代方案，以取代 Tailscale 控制服务器。Headscale 的目标是
为自托管爱好者和业余爱好者提供一个开源服务器，用于他们的项目和实验环境。本页面
提供了 Headscale 的功能概览以及与 Tailscale 控制服务器的兼容性：

- [x] 完整支持 Tailscale 的基础功能
- [x] [节点注册](../ref/registration.md)
    - [x] [网页认证](../ref/registration.md#web-authentication)
    - [x] [预认证密钥](../ref/registration.md#pre-authenticated-key)
- [x] [DNS](../ref/dns.md)
    - [x] [MagicDNS](https://tailscale.com/kb/1081/magicdns)
    - [x] [全局和受限的 DNS 服务器（拆分 DNS）](https://tailscale.com/kb/1054/dns#nameservers)
    - [x] [搜索域](https://tailscale.com/kb/1054/dns#search-domains)
    - [x] [额外 DNS 记录（Headscale 独有）](../ref/dns.md#setting-extra-dns-records)
- [x] [Taildrop（文件共享）](https://tailscale.com/kb/1106/taildrop)
- [x] [标签](../ref/tags.md)
- [x] [路由](../ref/routes.md)
    - [x] [子网路由器](../ref/routes.md#subnet-router)
    - [x] [出口节点](../ref/routes.md#exit-node)
- [x] 双协议栈（IPv4 和 IPv6）
- [x] 短暂节点
- [x] 内置 [DERP 服务器](../ref/derp.md)
- [x] 访问控制列表（[GitHub 标签 "policy"](https://github.com/juanfont/headscale/labels/policy%20%F0%9F%93%9D)）
    - [x] 通过 API 管理 ACL
    - [x] 部分 [自动分组](https://tailscale.com/kb/1396/targets#autogroups)，目前包括：`autogroup:internet`,
      `autogroup:nonroot`, `autogroup:member`, `autogroup:tagged`, `autogroup:self`
    - [x] [自动批准者](https://tailscale.com/kb/1337/acl-syntax#auto-approvers) 用于 [子网
      路由器](../ref/routes.md#automatically-approve-routes-of-a-subnet-router) 和 [出口
      节点](../ref/routes.md#automatically-approve-an-exit-node-with-auto-approvers)
    - [x] [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh)
* [x] [使用单点登录（OpenID Connect）注册节点](../ref/oidc.md)（[GitHub 标签 "OIDC"](https://github.com/juanfont/headscale/labels/OIDC)）
    - [x] 基本注册
    - [x] 从身份提供商更新用户资料
    - [ ] OIDC 组不能用于 ACL
- [ ] [Funnel](https://tailscale.com/kb/1223/funnel)（[#1040](https://github.com/juanfont/headscale/issues/1040)）
- [ ] [Serve](https://tailscale.com/kb/1312/serve)（[#1234](https://github.com/juanfont/headscale/issues/1921)）
- [ ] [网络流日志](https://tailscale.com/kb/1219/network-flow-logs)（[#1687](https://github.com/juanfont/headscale/issues/1687)）