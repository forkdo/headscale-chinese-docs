# OpenID Connect

Headscale 支持通过 OpenID Connect (OIDC) 使用外部身份提供商进行身份验证。其功能包括：

- 通过 OpenID Connect 发现协议自动配置
- [代码交换证明密钥 (PKCE) 代码验证](#enable-pkce-recommended)
- [基于用户域名、电子邮件地址或组成员身份进行授权](#authorize-users-with-filters)
- 同步[标准 OIDC 声明](#supported-oidc-claims)

已知问题和限制请参阅[限制](#limitations)。

## 配置

OpenID 需要在 Headscale 和身份提供商中进行配置：

- Headscale：Headscale [配置](configuration.md)中的 `oidc` 部分包含所有可用配置选项及其说明和默认值。
- 身份提供商：具体说明请参阅身份提供商的官方文档。此外，下方[特定身份提供商的配置](#identity-provider-specific-configuration)部分可能包含一些有用的提示。

### 基本配置

基本配置将 Headscale 连接到身份提供商，通常需要：

- 来自身份提供商的 OpenID Connect 颁发者 URL。Headscale 使用 OpenID Connect 发现协议 1.0 自动获取 OpenID 配置参数（例如：`https://sso.example.com`）。
- 来自身份提供商的客户端 ID（例如：`headscale`）。
- 由身份提供商生成的客户端密钥（例如：`generated-secret`）。
- 身份提供商的回调 URI（例如：`https://headscale.example.com/oidc/callback`）。

=== "Headscale"

    ```yaml
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
    ```

=== "身份提供商"

    * 创建新的机密客户端（`客户端 ID`、`客户端密钥`）
    * 将 Headscale 的 OIDC 回调 URL 添加为有效重定向 URL：`https://headscale.example.com/oidc/callback`
    * 配置其他参数以改善用户体验，例如：名称、描述、徽标等

### 启用 PKCE（推荐）

代码交换证明密钥 (PKCE) 通过防止授权代码拦截攻击，为 OAuth 2.0 授权码流程增加了一层额外的安全性，参见：<https://datatracker.ietf.org/doc/html/rfc7636>。建议启用 PKCE，并且需要在 Headscale 和身份提供商中进行配置：

=== "Headscale"

    ```yaml hl_lines="5-6"
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
      pkce:
        enabled: true
    ```

=== "身份提供商"

    * 为 headscale 客户端启用 PKCE
    * 将 PKCE 质询方法设置为 "S256"

### 使用过滤器授权用户

Headscale 允许根据用户的域名、电子邮件地址或组成员身份过滤允许的用户。这些过滤器有助于应用额外的限制并控制允许哪些用户加入。默认情况下，过滤器处于禁用状态，一旦与身份提供商的身份验证成功，用户就可以加入。如果配置了多个过滤器，用户需要通过所有过滤器。

=== "允许的域名"

    * 将每个身份验证用户的电子邮件域名与允许的域名列表进行比对，仅授权电子邮件域名与 `example.com` 匹配的用户。
    * 除非[禁用了电子邮件验证](#control-email-verification)，否则需要经过验证的电子邮件地址。
    * 允许访问：`alice@example.com`
    * 拒绝访问：`bob@example.net`

    ```yaml hl_lines="5-6"
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
      allowed_domains:
        - "example.com"
    ```

=== "允许的用户/电子邮件"

    * 将每个身份验证用户的电子邮件地址与允许的电子邮件地址列表进行比对，仅授权电子邮件属于 `allowed_users` 列表的用户。
    * 除非[禁用了电子邮件验证](#control-email-verification)，否则需要经过验证的电子邮件地址。
    * 允许访问：`alice@example.com`、`bob@example.net`
    * 拒绝访问：`mallory@example.net`

    ```yaml hl_lines="5-7"
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
      allowed_users:
        - "alice@example.com"
        - "bob@example.net"
    ```

=== "允许的组"

    * 使用每个身份验证用户的 OIDC `groups` 声明获取其组成员身份，仅授权至少属于其中一个引用组的用户。
    * 允许访问：`headscale_users` 组中的用户
    * 拒绝访问：无组用户、有其他组的用户

    ```yaml hl_lines="5-7"
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
      scope: ["openid", "profile", "email", "groups"]
      allowed_groups:
        - "headscale_users"
    ```

### 控制电子邮件验证

Headscale 使用身份提供商的 `email` 声明将其电子邮件地址同步到其用户配置文件。默认情况下，仅当身份提供商通过 `email_verified: true` 声明报告电子邮件地址已验证时，才会同步用户的电子邮件地址。

如果身份提供商不发送 `email_verified` 声明或不需要电子邮件验证，则可能允许未经验证的电子邮件。在这种情况下，用户的电子邮件地址始终会同步到用户配置文件。

```yaml hl_lines="5"
oidc:
  issuer: "https://sso.example.com"
  client_id: "headscale"
  client_secret: "generated-secret"
  email_verified_required: false
```

### 自定义节点过期时间

节点过期时间是指节点通过 OpenID Connect 进行身份验证后，直到过期并需要重新进行身份验证的时间量。默认节点过期时间为 180 天。这可以自定义，也可以设置为访问令牌的过期时间。

=== "自定义节点过期时间"

    ```yaml hl_lines="5"
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
      expiry: 30d   # 使用 0 禁用节点过期
    ```

=== "使用访问令牌的过期时间"

    请记住，访问令牌通常是短期令牌，会在几分钟内过期。您必须配置身份提供商中的令牌过期时间，以避免频繁重新进行身份验证。


    ```yaml hl_lines="5"
    oidc:
      issuer: "https://sso.example.com"
      client_id: "headscale"
      client_secret: "generated-secret"
      use_expiry_from_token: true
    ```

!!! tip "使节点过期并强制重新进行身份验证"

    可通过以下方式立即使节点过期：
    ```console
    headscale node expire -i <NODE_ID>
    ```

### 在策略中引用用户

您可以通过以下方式在 Headscale 策略中引用用户：

- 电子邮件地址
- 用户名
- 提供商标识符（仅在数据库中或从身份提供商获取）

!!! note "策略中的用户标识符必须包含单个 `@`"

    Headscale 策略需要单个 `@` 来引用用户。如果用户名或提供商标识符不包含单个 `@`，则需要在末尾添加。例如：用户名 `ssmith` 必须写为 `ssmith@` 才能在策略中正确识别为用户。

!!! warning "用户可能会更新电子邮件地址或用户名"

    许多身份提供商允许用户更新自己的配置文件。根据身份提供商及其配置，用户名或电子邮件地址的值可能会随时间而变化。这可能会对 Headscale 产生意外后果，策略可能不再起作用，或者用户可能通过劫持现有用户名或电子邮件地址获得更多访问权限。

## 支持的 OIDC 声明

Headscale 使用[标准 OIDC 声明](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)在每次登录时填充和更新其本地用户配置文件。OIDC 声明从 ID 令牌和 UserInfo 端点读取。

| Headscale 配置文件 | OIDC 声明            | 说明 / 示例                                                                                       |
| ------------------ | ------------------- | ------------------------------------------------------------------------------------------------ |
| 电子邮件地址       | `email`             | 仅同步经过验证的电子邮件，除非配置了 `email_verified_required: false`                            |
| 显示名称           | `name`              | 例如：`Sam Smith`                                                                                |
| 用户名             | `preferred_username`| 取决于身份提供商，例如：`ssmith`、`ssmith@idp.example.com`、`\\example.com\ssmith`               |
| 个人资料图片       | `picture`           | 个人资料图片或头像的 URL                                                                         |
| 提供商标识符       | `iss`、`sub`        | 用户的稳定且唯一的标识符，通常是 `iss` 和 `sub` OIDC 声明的组合                                  |
|                    | `groups`            | [仅用于过滤允许的组](#authorize-users-with-filters)                                              |

## 限制

- 对 OpenID Connect 的支持旨在实现通用性和供应商独立性。它仅对特定身份提供商的特殊情况提供有限支持。
- OIDC 组不能用于 ACL。
- 身份提供商提供的用户名必须符合以下模式：
    - 用户名长度必须至少为两个字符。
    - 它只能包含字母、数字、连字符、点、下划线和最多一个 `@`。
    - 用户名必须以字母开头。

有关 OIDC 相关问题，请参阅 [GitHub 标签 "OIDC"](https://github.com/juanfont/headscale/labels/OIDC)。

## 特定身份提供商的配置

!!! warning "第三方软件和服务"

本节文档专门针对第三方软件和服务。我们建议用户阅读第三方文档，了解如何配置和集成 OIDC 客户端。有关 Headscale 的 OIDC 相关配置设置，请参阅[配置](#configuration)部分。

任何支持 OpenID Connect 的身份提供商都应该可以与 Headscale“直接配合使用”。以下身份提供商已知可以正常工作：

- [Authelia](#authelia)
- [Authentik](#authentik)
- [Kanidm](#kanidm)
- [Keycloak](#keycloak)

### Authelia

Authelia 完全受 Headscale 支持。

### Authentik

- Authentik 完全受 Headscale 支持。
- [Headscale 不支持 JSON Web 加密](https://github.com/juanfont/headscale/issues/2446)。请在提供程序部分将 `Encryption Key` 字段留空。

### Google OAuth

!!! warning "由于缺少 preferred_username，因此没有用户名"

    当请求 `profile` 范围时，Google OAuth 不会发送 `preferred_username` 声明。Headscale 中的用户名将显示为空白/未设置。

要将 Headscale 与 Google 集成，您需要拥有一个 [Google Cloud Console](https://console.cloud.google.com) 账户。

如果您需要让域外的用户进行身份验证，Google OAuth 有一个[验证流程](https://support.google.com/cloud/answer/9110914?hl=en)。如果您只需要对来自您域名的用户进行身份验证（即 `@example.com`），则无需经过验证流程。

但是，如果您没有域名，或者需要添加域外的用户，可以通过 Google Console 手动添加电子邮件。

#### 步骤

1. 访问 [Google Console](https://console.cloud.google.com) 并登录，如果没有账户，请创建一个。
2. 创建一个项目（如果您还没有项目）。
3. 在左侧菜单中，转到 `API 和服务` -> `凭据`
4. 点击 `创建凭据` -> `OAuth 客户端 ID`
5. 在 `应用类型` 下，选择 `Web 应用`
6. 对于 `名称`，输入您喜欢的任何名称
7. 在 `已授权的跳转 URI` 下，添加 Headscale 的 OIDC 回调 URL：`https://headscale.example.com/oidc/callback`
8. 点击表单底部的 `保存`
9. 记下 `客户端 ID` 和 `客户端密钥`，如果需要，也可以下载以供参考。
10. [按照“基本配置”步骤配置 Headscale](#basic-configuration)。Google OAuth 的颁发者 URL 为：`https://accounts.google.com`。

### Kanidm

- Kanidm 完全受 Headscale 支持。
- [允许的组过滤器](#authorize-users-with-filters) 的组需要使用其完整的 SPN 指定，例如：`headscale_users@sso.example.com`。

### Keycloak

Keycloak 完全受 Headscale 支持。

#### 使用允许的组过滤器的额外配置

Keycloak 没有针对 OIDC `groups` 声明的内置客户端范围。只有在需要[基于组成员身份授权访问](#authorize-users-with-filters)时，才需要执行此额外配置步骤。

- 为 OpenID Connect 创建一个新的客户端范围 `groups`：
    - 配置一个名为 `groups` 的 `Group Membership` 映射器，并将令牌声明名称设置为 `groups`。
    - 将该映射器至少添加到 UserInfo 端点。
- 为您的 Headscale 客户端配置新的客户端范围：
    - 编辑 Headscale 客户端。
    - 搜索客户端范围 `group`。
    - 将其添加并分配类型为 `Default`。
- [在 Headscale 中配置允许的组](#authorize-users-with-filters)。如何指定组取决于 Keycloak 的 `Full group path` 选项：
    - 启用了 `Full group path`：组包含其完整路径，例如 `/top/group1`
    - 禁用了 `Full group path`：仅使用组的名称，例如 `group1`

### Microsoft Entra ID

要将 Headscale 与 Microsoft Entra ID 集成，您需要配置一个具有正确范围和跳转 URI 的应用注册。

[按照“基本配置”步骤配置 Headscale](#basic-configuration)。Microsoft Entra ID 的颁发者 URL 为：`https://login.microsoftonline.com/<tenant-UUID>/v2.0`。以下 `extra_params` 可能有用：

- `domain_hint: example.com` 以使用您自己的域
- `prompt: select_account` 以在登录期间强制显示账户选择器

当将 Microsoft Entra ID 与[允许的组过滤器](#authorize-users-with-filters)一起使用时，请配置 Headscale OIDC 范围而不包含 `groups` 声明，例如：

```yaml
oidc:
  scope: ["openid", "profile", "email"]
```

[允许的组过滤器](#authorize-users-with-filters) 的组需要使用其组 ID（UUID）而不是组名来指定。