# 第7章 运行您的工作流

在本章中，我将向您展示运行工作流的不同选项。 我们将研究托管和自托管执行器，并将解释如何利用不同的托管选项处理混合云场景或硬件循环测试。 我还将向您展示如何设置、管理和扩展自托管执行器，并向您展示如何进行监视和故障排除。

以下是本章将涵盖的核心主题：

- 托管执行器 
- 自托管执行器 
- 使用运行器组管理访问 
- 使用标签 
- 扩展自托管执行器 
- 监视和故障排除

## 托管执行器

在前面的章节我们已经使用过托管执行器(hosted runners)。托管执行器是Github托管的用于运行工作流的虚拟机。这种执行器适用于Linux、Windows和macOS操作系统。

### 隔离和隐私

工作流中的每个作业都在一个全新的虚拟机实例中执行，并且完全隔离。您拥有完整的管理员访问权限（在Linux上为无密码sudo），而Windows机器上的用户账户控制（UAC）已被禁用。这意味着您可以安装工作流中可能需要的任何工具（这仅会增加构建时间的成本）。

执行器还可以访问用户界面（UI）元素。这使您可以在执行器内执行UI测试，例如Selenium，而无需通过另一个虚拟机进行操作。

### 硬件

Github托管的Linux和Windows系统的执行器运行在Standard_DS2_v2虚拟机的Microsoft Azure中。Windows和Linux虚拟机的硬件要求如下：

- 2核CPU
- 7GB RAM
- 14GB SSD 硬盘空间

MacOS版本的runner运行在Github的macOS云上，其硬件要求如下：

- 3核CPU
- 14GB RAM
- 14GB SSD硬盘空间

### 软件

在表7.1中，您可以看到当前可用的镜像列表：

![](tab7-1.png)

| 虚拟环境             | YAML工作流标签                 | 备注                                                         |
| -------------------- | ------------------------------ | ------------------------------------------------------------ |
| Windows Server 2022  | windows-2022                   | 当前仍在beta阶段。windows-latest标签目前使用Windows Server 2019的runner镜像，当2022结束beta阶段后，将切换为Windows Server 2022 |
| Windows Server 2019  | windows-latest或者windows-2019 | windows-latest标签当前指向该镜像                             |
| Windows Server 2016  | windows-2016                   |                                                              |
| Ubuntu 20.04         | ubuntu-latest或者ubuntu-20.04  | ubuntu-latest标签当前指向该镜像                              |
| Ubuntu 18.04         | ubuntu-18.04                   |                                                              |
| Ubuntu 16.04         | ubuntu-16.04                   | 已弃用，仅限制已存在的用户使用。用户应迁移到Ubuntu 20.04     |
| macOS Big Sur 11     | macos-11                       |                                                              |
| macOS Catalina 10.15 | macos-latest or macos-10.15    | windows-latest标签当前使用macOS 10.15的runner镜像            |

表7.1 托管执行器当前可用的镜像

你可以在该网址中找到最新列表和所有包含的软件：https://github.com/actions/virtual-enviroments

如果你想请求一个新的工具作为默认工具安装，你也可以在该仓库提起issue。这个仓库还包括了执行器所有重大软件更新的公告。你还可以使用Gihub仓库的“关注”功能来获取新版本创建的消息。

### 网络

托管执行器使用的IP地址在随时变化。你可以通过Github的API来获取当前IP列表：

```shell
cur \
 -H “Accept: application/vnd.github.v3+json” \
 https://api.github.com/meta
```

更多信息可以查看网址： https://docs.github.com/en/rest/reference/meta#get-github-meta-information

通过这些信息，你可以通过一个允许列表来阻止来自因特网的对你内部资源的访问。但是要记得，所有人都可以使用托管runners和执行代码。阻止其他IP地址并不会使得你的资源安全。不要反对？这些 IP 地址的内部系统，因为这些 IP 地址并不是以你可以信任的方式从公共互联网访问的！这意味着必须对系统进行修补，并在适当的位置进行安全身份验证。如果情况不是这样，你必须使用自托管的runner。

通过这些信息，您可以通过一个允许列表来阻止从互联网访问您的内部资源。但请记住，任何人都可以使用托管执行器并执行代码！阻止其他IP地址并不能使您的资源安全。不要将这些 IP 地址和内部系统对立起来，因为这些 IP 地址并不是以你可以信任的方式从公共互联网访问的！这意味着必须对系统进行补丁，并在适当的位置进行安全身份验证。如果情况不是这样，你必须使用自托管执行器。

> 提示：
>
> 如果对Github组织或者公司账户使用IP地址允许列表，则不能使用Github托管执行器，而只能使用自托管执行器。

