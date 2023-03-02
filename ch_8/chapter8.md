# 第八章  使用GitHub管理依赖

使用包注册表来管理您的依赖项应该是毋庸置疑的。如果您在编写.NET，可以使用 NuGet，如果是在编写 JavaScript，也许可以用npm，如果您使用的是Java，则可以使用 Maven 或 Gradle。然而，我碰到许多团队仍然使用文件系统或Git子模块在多个代码库中重用代码文件，或者构建程序集并将其存储在源代码管理中。将共享代码迁移到具有语义版本控制的包十分简单且成本低廉，还能提高代码的质量和可发现性。



本章节将主要介绍如何使用GitHub Packages来管理您的内部依赖，就像管理软件供应链一样。主要内容如下：

- GitHub包
- 将npm包与Actions结合使用
- 将Docker和包结合使用
- Apache Maven, Gradle, NuGet, and RubyGems包

语义版本控制

语义版本控制是指定软件版本号的正式约定，它由不同的部分组成，且各部分含义相异。语义版本号的示例为1.0.0或1.5.99-beta。格式如下:

<主要>.<次要>.<补丁>-<预版本>

主要版本:如果版本不兼容以前的版本并且有重大更改，将增加该数字标识符。必须谨慎更新一个新的主要版本!主要版本设置为0用于初始开发。

次要版本:如果添加了新功能，将增加该数字标识符。该版本兼容以前的版本，如果您需要新功能，可以在不损坏任何内容的情况下进行更新。

补丁:如果您发布向后兼容的错误修复，将增加该数字标识符。用户应始终安装最新补丁。

预版本:即使用连字符附加的文本标识符。该标识符只能使用 ASCII 字母、数字字符和连字符([0-9A-Za-z-])。文本越长，预版本越小(即-alpha<-beta<-re).预发布版本总是比普通版本小(1.0.0-alpha < 1.0.0)。

请参阅 https://semver.org/ 以查看完整的规范。

使用包并不自动意味着您在使用松散耦合的体系结构，在大多数情况下，包仍然是硬依赖。这取决于您如何使用包来真正地解耦发布节奏。

## GitHub包

GitHub包是一个用来托管和管理包、容器以及其他依赖的平台。

您可以将GitHub包与GitHub Actions、GitHub API 和web挂钩集成在一起，以创建一个端对端的工作流来发布和使用代码。

GitHub包当前支持以下注册表：

- 支持**Docker** 和**OCI**镜像的**容器**注册表
- 适用于使用npm的JavaScript的**npm**注册表(package.json)
- 适用于.NET的**NuGet**注册表(nupRg)
- 适用于Java的**Apache** **Maven**注册表(pom.xml)
- 适用于Java的**Gradle**注册表 (build.gradle)
- 适用于Ruby的**RubyGems**注册表(Gemfile)

### 价格

对于公共包，GitHub包是免费的，而对于专用包，每个GitHub版本都包含一定数量的存储和数据传输，超出部分将单独收费，并且可以由支出限额进行控制。

按月计费的客户的默认支出限额为0美元，这可以防止额外使用存储或数据传输，使用发票支付的客户具有无限制的默认支出限额。

每种产品包含的存储量和数据传输量如表8.1所示：

![col8-1](./chapter8.assets/col8-1.png)

表8.1 GitHub产品中包的存储量和数据传输量

产品                          存储量           数据传输量（每月）

GitHub Free                   500MB             1GB

GitHub Pro                    2GB               10GB

GitHub Free for organizations 500MB             1GB

GitHub Team                   2GB               10GB

GitHub Enterprise Cloud       50GB              100GB 



由GitHub Action触发时，所有出站数据传输都是免费的，任何来源的入站数据传输也是免费的。

当达到产品的限制容量时，将按以下标准进行收费：

- 存储：每GB 0.25美元
- 数据传输：每GB0.5美元

有关定价的更多信息，请参阅https://docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages.



### 权限和可见性

发布到仓库的包将继承拥有该包的仓库的权限和可见性。目前，只有容器包支持精细权限和访问控制（见图8.1）

![fig8-1](./chapter8.assets/fig8-1.png)

图8.1 — 管理对容器包的访问权限

其他所有类型的包都遵循仓库范围内包的仓库访问权限。在组织级别，包是私有的，所有者具有写入权限，成员具有读取权限。

