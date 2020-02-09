# 如何运用Wercker开发与部署应用程序

### 在这篇文章中：

- [Wercker 是什么？](javascript:;)
- [准备工作](javascript:;)
- [设置演示程序以及版本控制](javascript:;)
- [设置 Wercker 帐户](javascript:;)
- 配置 wercker.yml 和练习示例
  - [jClocksGMT 示例](javascript:;)
  - [Hello.go 示例](javascript:;)
  - [Golang入门 —— 使用 Wercker CLI 的示例](javascript:;)
- 怎样添加应用程序 Wercker？
  - [配置应用程序](javascript:;)
- [下一步计划](javascript:;)
- [更多信息](javascript:;)

# Wercker 是什么？

Wercker 是一款软件自动化工具，旨在为开发者和企业改善持续集成（CI, Continuous-Integration）和持续交付（CD, Continuous-Delivery）流程。这个工具支持创建自动化工作流（Workflow）或管道（Pipelines），它指定了一系列任务或命令，当将更改推到源存储库时，这些任务或命令将在代码上运行。

本指南将使用三个示例的 [Go ](https://golang.org/)应用程序来演示关于 Wercker 的安装与配置的基础知识，并展示如何使用这些应用程序来创建不同类型的工作流。

# 准备工作

1. 完成[入门](https://www.linode.com/docs/getting-started)指南以创建一个 Linode（虚拟专用服务器）。本指南中的命令是在 Ubuntu 16.04 中编写的，不过其他发行版应该也能使用。
2. 按照[服务器安全指南](https://cloud.tencent.com/docs/security/securing-your-server/?from=10680)创建一个标准帐户，强化 SSH 访问并删除不必要的网络服务。本指南将尽可能地使用`sudo`命令。
3. 更新你的软件包： `sudo apt update && sudo apt upgrade`
4. 本指南要求在您的 Linode 上安装 Docker。详情请参阅我们的另一指南：[如何安装 Docker 并拉起容器部署映像](https://www.linode.com/docs/applications/containers/how-to-install-docker-and-pull-images-for-container-deployment)。
5. 创建一个 GitHub 或类似的帐户。修改命令以匹配您选择的 git 变体。
6. 创建一个 Docker 帐户。其他服务可能需要修改命令以及示例。

# 设置演示程序以及版本控制

\1. 在你的本地计算机以及 Linode 上安装`git`： `sudo apt install git`

\2. 登录 GitHub 并 fork 以下仓库：

​    **·** [jClocksGMT](https://github.com/mcmastermind/jClocksGMT)，一个基本的 jQuery 数字与模拟时钟集合。

​    **·** [golang/example](https://github.com/golang/example)，由`golang`项目组制作的一组最轻量的`Go`应用实例。

​    **·** [golang 入门](https://github.com/wercker/getting-started-golang)，关于 Wercker 的`Go`应用示例。

\3. 在您的本地计算机上克隆您的 fork。（``需要根据实际情况替换）：    **·** `git clone https://github.com//jClocksGMT.git`

​    **·** `git clone https://github.com//example.git `

​    **·** `git clone https://github.com//getting-started-golang.git`

\4. 在您的系统上安装 Go，并确保它在您系统的`$PATH`中：    **·** `sudo apt install golang-go go version`

# 设置 Wercker 帐户

\1. 在 Web 浏览器中，导航到 [Wercker 主页](http://www.wercker.com/)并注册一个免费帐户。

![img](https://ask.qcloudimg.com/draft/1206219/qe1c58mjx4.png?imageView2/2/w/1620)

\2. 最简单的注册方法就是使用您的 GitHub 帐户。

![img](https://ask.qcloudimg.com/draft/1206219/0fex08zob1.png?imageView2/2/w/1620)

# 配置 wercker.yml 和练习示例

Wercker 的优点之一是它的简单性。通过一个`wercker.yml`配置文件管理需要进行多个步骤的自动化管道。您可以将步骤（Step）视为对操作流程的调用，而将管道视为一个或多个步骤的集合。

## jClocksGMT 示例

此示例演示了如何使用 Wercker 更新远程服务器上的源码（当 GitHub 仓库有更新时）。这是静态网站的常见用例：每当您从本地计算机上推送到 GitHub 时，托管该网站的服务器上的代码也会自动更新。

在`jClocksGMT`目录的根目录中创建一个`wercker.yml`文件，并粘贴下面的内容。替换`192.0.2.0`为您的 Linode 公共 IP 地址，并更新最后一行以使用正确的用户名和文件路径。`wercker.yml`文件中所有的缩进必须使用空格，而不能是 Tab。

| 1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 | box: debian # Build definition build:  # The steps that will be executed on build  steps:    # Installs openssh client and server.    - install-packages:        packages: openssh-client openssh-server    # Adds Linode server to the list of known hosts.    - add-to-known_hosts:            hostname: 192.0.2.0            local: true    # Adds the Wercker SSH key.    - add-ssh-key:            keyname: linode    # Custom code to be executed on remote Linode    - script:        name: Update code on remote Linode        code: \|          ssh username@<Linode IP or hostname> git -C /path/to/jClocksGMT pull |
| :--------------------------------------------------------- | :----------------------------------------------------------- |
|                                                            |                                                              |

每当（通过推送到仓库）触发 Wercker 运行时，Wercker 将加载 Docker 映像并从该映像运行指定的步骤。这就是为什么在 Linode 上运行的所有命令都以`ssh`开头。在这种情况下，该`wercker.yml`文件包含以下步骤：

- `box`：定义所使用的 Docker 映像。在本文情景中，是指定了一个全局 Debian 映像。
- `install-packages`：这一步是`apt-get install`命令的一个快捷方式。列出的所有软件包都将安装到您的容器中。
- `add-to-known_hosts`：将 Linode 的 IP 或域名添加到已知主机文件。
- `add-ssh-key`：将 Wercker 生成的公共 SSH 密钥添加到容器中。
- `script`：脚本是自定义步骤，它几乎可以执行所有命令，在本例中这段代码表示的是在远程 Linode 上执行的动作。

有了这个`build`管道，Wercker 每次运行时都会执行以下操作：

1. 在容器中加载 Debian 映像。
2. 安装必要的包，`openssh-client`和`openssh-server`。
3. 设置 Wercker 容器和 Linode 之间的 SSH 连接。
4. Debian 容器从远程 Linode 上运行`git pull`命令。

## Hello.go 示例

该示例演示了一个更复杂的管道——包含`build`和`deploy`的管道。这次，Wercker 将构建一个简单的 Go 应用程序并将其部署到 DockerHub，然后在将映像从 DockerHub 部署到远程 Linode。

\1. 去到`example`这一 fork 的根目录，并将`hello.go`文件复制到那里： `cp ./hello/hello.go .`

\2. 在同目录下创建`wercker.yml`文件：

```javascript
box: google/golang

build:

    steps:
    # Sets the go workspace and places your package
    # in the right place in the workspace tree
    - setup-go-workspace

    # Build the project
    - script:
        name: Build application
        code: |
            go get github.com/<user>/example
            go build -o myapp

    - script:
        name: Copy binary
        code: |
          cp myapp "$WERCKER_OUTPUT_DIR"

### Docker Deployment
deploy:

    # This deploys to DockerHub
    steps:
    - internal/docker-scratch-push:
        username: $DOCKER_USERNAME
        password: $DOCKER_PASSWORD
        repository: <docker-username>/myapp
        cmd: ./myapp

### Linode Deployment from Docker
linode:

    steps:
    # Installs openssh client and other dependencies.
    - install-packages:
        packages: openssh-client openssh-server
    # Adds Linode server to the list of known hosts.
    - add-to-known_hosts:
        hostname: 192.0.2.0
        local: true
    # Adds SSH key created by Wercker
    - add-ssh-key:
        keyname: linode
    # Custom code to pull image
    - script:
        name: pull latest image
        code: |
            ssh username@192.0.2.0 docker pull <docker-username>/myapp:latest
            ssh username@192.0.2.0 docker tag <docker-username>/myapp:latest <docker-username>/myapp:current
            ssh username@192.0.2.0 docker rmi <docker-username>/myapp:latest
```

此配置中带有三个管道：

\1. `build`管道：在本例中是用于构建应用程序的强制管道。由于此示例使用 Go 语言，因此最方便的`box`是官方的`google/golang`（附带了必要的工具配置）。该管道执行的步骤是：

​    **·** `setup-go-workspace`：准备好您的 Go 环境。

​    **·** `Build application`：运行名为`myapp`的示例应用程序的实际构建进程。应用保存在相应的工作区中。

​    **·** `Copy binary`：请记住，您正在处理临时管道。此步骤将应用的二进制文件保存为预定义的环境变量`$WERCKER_OUTPUT_DIR`，以便在下一个管道中使用它。

\2. `deploy`管道：从`$WERCKER_OUTPUT_DIR`中获取二进制文件，然后将其推送到 Docker 帐户。

​    **·** `internal/docker-scratch-push`：让所有的奇迹发生。使用环境变量`$DOCKER_USERNAME`和`$DOCKER_PASSWORD`，这样可以保存您的二进制文件到一个轻量级的`scratch`映像中。`repository`参数指定了要使用的 Docker 仓库。

\3. `linode`管道：头三个步骤，`install-packages`，`add-to-known_hosts`和`add-ssh-key`在以前的例子解释过：他们负责您的管道容器和你的 Linode 之间的 SSH 通信。 自定义的`-script`，`pull latest image`从上面示例中的第 48 行开始：

​    **·** 从 Docker Hub 中提取最新的映像构建。默认情况下，此映像被标记为`latest`（除非另有说明）。

​    **·** 克隆您的最新映像并标记其为`current`。

​    **·** 删除标记为`latest`的已拉起的映像，为下次更新做准备。

这是让 “当前” 应用程序一直运行的一种简单方法。请记住，这只是一个您可以自定义流程的示例。

## Golang入门 —— 使用 Wercker CLI 的示例

最后一个示例介绍了 **Wercker CLI**。此工具要求本地计算机上安装有 Docker。您可以在您的 Linode 中采用与 “[拉起容器部署映像](https://www.linode.com/docs/applications/containers/how-to-install-docker-and-pull-images-for-container-deployment)” 指南相同的向导。

1. 安装 Docker Machine： `curl -L https://github.com/docker/machine/releases/download/v0.12.2/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine`
2. 安装 Wercker CLI： `sudo curl -L https://s3.amazonaws.com/downloads.wercker.com/cli/stable/linux_amd64/wercker -o /usr/local/bin/wercker sudo chmod 777 /usr/local/bin/wercker`
3. 检查 CLI 是否已正确安装： `wercker version`
4. 如果上述命令失败，您可能还需要添加`/usr/local/bin`到您的`$PATH`变量： `echo 'PATH=/usr/local/bin:$PATH' >> ~/.bash_profile`
5. 导航到 fork 的根目录： `cd /path/of/your/getting-started-golang` 此时一个`wercker.yml`文件应该已经存在：

```javascript
box:
id: golang
ports:
    - "5000"

dev:
steps:
- internal/watch:
    code: |
      go build ./...
      ./source
    reload: true

# Build definition
build:
    # The steps that will be executed on build
    steps:

    # golint step
        - wercker/golint

    # Build the project
        - script:
            name: go build
            code: |
              go build ./...

# Test the project
- script:
    name: go test
    code: |
      go test ./...
```

此`yml`文件中只定义了两个管道：`dev`和`build`。请注意，在此示例中，暴露的端口为`5000`。

​    **·** `dev`：这种特殊类型的管道只能在本地使用，并且仅用于应用程序测试。

​    **·** `internal/watch`：观察代码的更改。如果发生任何事件，它会再次触发应用程序的构建（`reload: true`）。这对调试进程来说很有用。

​    **·** `build`：这通常是构建应用程序的第一步。但现在您可以使用它来检查代码中的`build`步骤（`wercker/golint`）是否存在错误。

# 怎样添加应用程序 Wercker？

顾名思义，Wercker 应用程序对应于您的每个项目。由于使用 Wercker 的版本控制方案，您可以将每个应用程序视为特定的工作仓库。

\1. 注册三个 fork 仓库。单击右上角的 “**+”** 按钮：

![img](https://ask.qcloudimg.com/draft/1206219/zyn14od806.jpg?imageView2/2/w/1620)

\2. 选择第一个示例仓库 ** / jClocksGMT**：

![img](https://ask.qcloudimg.com/draft/1206219/k7drk2balh.png?imageView2/2/w/1620)

\3. 配置仓库的访问权限。如果项目不使用子模块，“recommended（推荐的）” 选项通常是最佳选择。

![img](https://ask.qcloudimg.com/draft/1206219/f90aobi4rj.jpg?imageView2/2/w/1620)

\4. 选择您的应用程序是私有的（“private”，默认选项）还是公有的（“public”）。将示例标记为公有，然后单击 **完成（Finish）** 按钮。

此时出现一个问候消息，表明您已准备好开始构建应用程序。它提供了启动向导来帮助您创建应用程序`wercker.yml`文件，但这不是必需的，因为您已经在上一节中已经这样做了。

![img](https://ask.qcloudimg.com/draft/1206219/ufsgtdnk67.jpg?imageView2/2/w/1620)

\5. 对其他两个示例项目重复相同的过程。

## 配置应用程序

### jClocks 示例

与配置文件类似，您需要设置几个环境变量。

\1. 对于第一个示例，您需要一个 SSH 密钥对来与您的 Linode 进行通信。单击 “**环境（Environment）**“ 选项卡：

![img](https://ask.qcloudimg.com/draft/1206219/xnbkr58bv4.jpg?imageView2/2/w/1620)

\2. 单击 “生成 SSH 密钥（Generate new SSH key）”。接下来弹出的窗口将会询问密钥名称（使用与`wercker.yml`文件中相同的名称，文件: `linode`）。选择 **4096位** 加密：

![img](https://ask.qcloudimg.com/draft/1206219/3r9w6g90qp.jpg?imageView2/2/w/1620)

 这步操作将生成密钥对：`linode_PUBLIC`和`linode_PRIVATE`。后缀会自动添加，配置文件中就不需要自己加上了。

\3. 将`linode_PUBLIC`密钥复制到您的 Linode 中。如果终端应用程序支持复制和粘贴，则可以使用 **CTL-C** 和 **CTL-V** 将文本从 Wercker 仪表板复制到 Linode 的`~/.ssh/authorized_keys`中。如果不支持，您可以将密钥复制到本地计算机，再将其复制到远程服务器： `cat ~/.ssh/jclock.pub | ssh root@ "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"`

\4. 您的第一个示例已准备好部署：应用程序在 Wercker 上配置，您的本地仓库包含了`wercker.yml`文件，它解释了要执行的步骤。想要触发自动化操作，请提交一些更改。要查看相关进度，请单击 Wercker 中的 “**运行（Runs）**” 选项卡。在 jClocksGMT 这一 fork 的根目录运行以下命令： `git add . && git commit -m "initial commit" && git push origin master`

\5. 会有动效显示出每个步骤的进度，并允许您调试任何问题。下面是一个构建失败的情况：

![img](https://ask.qcloudimg.com/draft/1206219/h1lylje6fz.jpg?imageView2/2/w/1620)

提示 “远程 Linode 上的代码更新出现失败。”，单击构建管道以获取详细信息：

![img](https://ask.qcloudimg.com/draft/1206219/vmg8xcoell.jpg?imageView2/2/w/1620)

\6. 这表明该过程出现失败的步骤为 “更新远程 Linode 上的代码”。其原因是仓库起初并没有克隆在远程 Linode 上。连接到您的 Linode 并在适当的位置克隆存储库，然后返回到 Wercker 仪表板并单击 “重试（Retry）” 按钮：

![img](https://ask.qcloudimg.com/draft/1206219/146tzqt4v7.jpg?imageView2/2/w/1620)

 这次就应该运行成功了，并且您的远程 Linode 仓将被更新。此基本示例可用于许多情景。如果您正在托管静态网站，则可以将 Wercker 配置为 ”每当提交更改时，相应地更新远程服务器“。

### hello.go 示例

单击 Wercker 仪表板中的 “**工作流程（Workflows）**” 选项卡。编辑器将展示一个由 Wercker 自动创建的单独管道`build`。此示例需要您必须手动创建的其他管道。

\1. 单击 ”**添加新管道（Add new pipeline）**“：

![img](https://ask.qcloudimg.com/draft/1206219/7jg36dfsh6.jpg?imageView2/2/w/1620)

\2. 填写如下字段：

​    **· 名称：**您可以使用任意名称来描述您的管道，对于此示例我们将使用`deploy-docker`与`deploy-linode`。

​    **· YML 管道名称：**这是在`wercker.yml`文件中声明的名称。分别填写`deploy`和`linode`。

​    **· 钩类型（Hook type）：**使用默认行为，链接（Chain）这条管道到另一个管道。如果要在每次提交推送时并行运行不同的管道，则可以选择 **Git push**。

\3. 配置管道后，您可以链接它们。单击 ”+“ 到`build`管道右侧：

![img](https://ask.qcloudimg.com/draft/1206219/onlbzzny04.jpg?imageView2/2/w/1620)

您可以选择定义特定分支（或多个分支）以触发管道。默认情况下，Wercker 将监视所有分支，如果有任何提交出现，就会开始执行步骤，这就是我们的示例。在下拉列表中选择 ”**deploy-docker**“，然后再单击 “**添加（Add）**”。

\4. 重复此过程以在该管道之后链接`deploy-linode`管道。最终结果如下：

![img](https://ask.qcloudimg.com/draft/1206219/fy814g3ihd.jpg?imageView2/2/w/1620)

\5. 接下来，您需要定义环境变量，但这次您将在每个管道内分别执行，而不是进行全局操作。在 “工作流（Workflows）” 选项卡上，单击屏幕底部的 ”**deploy-docker**“ 管道。此处您可以创建变量。这个例子的`wercker.yml`中有两个变量必须在这里定义：即`DOCKER_USERNAME`和`DOCKER_PASSWORD`。创建它们并将密码标记为 ”**受保护的（protected）**“。

\6. 选择 **deploy-linode** 管道并创建 SSH 密钥对，与上一示例类似。请记住将公钥复制到远程服务器。

\7. 通过提交到本地 fork 以测试您的工作流： `git add . && git commit -m "initial commit" && git push origin master` 您最终得到类似如下结果：

![img](https://ask.qcloudimg.com/draft/1206219/6y7qvlveb0.jpg?imageView2/2/w/1620)

\8. 通过远程登录并运行`docker images`以测试远程服务器上的应用程序：

![img](https://ask.qcloudimg.com/draft/1206219/k5y25wb2uz.jpg?imageView2/2/w/1620)

此时仅有一个标签为`current`的映像。

\9. 通过`docker run`命令运行应用： `docker run /myapp:current`

\10. 如果要进一步测试自动化步骤，则在`/example`文件夹内编辑`hello.go`。在消息中添加一些文字。提交更改并等待 Wercker 自动化运行。

\11. 再次运行您的应用程序，此时您应该能看到修改后的消息：

![img](https://ask.qcloudimg.com/draft/1206219/wisgkk98na.jpg?imageView2/2/w/1620)

### Wercker CLI 示例

最后一个示例演示了 Wercker CLI 相关内容。

\1. 导航到`getting-started-golang`这一 fork 的根目录。

\2. 通过运行下列命令启动 Wercker： `wercker build`

![img](https://ask.qcloudimg.com/draft/1206219/36wgi4d8jn.jpg?imageView2/2/w/1620)

此处的输出应类似于您在 Wercker 仪表板上所看到的日志。不同之处在于，您可以在本地检查每个步骤，并在流程中更早地检测到错误情况。Wercker CLI 重复 SaaS 的行为：它下载指定的图像，构建，测试并显示错误。由于 CLI 是一种旨在促进本地测试更加便利的开发工具，因此您将无法远程部署最终结果。

\3. 使用 Go 构建应用程序： `go build` 该应用程序（即`getting-started-golang`）构建在根目录中。

\4. 运行程序： `./getting-started-golang`

\5. 导航到`localhost:5000/cities.json`。您应该会看到显示的城市列表：

![img](https://ask.qcloudimg.com/draft/1206219/3j4477c03f.png?imageView2/2/w/1620)

\6. 使用 **CTRL + C** 关闭程序。运行`wercker dev`： `wercker dev --expose-ports`

![img](https://ask.qcloudimg.com/draft/1206219/gl47bw8mjr.jpg?imageView2/2/w/1620)

此命令会启动`dev`管道中的自动构建功能。它在 Docker 容器中构建应用程序并从那里提供服务。如果您对应用程序进行任何更改，容器将重建以反映这一更改。

\7. 在文本编辑器中打开`main.go`文件，并在城市列表中添加一个条目。刷新浏览器，此时您应该能看到更新的列表。

# 下一步计划

开发者在使用 Wercker 时具有无限的可能性：

- 您可以指定 ”局部框（local boxes）“，这意味着您可以根据管道的目标而使用专门的图像。DockerHub 中有许多图像可以用于此目的。
- 您不仅限于 ”链接（Chain）“ 工作流，您可以并行启动管道（尽可能多地）并在必要时才进行链接。如果您需要构建需要很长编译时间的复杂应用程序，这将会非常有用。您可以在与其他任务并行的早期启动编译管道。您还可以将应用程序划分为多个管道，以减少每个进程的时间并隔离问题。
- Wercker 是无关于语言、流程、平台的。换句话说 —— 只有你想不到，没有 Wercker 工具做不到。

# 更多信息

有关此主题的其他信息，您可能需要参考以下资源。此处提供这些资源是希望它们能够有所帮助，但请您注意，我们无法保证外部托管材料的准确性或及时性