### 价格

托管执行器对于公共仓库的使用是免费的。根据您的GitHub版本，您将拥有指定的存储量和每月免费构建分钟数。

![](tab7-2.png)

Github edition Github版本

Storage 存储空间

Minutes 分钟数

Max concurrent jobs 最大并发任务数量

表7-2 不同Github版本所包含的存储量和构建分钟数

如果您通过Microsoft企业协议购买了GitHub Enterprise，则可以将Azure订阅ID连接到您的GitHub Enterprise帐户。这使您可以支付额外的GitHub Actions使用费用，以补充您的GitHub版本中包含的内容。

在Windows和macOS执行器上运行的作业比Linux消耗更多的构建分钟数！ Windows消耗的时间因子是2，而macOS消耗的时间因子是10。这意味着使用1,000分钟Windows系统将消耗您帐户中包含的2,000分钟，而使用1,000分钟macOS系统将消耗您帐户中包含的10,000分钟。

这是因为构建分钟数更昂贵。您可以支付额外的分钟数，以补充您的GitHub版本中包含的分钟数。这些是每个操作系统的构建分钟费用：

- Linux:$0.008
- macOS:$0.08
- Windows:$0.016

> 提示：
>
> 您应该尽可能多地在工作流中使用 Linux，并将 macOS 和 Windows 减少到最低限度，以降低构建成本。另外，Linux 也有最好的启动性能。

额外存储的成本对所有的执行器来说是一样的——0.25美元每GB。

如果您是按月结账的客户，您的帐户将有一个默认的支出限额：0美元。这可以防止使用额外的分钟或存储。如果你用发票支付，你的账户将默认有无限的支出限制。

如果您设置的消费限额高于0美元，您的账户将被计费任何额外的分钟或存储，直到达到消费限额。

## 自托管执行器

如果您需要比GitHub 托管执行器支持的硬件、操作系统、软件和网络访问更多的控制，那么您可以使用自托管执行器。自托管执行器可以安装在物理机器、虚拟机或容器中。它们可以在本地或任何公共云环境中运行。

自托管运行程序允许从其他构建环境轻松迁移。如果您已经有了自动构建，那么只需在机器上安装运行程序，您的代码就可以构建了。但如果您的构建机器仍然是手动维护的老式机器，或者机器位置远离开发人员，那么这不是一个永久性的解决方案。请记住，构建和托管一个动态扩展环境需要专业知识，并且需要花费资金，无论是托管在云端还是本地。因此，如果您可以使用托管执行器，那么它总是更容易的选择。但是，如果您需要自托管的解决方案，请确保使其具有弹性可扩展性。

> 提示：
>
> 拥有自己的执行器可以让您在GitHub Enterprise Cloud内安全地构建和部署本地环境。这样，您就可以使用混合模式运行GitHub，即您可以在云端使用GitHub Enterprise，并使用托管执行器进行基本的自动化和部署到云环境，而使用自托管执行器来构建或部署托管在本地的应用程序。这比自行运行GitHub Enterprise Server和所有构建和部署的构建环境更加便宜和简单。

如果您依赖硬件来测试软件，例如使用硬件在环测试(HIT)，则必须使用自托管执行器。这是因为没有办法将硬件连接到GitHub托管执行器。

### 执行器软件

运行器是开源的，可以在https://github.com/actions/runner上找到。它支持Linux、macOS和Windows上的x64处理器架构。它也支持ARM64和ARM32架构，但仅在Linux上。该运行器支持许多操作系统，包括Ubuntu、Red Hat Enterprise Linux 7或更高版本、Debian 9或更高版本、Windows 7/8/10和Windows Server、macOS 10.13或更高版本等。有关完整列表，请参阅文档：[https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners#supported-architectures-and-operating-systems-for-self-hosted-runners。](#supported-architectures-and-operating-systems-for-self-hosted-runners。)

运行器会自动更新，因此您不需要管理更新。

### 运行器和GitHub之间的通信

运行器软件使用HTTPS长轮询在端口443上从GitHub轮询，使用出站连接。它打开一个50秒的连接，如果没有收到响应，则会超时。

您必须确保机器具有网络访问以下URL：

```
github.com
api.github.com
*.actions.githubusercontent.com
github-releases.githubusercontent.com
github-registry-files.githubusercontent.com
codeload.github.com
*.pkg.github.com
pkg-cache.githubusercontent.com
pkg-containers.githubusercontent.com
pkg-containers-az.githubusercontent.com
*.blob.core.windows.net
```