如果您对容器镜像具有管理者权限，则可以将容器镜像的访问权限设置为私有或公共。公共镜像允许匿名访问而无需身份验证。您还可以授予容器镜像的访问权限，该权限与在组织和仓库级别设置的权限不同。

在组织级别，您可以设置成员允许发布的容器包类型，您也可以查看和恢复已删除的包（如图8.2所示）

![fig8-2](./chapter8.assets/fig8-2.png)

图8.2 — 组织级别的包的权限

对于个人账户拥有的容器镜像，您可以将访问角色授予给任何人。对于组织发布和拥有的容器镜像，您只能向组织中的个人或团队授予访问角色。

有关权限和可见性的更多详情，可参阅https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility.

## 将npm和Actions结合使用

使用GitHub Actions设置发布包的工作流非常容易。您可以使用GITHUB_TOKEN对包管理者的本地客户端进行身份验证。要使用npm进行试用，您可以按照此处的分步说明进行操作：https : //github.com/wulfland/package-demo。

如果您的计算机上安装了npm，则可以使用npm init创建包，否则，需从上述仓库中复制package.json和package-lock.json文件的内容。

发布包的工作流程非常简单，首先设置每次创建新版本时都会触发：

```
on:
  release:
    types: [created]
```

工作流由两个作业组成。第一个作业只使用npm构建和测试包：

```
 build:
   runs-on: ubuntu-latest
   steps:
     - uses: actions/checkout@v2
     - uses: actions/setup-node@v2
     with:
       node-version: 12
     - run: npm ci
     - run: npm test
```

第二个作业将镜像发布到注册表中，这需要编写包和读取内容的权限，使用${{ secrets.GITHUB_TOKEN }}向注册表进行身份验证。

```
publish-gpr:
  needs: build
  runs-on: ubuntu-latest
  permissions:
    packages: write
    contents: read
  steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v2
      with:
        node-version: 12
        registry-url: https://npm.pkg.github.com/
    - run: npm ci
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

工作流程很简单，每当您在GitHub中创建一个新版本，都会将一个新包发布到您的npm注册表中。您可以在**Code | Packages**下找到包的详情与设置（如图8.3所示）。

![fig8-3](./chapter8.assets/fig8-3.png)

图8.3 — 包的详情与设置

然后，您可以通过npm install @<owner-name>/<package-name>在其他项目中使用该包。

> 注意
>
> 请注意包的版本不是标签或者发布版本，而是package.json文件中的版本。如果您在创建第二个版本之前不将其更新，工作流将会失败。

如果您想要自动执行此操作，有一些操作可以提供帮助。您可以使用**NPM-Version**(参阅https://github.com/marketplace/actions/npm-version)在发布前自动设置npm的版本。您可以将版本名称（github.event.release.name)或版本标签(github.event.release.tag_name)设置为包的版本：

```
- name: 'Change NPM version'
  uses: reedyuk/npm-version@1.1.1
  with:
    version: ${{github.event.release.tag_name}}
```

如果您想要一种更灵活的方式来根据标签及分支计算语义版本号，可以使用**GitVersion**(参阅https://gitversion.net/)。**GitVersion**是**GitTools**操作的一部分(参阅https://github.com/marketplace/actions/gittools)

要使**GitVersion**正常运行，必须执行所谓的**浅克隆**，可以通过将fetch-depth参数添加到checkout操作中并将其设置为0：

```
steps:
 - uses: actions/checkout@v2
   with: 
     fetch-depth:0
```

然后，下载**GitVersion**并运行execute操作。如果想要获取语义版本的详细信息，请设置一个id。

```
- name: Install GitVersion
  uses: gittools/actions/gitversion/setup@v0.9.7
  with:
  versionSpec: '5.x'
 
 - name: Determine Version
   id: gitversion
   uses: gittools/actions/gitversion/execute@v0.9.7
 
```

最终计算出的语义版本号存储为环境变量$GITVERSION_SEMVER。例如，您可以将其作为**npm-version**的输入。

> 注意
>
> 请注意，**GitVersion**支持配置文件以了解它应如何计算版本号！请参阅https://gitversion.net/ 获取更多信息。

如果您需要从**GitVersion**中访问详细信息（例如主要版本，次要版本或补丁），您可以将其作为gitversion任务的输出参数，进而访问：

```
 - name: Display GitVersion outputs
   run: |
     echo "Major: ${{ steps.gitversion.outputs.major }}"
