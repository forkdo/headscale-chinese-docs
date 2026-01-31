# 从源码构建

!!! warning "社区文档"

    本页面未由 headscale 作者积极维护，而是由社区成员编写。它 _未_ 经过 headscale 开发者验证。

    **它可能已过时，并可能遗漏必要的步骤**。

可以使用最新版本的 [Go](https://golang.org) 和 [Buf](https://buf.build)
(Protobuf 生成器) 从源码构建 headscale。有关更多信息，请参阅 [GitHub
README 中的贡献指南](https://github.com/juanfont/headscale#contributing)。

## OpenBSD

### 从源码安装

```shell
# 安装依赖
pkg_add go git

git clone https://github.com/juanfont/headscale.git

cd headscale

# 可选：切换到一个发布版本
# 选项 a. 你可以在 https://github.com/juanfont/headscale/releases/latest 找到官方发布版本
# 选项 b. 获取最新标签，这可能是测试版
latestTag=$(git describe --tags `git rev-list --tags --max-count=1`)

git checkout $latestTag

go build -ldflags="-s -w -X github.com/juanfont/headscale/hscontrol/types.Version=$latestTag" -X github.com/juanfont/headscale/hscontrol/types.GitCommitHash=HASH" github.com/juanfont/headscale

# 设置可执行权限
chmod a+x headscale

# 复制到 /usr/local/sbin
cp headscale /usr/local/sbin
```

### 通过交叉编译从源码安装

```shell
# 安装依赖
# 1. go v1.20+: 0.21 之后的 headscale 需要 go 1.20+ 才能编译
# 2. gmake: headscale 仓库中的 Makefile 使用 GNU make 语法

git clone https://github.com/juanfont/headscale.git

cd headscale

# 可选：切换到一个发布版本
# 选项 a. 你可以在 https://github.com/juanfont/headscale/releases/latest 找到官方发布版本
# 选项 b. 获取最新标签，这可能是测试版
latestTag=$(git describe --tags `git rev-list --tags --max-count=1`)

git checkout $latestTag

make build GOOS=openbsd

# 将 headscale 复制到 openbsd 机器，并放置在 /usr/local/sbin
```