您不必在防火墙上打开任何入站端口。所有通信都通过客户端运行。如果您对 GitHub 组织或企业使用 IP 地址允许列表，您必须将自托管执行器的 IP 地址范围添加到该允许列表。

### 在代理服务器上使用自托管执行器

如果您需要在代理服务器上运行自托管执行器，可以这样做。但请注意，这可能会导致很多问题。执行者本身可以正常通信，但是，包管理、容器注册表以及执行器执行需要访问资源的所有内容都会产生开销。请尽可能避免使用代理服务器。如果不得不在代理服务器上运行工作流，您可以使用以下环境变量配置执行器：

- https_proxy：这包括 HTTPS（端口443）流量的代理 URL。您还可以包括基本身份验证（例如 https://user:password@proxy.local）
- http_proxy：这包括 HTTP（端口80）流量的代理 URL。您还可以包括基本身份验证（例如[http://user:password@proxy.local) 
- no_proxy：这包括应该绕过代理服务器的逗号分隔的主机列表。

如果更改环境变量，必须重新启动执行器才能使更改生效。

另一种代替使用环境变量的方法是使用.env文件。在执行器的应用程序文件夹中保存一个名为.env的文件，之后语法与环境变量相同：

```
https_proxy=http://proxy.local:8081
no_proxy=example.com,myserver.local:443
```

接下来，让我们看一下如何将自托管执行器添加到GitHub。

### 添加自托管运行器到 GitHub

您可以在 GitHub 的不同层面添加执行器：仓库、组织或企业。如果在仓库级别添加执行器，它们仅限于该单个仓库。组织级别的执行器可以处理组织中多个仓库的作业，企业级别的执行器可以分配给企业中的多个组织。 安装执行器并在您的 GitHub 实例中注册很容易。只需转到您希望添加执行器的级别的设置|操作|执行器。然后选择操作系统和处理器架构（参见图 7.1）：

![](fig7-1.png)

图7.1-安装自托管执行器

这个脚本为您生成了以下内容：

- 下载和解压程序
- 使用相应配置程序
- 启动程序

 脚本的第一部分始终创建一个名为 actions-runner 的文件夹，然后将工作目录更改为该文件夹

```
$ mkdir actions-runner && cd actions-runner
```

在 Linux 和 macOS 上，使用 curl 命令下载最新的 runner 包；在 Windows 上使用 Invoke-WebRequest。 

```
# Linux and macOS:
$ curl -o actions-runner-<ver>.tar.gz -L https://github.com/
actions/runner/releases/download/<ver>/actions-runner-<ver>.
tar.gz
# Windows:
$ Invoke-WebRequest -Uri https://github.com/actions/runner/
releases/download/<ver>/actions-runner-<ver>.zip -OutFile 
actions-runner-<ver>.zip
```

为确保安全，验证下载的包的哈希值，以确保包没有被破坏： 

```
# Linux and macOS:
$ echo "<hash> actions-runner-<ver>.tar.gz" | shasum -a 256 -c
# Windows:
$ if((Get-FileHash -Path actions-runner-<ver>.zip -Algorithm 
SHA256).Hash.ToUpper() -ne '<hash>'.ToUpper()){ throw 'Computed 
checksum did not match' }
```

然后，执行器从 ZIP/TAR 文件中被提取出来：

```
# Linux and macOS:
$ tar xzf ./actions-runner-<ver>.tar.gz
# Windows:
$ Add-Type -AssemblyName System.IO.Compression.FileSystem ; 
[System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/
actions-runner-<ver>.zip", "$PWD")
```

配置是使用 config.sh / config.cmd 脚本完成的，URL 和令牌由 GitHub 自动为您创建：

```
# Linux and macOS:
$ ./config.sh --url https://github.com/org --token token
# Widows:
$ ./config.cmd --url https://github.com/org --token token
```

配置要求选择执行器组（默认为 Default 组）、执行器名称（默认为机器名称）以及其他标签。默认标签用于描述自托管状态，操作系统和处理器架构（例如分别是自托管，Linux 和 X64）。默认的工作文件夹是 _work，不应更改。在 Windows 上，您还可以选择将 action runner 作为服务运行。在 Linux 和 macOS 上，必须在配置后使用另一个脚本安装服务：

```
$ sudo ./svc.sh install
$ sudo ./svc.sh start
```

如果你不想以服务方式运行执行器，你可以使用 run 脚本交互式运行它：

```
$ ./run.sh
$ ./run.cmd
```

如果执行器正在运行，您可以在“设置|操作|执行器”中看到其状态和标签（见图7.2）：

![](fig7-2.png)

图7.2-自托管执行器及其标签和状态

现在我们来学习如何从GitHub中删除这些自托管执行器。

###  删除自托管执行器

如果您想重新配置或从GitHub中删除执行器，您必须使用带有“删除”选项的config脚本。 如果您通过单击其名称打开执行器的详细信息，您会看到一个“删除”按钮（请参见图7.2）。 如果您单击此按钮，它将为您生成脚本和令牌。

![](fig7-3.png)

图7.3-执行器详细

不同操作系统的脚本如下：

```
# Linux and macOS
./config.sh remove --token <token>
# Windows
./config.cmd remove --token <token>
```

永远要在销毁机器之前先删除运行器！如果您忘记这样做，您仍然可以在“删除”对话框中使用“强制删除此运行器”按钮。但是，这应该仅作为最迫不得已的方式。

## 管理访问权限

如果在组织或企业级别注册您的执行器，则使用执行器组来控制对自托管执行器的访问。企业管理员可以配置访问策略，以控制企业中的哪些组织有访问执行器组的权限，而组织管理员可以配置访问策略，以控制组织中的哪些仓库有访问执行器组的权限。每个企业和每个组织都有一个默认的执行器组，名为默认，不能被删除。

> 提示：
>
> 一个执行器只能在一个执行器组中。

在进行管理访问时，打开企业级别的政策或组织级别的设置，并在菜单中找到 操作|执行器组。 在此处，您可以创建新的执行器组或单击现有组件以调整其访问设置。 根据您的级别是企业还是组织，您可以允许访问特定组织或仓库（请参见图 7.3）。

![](fig7-4.png)

图7.4-执行器组设置

> 警告
>
> 默认情况下，禁用访问公共仓库。请不要更改该设置！不应该在公共存储库中使用自托管执行器！这会带来风险，因为Forks可能会在您的执行器上执行恶意代码。如果您需要公共仓库的自托管执行器，请确保使用没有访问您内部资源的临时和加固执行器。一种可能的情况是，您需要开源项目的特殊工具，并且它在托管执行器上安装时间过长。但这是很少见的情况，您应该尽量避免它们。
>

当你注册一个新的 runner 时，你需要为它输入 runner 组的名称。你也可以在配置脚本中作为参数传递：

```
 $./config.sh --runnergroup <group> 
```

现在我们已经学会了如何使用执行器组管理访问权限，接下来我们将学习使用标签。

## 使用标签

GitHub Actions 通过搜索正确的标签来将工作流匹配到你的执行器。标签在您注册执行器时被应用。您也可以在配置脚本中作为参数传递：

```
$./config.sh --labels self-hosted,x64,linux
```

您可以在执行器的详情中稍后修改标签，并点击标签旁边的齿轮图标创建新标签（参见图 7.4）。

![](fig7-5.png)

图7.5-为执行器创建新标签

如果您的工作流程有特定的需求，您可以为它们创建自定义标签。自定义标签的一个例子可能是为工具（如matLab）添加标签，或者添加必需的GPU访问标签。所有自托管执行器默认都有自托管标签。要在工作流中使用执行器，您需要在标签形式中指定需求：

```
runs-on：[self-hosted，linux，X64，matlab，gpu]
```

这样，您的工作流程就可以找到满足必要需求的相应运行器。 

## 扩展自托管执行器

在现有构建机器上安装action runner可以轻松迁移到 GitHub。但这不是长期解决方案！如果您无法使用托管执行器，则应该自己构建弹性扩展的构建环境。

### 瞬时执行器

如果您为构建机器或容器构建了弹性扩展解决方案，您应该使用瞬时执行器。这意味着您使用来自空白镜像的虚拟机或 Docker 镜像并安装临时运行程序。每次运行后，一切都会被清除。这种情况下，不推荐使用弹性扩展的持久性执行器！

要配置执行器为瞬时的，您需要向配置脚本传递以下参数：

```
$ ./config.sh --ephemeral
```

### 使用 GitHub Webhooks扩展

您可以使用 GitHub Webhooks 调整虚拟环境的配置。如果有新工作流到达队列，则 webhook的workflow_job会调用queued操作。您可以使用此事件创建新的构建机器并将其添加到机器池中。如果工作流运行完成，则 workflow_job 会调用completed操作。您可以使用此事件进行清理和销毁机器。 有关更多信息，请参阅文档 [https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#workflow_job](#workflow_job) 。 

### 现有解决方案

在 Kubernetes、AWS EC2 或 OpenShift 中构建弹性虚拟构建环境超出了本书的范围。GitHub 本身并不提供此解决方案，但如果您希望利用它们，则有很多开源解决方案可以节省您的时间和精力。 Johannes Nicolai (@jonico) 对所有解决方案进行了整理。您可以在 https://github.com/jonico/awesome-runners 中找到相关资料。矩阵在 GitHub 页面中更容易阅读，因此您可能更喜欢访问 https://jonico.github.io/awesome-runners。矩阵根据目标平台、是否具有 GitHub Enterprise 支持、自动缩放功能、清理因素等标准进行比较。

> 提示：
>
> 请记住，使用自定义镜像构建和运行可扩展的构建环境需要大量的时间和精力，这些时间和精力可以用于其他事情。使用托管执行器是更便宜和更可持续的解决方案。请确保您真的需要在自己的平台上进行这样的投资。通常，还有其他选择搭载自己的执行器，例如将自己的Docker镜像带入GitHub Actions或使用机器人自动部署到本地资源。

 

## 监控和故障排除

如果您有自托管执行器的问题，以下内容可以帮助您进行故障排除。

### 检查运行者的状态

您可以在“设置|操作|执行器”下检查执行器的状态。执行器的状态可以是空闲、活动或离线。如果执行器状态是离线，则机器可能已经关闭或未连接到网络，或者自托管执行器应用程序可能未在机器上运行。

### 查看应用程序日志文件

日志文件保存在运行者的根目录中的“_diag”文件夹中。您可以在其中中查看运行者应用程序日志文件。应用程序日志文件名以Runner_开头，并追加UTC时间戳：

```
Runner_20210927-065249-utc.log 
```

### 查看作业日志文件

作业日志文件也位于_diag中。每个作业都有自己的日志。应用程序日志文件名以Worker_开头，也有UTC时间戳：

```
Worker_20210927-101349-utc.log
```

### 检查服务状态

如果您的执行器以服务的形式运行，您可以根据您的操作系统检查服务状态。

#### Linux

在 Linux 上，您可以从 runner 文件夹中的 .service 文件中获取您的服务的名称。使用 journalctl 工具监视 runner 服务的实时活动：

```
$ sudo journalctl -u $(cat ~/actions-runner/.service) -f 
```

您可以在 /etc/systemd/systemd/ 下检查和自定义服务的配置： 

```
$ cat /etc/systemd/system/$(cat ~/actions-runner/.service) 
```

#### macOS

在 macOS 上，您可以使用 svc.sh 脚本检查服务的状态： 

```
$ ./svc.sh status 
```

上述脚本的输出包含服务名称和进程 ID。

要检查服务配置，请找到以下位置的文件： 

```
$ cat /Users/<user_name>/Library/LaunchAgents/<service_name> 
```

#### Windows

在 Windows 上，您可以使用 PowerShell 检索有关您的服务的信息： 

```
$ Get-Service "action*" 
```

使用 EventLog 监视您的服务的最近活动：

```
Get-EventLog -LogName Application -Source ActionsRunnerService 
```

### 监控执行器更新过程

执行器会自动更新。如果更新失败，执行器将无法运行工作流。您可以在_diag 目录中的 Runner_*日志文件中检查它的更新活动。

## 案例研究

Tailwind Gears的两个试点团队在新平台上开始了他们的第一个Sprint周期。 他们首先要自动化的是构建过程，以便在合并前对所有pull请求进行构建。 Tailwind Gears尽量多地使用GitHub托管的执行器。 大部分软件都能很好地构建。 然而，一些使用旧版编译器的C代码以及在当前构建机器上安装的其他依赖项存在问题。 该代码目前在两个由开发人员自行维护的本地Jenkins服务器上构建。 这些服务器也连接到用于硬件在环测试的硬件上。 为了方便过渡，在这些机器上安装了自托管执行器，并且构建正常运行。 IT部门本来就想摆脱本地服务器，因此他们与GitHub合作伙伴一起构建了一个弹性、可扩展、基于容器的解决方案，可以运行自定义镜像，并可以访问附加硬件。

## 总结

在本章中，您了解了运行工作流的两种托管选项：

- GitHub托管的执行器
- 自托管的执行器 

我们解释了自托管执行器如何使您能够在混合云场景中运行GitHub。您了解了如何设置自托管执行器以及如何找到帮助您构建自己的弹性可扩展构建环境的信息。

 在下一章中，您将了解如何使用GitHub Packages管理代码依赖关系。

## 进一步阅读

有关本章主题的更多信息，您可以参考以下资源：

- 使用GitHub托管的执行器：https://docs.github.com/en/actions/using-github-hosted-runners 
- 自托管执行器：https://docs.github.com/en/actions/hosting-your-own-runners
- awesome-runners-大量比较矩阵中优秀的自托管GitHub动作执行器解决方案的列表：https://jonico.github.io/awesome-runners