```

使用**GitVersion**，您可以扩展工作流，从分支或标签中创建包————而不仅仅是发布版本：

```
on:
 push: 
   tags:
     - 'v*'
   branches:
     - 'release/*'
```

使用自动语义版本控制来构建发布工作流很复杂，并且很大程度上依赖于您所使用的工作流和包管理器。本章应该可以帮助您入门。这些技术还可以应用于**NuGet**、**Maven**或其他任何包管理器。

## 将Docker和包结合使用

GitHub的容器注册表是ghcr.io。容器镜像可以由组织或个人账户所持有，但您可以自定义对它们的访问权限。默认情况下，镜像继承运行工作流的仓库的权限及可见性模型。

如果您想要自己尝试以下，可以在此处找到分步指南：https://github.com/wulfland/container-demo。按照以下步骤理解构建的作用：

1. 新建一个名为container-demo的仓库，添加一个简单的Dockerfile文件（不带扩展名）

```
FROM alpine
CMD ["echo", "Hello World!"]
```

Docker 镜像继承自alpine发行版，输出Hello World! 到您的控制台。如果您是Docker 新手并且想要尝试一下，请将仓库克隆下来，并更改本地仓库根目录中的目录。为容器构建镜像：

```
$ docker build -t container-demo
```

然后运行容器：

```
$ docker run --rm container-demo
```

\- - rm参数的作用是在容器结束后将其自动删除。这里应该在您的控制台中打印Hello World!

2. 现在在.github/workflows/目录下创建一个名为release-container.yml的工作流文件。每次创建新版本时都会触发工作该流：

```
name: Publish Docker image

on:
 release:
 types: [published]
```

注册表和镜像名将被设置为环境变量。我使用仓库名作为镜像名，您也可以在此设置为一个固定名字：

```
env:
 REGISTRY: ghcr.io
 IMAGE_NAME: ${{ github.repository }}
```

该作业需要对包的写入权限，并且需要克隆仓库：

```
jobs:
 build-and-push-image:
 runs-on: ubuntu-latest
 permissions:
   contents: read
   packages: write
 steps:
   - name: Checkout repository
     uses: actions/checkout@v2
```

docker/login-action 使用GITHUB_TOKEN来对工作流进行验证。这是推荐的方式：

```
- name: Log in to the Container registry
 uses: docker/login-action@v1.10.0
 with:
   registry: ${{ env.REGISTRY }}
   username: ${{ github.actor }}
   password: ${{ secrets.GITHUB_TOKEN }}
```

metadata-action从Git上下文中提取元数据，并将标签应用于Docker镜像。每当我们创建一个版本，将会推送一个标签**(refs** /tags / < tag-name >)。该操作将创建一个与Git标签同名的Docker 标签，并为镜像创建一个latest标签。请注意，元数据是作为输出变量传递给下一个步骤！这就是我为这一步设置id的原因：

```
- name: Extract metadata (tags, labels)
  id: meta
  uses: docker/metadata-action@v3.5.0
  with:
    images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
```

build-push-action构建镜像并将其推送到容器注册表中。？？标签(tag)和？？标记(label)是从meta步骤的输出中提取的：

```
- name: Build and push Docker image
  uses: docker/build-push-action@v2.7.0
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

3. 新建一个版本和标签来触发工作流。工作流完成后，您可以在**Code | Packages**下找到包的详情和设置信息（如图8.4所示）。

   ![fig8-4](./chapter8.assets/fig8-4.png)

图8.4 — 容器包的详情和设置

如果您创建新版本，GitHub 将新建一个Docker 镜像并将其添加到注册表中。

4. 您可以从注册表中本地拉取容器并运行：

```
$ docker pull ghcr.io/<user>/container-demo:latest
$ docker n --rm ghcr.io/<user>/container-demo:latest
> Hello World!
```

请注意，如果您的包不是公开的，则在拉取镜像之前必须使用docker login ghcr.io进行身份验证。

容器注册表不失为一个发布软件的好方法，从命令行工具到完整的微服务，您可以将软件及其所有依赖项一起发送给其他人来进行使用。

## **Apache Maven, Gradle, NuGet, and RubyGems** packages

其他包类型与npm和Docker基本一致：如果你了解本机包管理器，它们真的很容易使用。对于每一个类型我只会进行简短的介绍。

### Apache Maven管理的Java包

