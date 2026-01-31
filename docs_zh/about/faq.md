# 常见问题

## 什么是 Headscale 的设计目标？

Headscale 旨在实现一个自托管、开源的 [Tailscale](https://tailscale.com/) 控制服务器替代方案。Headscale 的目标是为自托管用户和爱好者提供一个开源服务器，可用于他们的项目和实验环境。它实现了有限的功能范围，支持单一的 Tailscale 网络（tailnet），适用于个人使用或小型开源组织。

## 如何参与贡献？

Headscale 采用“开源，确认式贡献”的模式，这意味着任何贡献都必须在提交之前与维护者进行讨论。

请参阅 [Contributing](contributing.md) 以获取更多信息。

## 为何选择“确认式贡献”这一模式？

维护者都有全职工作和家庭，我们希望避免倦怠。我们也希望避免贡献者因 PR 被拒绝而感到沮丧。

在 PR 提交之前，我们很乐意进行邮件交流或安排专门的通话。

## 特定功能 X 何时以及为何会被实现？

我们不知道。我们可能正在开发它。如果您有兴趣贡献，请提交一个功能请求。

请注意，我们可能拒绝特定贡献的原因有很多：

- 在自托管环境中，该功能无法以合理的方式实现。
- 由于我们通过逆向工程 Tailscale 来满足自己的好奇心，我们可能更倾向于自己实现该功能。
- 您未随贡献一起提交单元测试和集成测试。

## 是否支持使用 Y 方法部署 headscale？

我们目前支持使用我们的二进制文件和 DEB 包部署 headscale。请参阅[使用官方发布版本的安装指南](../setup/install/official.md)了解更多信息。

此外，您还可以使用社区或发行版提供的软件包。请参阅[使用社区包的安装指南](../setup/install/community.md)了解更多信息。

为了方便起见，我们还[构建了包含 headscale 的容器镜像](../setup/install/container.md)。但请注意，**我们不正式支持使用 Docker 部署 headscale**。在我们的[Discord 服务器](https://discord.gg/c84AZQhmpx)上，有一个 "docker-issues" 频道，您可以在其中向社区寻求 Docker 相关帮助。

## 建议的更新路径是什么？更新时能否跳过多版本？

请按照[升级指南](../setup/upgrade.md)中的步骤更新现有的 Headscale 安装。如果您落后多个版本，建议每次更新一个稳定版本（例如 0.24.0 → 0.25.1 → 0.26.1）。您应始终选择最新的补丁版本。

请务必查看[变更日志](https://github.com/juanfont/headscale/blob/main/CHANGELOG.md)，获取版本特定的升级说明和破坏性变更。

## 可扩展性 / Headscale 支持多少客户端？

取决于具体情况。正如经常强调的，Headscale 并非企业级软件，我们的重点是家庭实验室用户和自托管用户。当然，我们也无法阻止人们在商业/专业环境中使用它，经常有人询问可扩展性问题。

请注意，在开发 Headscale 时，性能并不是考虑因素，因为目标用户群被认为是设备数量有限的用户。我们更关注正确性以及与 Tailscale SaaS 的功能对齐。

为了帮助您了解是否可以将 Headscale 用于您的场景，我将描述两个场景来解释 Headscale 的主要瓶颈：

1. 包含 1000 台服务器的环境

   - 这些服务器很少“移动”（更改其端点）
   - 很少添加新节点

2. 包含 80 台笔记本电脑/手机（终端用户设备）的环境

   - 节点经常移动，例如从家庭切换到办公室

Headscale 计算所有需要相互通信的节点的地图，创建此“世界地图”需要大量 CPU 时间。当发生需要更改此地图的事件时，整个“世界”将被重新计算，并为网络中的每个节点创建新的“世界地图”。

这意味着在某些条件下，如果网络中几乎没有变化，Headscale 可能能够处理数百台设备（可能更多）。例如，在场景 1 中，由于网络规模庞大，计算世界地图的过程非常耗费资源，但一旦地图创建完成且节点不再变化，Headscale 实例将很可能回到非常低的资源使用状态，直到下次发生需要更新地图的事件。

在场景 2 中，由于网络规模较小，计算世界地图的过程资源消耗较少，但是节点类型很可能频繁变化，这将导致持续的资源使用。

当两种场景重叠时，Headscale 将开始遇到困难，例如许多节点频繁变化将导致资源使用持续处于高位。在最坏情况下，等待地图更新的节点队列可能会增长到 Headscale 无法跟上的程度，节点将永远无法了解世界当前的状态。

我们预计随着代码库的改进，性能将逐步提升，但这并非我们的主要关注点。总体而言，我们永远不会为了提升速度而牺牲代码的可维护性和可读性。我们是一个小团队，必须优化可维护性。

## 应该使用哪种数据库？

我们推荐使用 SQLite 作为 headscale 的数据库：

- SQLite 配置简单，易于使用
- 它在所有 headscale 的使用场景中都能很好地扩展
- 开发和测试主要在 SQLite 上进行
- PostgreSQL 仍受支持，但已进入“维护模式”

headscale 项目本身不提供从 PostgreSQL 迁移到 SQLite 的工具。请查看 [相关工具文档](../ref/integration/tools.md)，了解社区提供的迁移工具。

数据库选择对服务器性能影响很小或没有影响，详见[可扩展性 / Headscale 支持多少客户端？](#scaling-how-many-clients-does-headscale-support)了解 Headscale 资源使用情况。

## 为什么我的反向代理与 headscale 无法正常工作？

我们不知道。我们自己不使用 headscale 的反向代理，因此没有相关经验。我们有[社区文档](../ref/integration/reverse-proxy.md)说明如何配置各种反向代理，并在我们的[Discord 服务器](https://discord.gg/c84AZQhmpx)上设有专门的 "reverse-proxy-issues" 频道，您可以在其中向社区寻求帮助。

## 我可以在同一台机器上使用 headscale 和 tailscale 吗？

在一台同时加入 tailnet 的机器上运行 headscale 可能会导致子网路由器、流量中继节点和 MagicDNS 的问题。虽然可能正常工作，但不被支持。

## 为什么两个节点在彼此的状态中可见，即使 ACL 只允许单向流量？

一个常见的用例是只允许从一个节点到另一个节点的流量，而不允许反向流量。例如，管理员的工作站应能连接到所有节点，但节点本身不应能连接回管理员的设备。为什么所有节点在 `tailscale status` 的输出中都能看到管理员的工作站？

这本质上就是 Tailscale 的工作方式。如果允许单向流量，则两个节点在各自的 `tailscale status` 输出中都能看到彼此。流量仍然根据 ACL 进行过滤，唯一的例外是 `tailscale ping`，它始终在两个方向都允许。

另请参阅 <https://tailscale.com/kb/1087/device-visibility>。

## 我的策略存储在数据库中，且由于策略无效，Headscale 拒绝启动。如何恢复？

Headscale 在启动时会检查策略是否有效，如果检测到错误则拒绝启动。错误信息会指出策略中哪一部分无效。请按照以下步骤修复策略：

- 将策略导出到文件：`headscale policy get --bypass-grpc-and-access-database-directly > policy.json`
- 编辑并修复 `policy.json`。使用命令 `headscale policy check --file policy.json` 验证策略。
- 加载修改后的策略：`headscale policy set --bypass-grpc-and-access-database-directly --file policy.json`
- 按常规方式启动 Headscale。

!!! warning "需要完整的服务器配置"

    上述获取/设置策略的命令需要包含数据库设置的完整服务器配置文件。仅使用最小配置通过远程 CLI 控制 Headscale 是不够的。您可以使用 `headscale -c /path/to/config.yaml` 指定备用配置文件的路径。

## 如何避免将日志发送给 Tailscale Inc？

Tailscale 客户端[收集有关其操作和与其他客户端连接尝试的日志](https://tailscale.com/kb/1011/log-mesh-traffic#client-logs)，并将其发送到由 Tailscale Inc 运营的中心日志服务。

默认情况下，Headscale 指示客户端禁用向中心日志服务提交日志。此配置在客户端成功连接到 Headscale 后由客户端应用。请参阅[配置文件](../ref/configuration.md)中的配置选项 `logtail.enabled` 了解详情。

此外，也可以在客户端禁用日志记录。这与 Headscale 无关，退出客户端日志记录会在客户端启动早期就禁用日志提交。配置因操作系统而异，通常通过设置环境变量 `TS_NO_LOGS_NO_SUPPORT=true` 或向 `tailscaled` 传递标志 `--no-logs-no-support` 来实现。详见 <https://tailscale.com/kb/1011/log-mesh-traffic#opting-out-of-client-logging> 了解详情。