对于用**Maven**管理的**java**包，您只需将包注册表添加到pom.xml文件中：

```
<distributionManagement>
 <repository>
   <id>github</id>
   <name>GitHub Packages</name>
   <url>https://maven.pkg.github.com/user/repo</url>
 </repository>
</distributionManagement>
```

然后，您可以使用GITHUB TOKEN在工作流中发布包：

```
- name: Publish package
  run: mvn --batch-mode deploy
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

如果想要从您的开发机中检索包，必须使用具有read: packages范围的**个人访问令牌（PAT）**。您可以在**Settings | DeveloperSettings | Personal access tokens**下生成一个新令牌，并添加您的用户和PAT至-/.m2/ settings.xml文件中。

更多详细信息，请参阅https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry

### Gradle

在**Gradle**中，您必须将注册表添加到. gradle文件中。您可以从环境变量中读取用户名和访问令牌：

```
repositories {
 maven {
   name = "GitHubPackages"
   url = "https://maven.pkg.github.com/user/repo"
   credentials {
     username = System.getenv("GITHUB_ACTOR")
     password = System.getenv("GITHUB_TOKEN")
 }
 }
}
```

在工作流中，您可以使用gradle publish进行发布

```
- name: Publish package
  run: gradle publish
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

更多详细信息，请参阅https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry。

### RubyGems

如果您想在仓库中为.gemspec文件构建和发布所有gem，可以使用GitHub上的marketplace中的操作：

```
- name: Build and publish gems got .gemspec files
  uses: jstastny/publish-gem-to-github@master
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    owner: OWNER
```

要使用包，您至少需要RubyGems 2.4.1和bundler 1.6.4的环境。修改～/. gemrc文件，并通过提供用户名和个人访问令牌来添加注册表，并将其作为源文件来安装包：

```
---
:backtrace: false
:bulk_threshold: 1000
:sources:
- https://rubygems.org/
- https://USERNAME:TOKEN@rubygems.pkg.github.com/OWNER/
:update_sources: true
:verbose: true
```

要使用**bundler**安装包，您还必须使用用户和令牌对其进行配置： 

```
$ bundle config \
https://rubygems.pkg.github.com/OWNER \
USERNAME:TOKEN
```

更多详细信息，请参阅https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-rubygems-registry。

### NuGet

您可以使用setup -dotnet操作来发布**NuGet**包，它有source -url的附加参数。令牌是使用环境变量设置的：

```
- uses: actions/setup-dotnet@v1
 with:
 dotnet-version: '5.0.x'
 source-url: https://nuget.pkg.github.com/OWNER/index.json
 env:
 NUGET_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

然后您可以构建并测试项目。完成后，只需打包并将包推送到注册表：

```
- run: |
  dotnet pack --configuration Release
  dotnet nuget push "bin/Release/*.nupkg"
```

要安装包，您必须将注册表作为源文件添加到nuget. config文件中，以及您的用户和令牌：

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
      <packageSources>
         <add key="github" value="https://nuget.pkg.github.com/
OWNER/index.json" />
   </packageSources>
   <packageSourceCredentials>
      <github>
         <add key="Username" value="USERNAME" />
         <add key="ClearTextPassword" value="TOKEN" />
      </github>
   </packageSourceCredentials>
</configuration>
```

更多详细信息，请参阅https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry。

## 总结

使用包很简单，最大的挑战是身份验证。但通过GitHub中的GITHUB_TOKEN可以轻松地设置完全自动化的发布工作流。这就是您的团队需要将其纳入工具箱的重要原因。如果您使用语义版本控制和单独的发布流程将代码共享为容器或包，在发布代码时将会减少很多问题。

在本章中，您学习了如何使用语义版本控制和包来更好地管理内部依赖和共享代码，也了解了包是什么，以及如何为每种包类型设置发布工作流。

在下章中，我们将更深入地了解环境以及如何将GitHub 操作部署到任意平台上。

## 延伸阅读

有关本章主题的更多信息，请参阅以下内容：

- *Semantic versioning*: https://semver.org/
- *Billing and pricing*: https://docs.github.com/en/billing/managingbilling-for-github-packages/about-billing-for-github-packages
- *Access control and visibility*: https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility
- *Working with the registry* (Container, Apache Maven, Gradle. NuGet, npm, RubyGems): https://docs.github.com/en/packages/working-with-a-github-packages-registry
