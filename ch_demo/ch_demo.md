# 第15章 保护您的部署

在本章中，本书将讨论如何保护开发者的完整部署和??发布流水线 (release pipeline) ，不仅要保护代码和依赖项，还要快速但安全且符合要求地将软件交付到安全环境，以满足监管要求。

本章将涵盖以下主要主题：
- 容器和基础设施安全扫描
- 自动化基础设施变更流程
- 源代码和基础设施完整性
- 动态应用程序安全测试
- 加固发布流程安全性

## 容器和基础设施安全扫描

在过去几年中，最引人注目的黑客攻击之一是**SolarWinds**，这是一家为网络和基础设施监控提供系统管理工具的软件公司。攻击者成功地在**Orion**软件中引入了后门，并将其推出到超过30,000个客户端，然后利用这个后门对这些客户端进行了攻击，其中包括国土安全部和财政部（*Oladimeji S.,Kerner S. M.,2021*）。

SolarWinds攻击被认为是软件供应链攻击，对于安装了被损坏版本的Orion的客户来说确实如此。但是，对Orion的攻击远不止是受感染的依赖关系的更新这么简单；攻击者获得了SolarWinds网络的访问权限，并成功地在SolarWinds构建服务器上安装了一个名为**Sunspot**的恶意软件。 Sunspot通过替换源文件，在不跟踪任何构建失败或其他可疑输出的情况下，将后门**Sunburst**插入Orion的软件构建（*Eckels S.,Smith J.,& Ballenthin W .,2020*）。

此攻击表明，如果网络被入侵，内部攻击有多么致命，以及保护整个生产线有多么重要——不仅是代码、依赖项和开发环境。构建服务器和软件生产中包含的所有其他系统都必须保持安全。

### 容器扫描

容器在今天的每个基础设施中都扮演着重要的角色。与传统的**虚拟机**  **(virtual machines, VMs)** 相比，它们具有很多优势，但也有其缺点。容器需要一种新的操作文化和？？既存流程，以前的实践可能不适用（参见*Souppaya M.,Morello J.,& Scarfone K.,2017*）。

容器由许多不同的层构成，与软件依赖关系一样，这些层也可能引入漏洞。为了检测这些漏洞，开发者可以使用所谓的**容器漏洞分析 (vulnerability analysis, CVA) **，也称为**容器安全分析 (container security analysis, CSA) **。

GitHub没有内置的CVA工具，但几乎所有解决方案都能很好地集成到GitHub中。

一个非常流行的开源容器镜像和文件系统漏洞扫描器是Anchore的**grype**(https://github.com/anchore/grype/)。它易于集成到GitHub Actions工作流中：
```bash
- name: Anchore Container Scan
  uses: anchore/scan-action@v3.2.0
  with:
    image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
    debug: true
```
另一个CVA扫描器的例子是**Clair**(https://github.com/quay/clair)，这也是一个用于静态分析Docker和**开放容器倡议 (Open** **Container Initiative, OCI) **容器漏洞的开源解决方案。Clair 可以作为容器运行，并将扫描结果存储在 Postgres 数据库中。有关完整文档，请参阅https://quay.github.io/clair/。

商业容器扫描器通常是更全面的安全平台的一部分。一个例子是 **Aqua** 的**容器安全 (Container Security) ** (https://www.aquasec.com/products/container-security/)。**Aqua 平台** (https://www.aquasec.com/aqua-cloud-native-security-platform/) 是一个面向容器化、无服务器和基于虚拟机的应用程序的云原生安全平台。Aqua 可以作为 SaaS 或自托管版本运行。

另一个例子是 **WhiteSource** (https://www.whitesourcesoftware.com/solution-for-containers/)。他们在 GitHub 市场上有 **GP 安全扫描 (GP Security Scan) **操作，用于在将映像推送到 GitHub 包之前扫描它们 (https://github.com/marketplace/actions/gp-security-scan)。

这两个方案都很好，但是它们的价格不便宜，且与 GitHub 的高级安全功能有很大的重叠，因此这里不再介绍。

### 基础设施策略

并不是所有与基础设施相关的都是容器。从安全的角度来看，需要考虑的还有很多，尤其是在云计算方面。

如果开发者正在使用云提供商，就有必要查看他们的安全组合。例如，Microsoft Azure包含Microsoft **Defender for Cloud**，这是一种**云安全态势管理 (cloud security posture management, CSPM) **工具，用于保护跨多云和混合环境的工作负载，并在云配置中寻找薄弱之处(https://azure.microsoft.com/en-us/services/defender-for-cloud)。它支持Microsoft Azure, AWS，谷歌云平台和本地工作负载(使用Azure Arc)。Microsoft Defender for Cloud中的一些功能对于Microsoft Azure是免费的——但不是全部。

Microsoft Azure还包含**Azure Policy**（https://docs.microsoft.com/en-us/azure/governance/policy/），一种帮助用户强制执行标准和评估合规性的服务。它允许用户将某些规则定义为策略定义，并在需要时评估这些策略。以下是每天上午8点在GitHub Action工作流中运行的示例：

```
on:
  schedule:
    - cron:  '0 8 * * *'
jobs:
  assess-policy-compliance:
    runs-on: ubuntu-latest
    steps:
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{secrets.AZURE_CREDENTIALS}}
    - name: Check for resource compliance
      uses: azure/policy-compliance-scan@v0
      with:
        scopes: |
          /subscriptions/<subscription id>
          /subscriptions/<...>
```

加上人工智能驱动的**安全信息和事件管理 (security information and event management, SIEM) **系统，称为**Microsoft Sentinel**，一个非常强大的安全工具链。但是，它是否有用取决于用户的主要云提供商？？是否是Azure，如果不是，那么用户对CSPM和SIEM的决定可能会截然相反，而**AWS安全枢纽 (AWS Security Hub) **对用户来说可能更合适。

**Checkov** (https://github.com/bridgecrewio/checkov)是一个很好的保护**基础设施代码(Infrastructure as Code, IaC) **的开源工具，它是一个静态代码分析工具，可以扫描使用**Terraform、Terraform plan、CloudFormation、AWS Serverless Application Model (SAM) 、Kubernetes、Dockerfile、Serverless**或**ARM templates**提供的云基础设施，并检测安全性和合规性配置错误。它拥有超过1000种不同平台的内置策略。它非常容易在GitHub中使用，只需在工作流中使用**Checkov GitHub Action**(https://github.com/marketplace/actions/checkov-github-action)，并将其指向包含基础设施的目录即可：

```
- name: Checkov GitHub Action
  uses: bridgecrewio/checkov-action@master
  with:
    directory: .
    output_format: sarif
```

该操作支持 SARIF 输出，且可以集成到 GitHub 的高级安全中：

```
- name: Upload SARIF file
  uses: github/codeql-action/upload-sarif@v1
  with:
    sarif_file: results.sarif
  if: always()
```

这些结果将在安全性|代码扫描警报下显示（参见*图15.1*）:

![image-20230212161612036](image-20230211231135576.png)

<center><p>图15.1-Checkov在GitHub中的结果</p></center>

Checkov非常适合检查IaC，但不能检查基础设施的变化。但是，如果开发者有Terraform或ARM等解决方案，可以定期在工作流中运行验证，以检查没有任何变化。

## 自动化基础设施变更流程

大多数 IT 组织都已经制定了变革管理流程，以降低运营和安全风险。大多数公司遵循**信息技术基础设施库 (Information Technology Infrastructure Library, ITIL) **。在 ITIL 中，**变更请求 (Request for Change, RFC)** 必须经过**变更咨询委员会 (Change-Advisory Board, CAB)** 的批准。问题是，CAB 的批准与软件交付性能不佳有关（参见 *Forsgren N.,Humble J.,&Kim G.,2018*）。

从安全的角度来看，**变更管理**和**职责分离**是重要的，通常也是遵从性所必需的。关键还是要以 DevOps 的方式重新思考基本原则。

有了IaC和完全自动部署，可以对所有基础设施更改进行完整的审计跟踪。如果对过程完全控制，最好的做法是将CAB设置为IaC文件的CODEOWNERS，并在pull请求中进行审批。对于应用层的简单标准变更（例如，Kubernetes集群中的容器），同行评审可能已经足够。对于对网络、防火墙或机密有更深影响的基础架构变更，应该增加审核人员的数量，也可以加入相应的专家。这些文件通常也驻留在其他存储库中，不会影响开发人员的速度，也不会减缓发布速度。

如果受到公司流程的约束，这可能并不那么容易。在这种情况下，开发者必须尝试重新分类更改，以便尽可能多地获得预先批准，同时出于安全因素对这些更改进行同行评审和自动检查。然后，自动化高风险的更改的流程，使得提交给CAB的信息尽可能全面和正确，以便尽快得到批准（参见*Kim G., Humble J., Debois P . & Willis J., 2016, Part VI, Chapter 23*）。

## 源代码和基础设施完整性

在制造业中，提供生产订单的**物料清单 (bill of materials, BOM)** 是一种常见做法。BOM是用于制造最终产品的原材料、子组件、中间组件、子组件和部件的清单。

？？软件业也存在同样的事物：**软件物料清单 (software bill of materials, SBOM)** ，但它仍然不太普遍。

### 软件物料清单

如果仔细观察软件供应链攻击，如**事件流事件 (event-stream incident**，参见*Thomas Claburn, 2018**)***，会发现它们在发布中注入恶意代码，因此GitHub中的源代码与npm包中包含的文件不匹配。SBOM在这可以用于比较不同版本的散列值来帮助进行？？司法鉴定。

在**SolarWinds攻击**中（参见 *the Crowdstrike blog, 2021*），依赖关系未被篡改；而是在MsBuild.exe执行期间有一个操纵文件系统的附加进程。为了预防和调查这类攻击，开发者必须扩展SBOM以包含构建过程中涉及的所有工具以及构建机上所有运行进程的详细信息。

有不同？？通用格式的SBOM：

-  **软件包数据交换 (Software Package Data Exchange, SPDX)** ：SPDX是SBOM的开放标准，源自Linux基金会。它源于许可证合规性，但它也包含版权、安全引用和其他元数据。SPDX最近被批准为ISO/IEC标准 (*ISO/IEC 5962:2021*) ，并且它满足NTIA的软件材料清单最低要求。
- **CycloneDX(CDX)** ：CDX是一个轻量级的开源格式，源自**OWASP**社区。它对于将SBOM生成集成到发布管道中进行了优化。
- **软件识别 (Software Identification, SWID)** 标记：SWID是ISO/IEC行业标准 (*ISO/IEC 19770-2*) ，由各种商业软件出版商使用。它支持软件清单的自动化、机器上软件漏洞的评估、缺失补丁的检测、配置清单评估的？？目标设定、软件完整性检查、安装和执行白名单/黑名单等安全和操作用例。这对为构建机器上安装的软件做清单而言是一种很好的各式。

对于每种格式，都有不同的工具和用例。**SPDX**由**syft**生成。开发者可以使用**Anchore SBOM Action**（请参阅https://github.com/marketplace/actions/anchore-sbom-action）为Docker或OCI容器生成SPDX SBOM：

```
- name: Anchore SBOM Action
        uses: anchore/sbom-action@v0.6.0
        with:
          path: .
          image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          registry-username: ${{ github.actor }}
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

SBOM 作为工作流工件上传（参见*图15.2*）：

![image-20230212201706267](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-2.png)

<center><p>图15.2-作为构建工件上传的SPDX SBOM</p></center>

**FOSSology** (https://github.com/fossology/fossology) 是一个开源许可合规解决方案，它也使用SPDX。
**CDX** (https://cyclonedx.org/) 更注重应用程序安全。市面上有**Node.js**、**.NET**、**Python**、**PHP**和**Go**的版本，但更多的语言可以通过 CLI 或其他包管理器（如 **Java**、**Maven** 和 **Conan**）支持。使用方法简单。下面是一个.NET的操作示例:

```
- name: CycloneDX .NET Generate SBOM
  uses: CycloneDX/gh-dotnet-generate-sbom@v1.0.1
  with:
    path: ./CycloneDX.sln
    github-bearer-token: ${{ secrets.GITHUB_TOKEN }}
```

与Anchore操作不同，SBOM不会自动上传，必须手动操作:

```
    - name: Upload a Build Artifact
          uses: actions/upload-artifact@v2.3.1
          with:
            path: bom.xml
```

CDX也用于**OWASP依赖追踪**(参见https://github.com/DependencyTrack/Dependency-track)——一个组件分析平台，开发者可以将其作为容器或在Kubernetes中运行。开发者可以把SBOM直接上传到他的DependencyTrack实例中:

```
uses: DependencyTrack/gh-upload-sbom@v1.0.0
with:
  serverhostname: 'your-instance.org'
  apikey: ${{ secrets.DEPENDENCYTRACK_APIKEY }}
  projectname: 'Your Project Name'
  projectversion: 'main'
```

SWID标签更多地用于**软件资产管理 (Software Asset Management, SAM)** 解决方案，如snow (https://www.snowsoftware.com/)、**Microsoft System Center**或**ServiceNow ITOM**。CDX和SPDX可以使用已存的SWID标签。

如果想了解更多关于SBOM的信息，请参阅https://www.ntia.gov/sbom。

如果开发者完全在GitHub企业云上工作，并使用托管的运行程序，SBOM就不是那么重要了。毕竟所有相关数据都已连接至GitHub上。但是，如果是在GitHub企业服务器上使用自托管运行器和其他未通过公共包管理器使用的商业软件，那么为所有版本创建 SBOM 可以帮助检测漏洞、许可问题，并在发生事故时帮助进行司法鉴定。

### 签署提交

许多人都经常讨论是否应该对所有提交都进行签名。Git 是一种非常强大的工具，可以随意修改现有的提交。但这也意味着，提交的作者未必就是提交代码的人。一次提交有两个字段：作者和提交者。这两个字段的值均来自 git config 的 user.name 和 user.email 加上时间戳。例如，如果进行重构，提交者会变为当前值，而作者保持不变。这两个字段与 GitHub 的身份验证没有任何关系。

读者可以在Linux仓库中查找**Linus Torvalds**的电子邮件地址，然后使用该电子邮件地址配置本地Git仓库，并将其提交到仓库。此提交的作者将显示为Linus（参见*图15.3*）：

![image-20230212211217918](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-3.png)

> Any valid email address on GitHub links to the actual profile    任何有效的GitHub电子邮件地址都可以链接到实际的个人资料页面。
>
> But the commit is not verified    但是该提交并未通过验证。
>
> 图的中文翻译

<center><p>图15.3-一次提交的作者信息与其身份完全无关</p></center>

头像中的链接也可以正常工作，并重定向到正确的个人主页。但是，与在服务器上进行的修改（通过在Web UI中修改文件或使用拉请求合并更改）不同，提交并没有被验证的徽章。验证徽章表示提交使用了**GNU隐私保护 (GNU Privacy Guard, GPG)** 密钥签名，该密钥包含了您帐户的已验证电子邮件地址（参见*图15.4*）。

![image-20230212214957305](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-4.png)

<center><p>图15.4-签名提交在GitHub上有一个经过验证的徽章</p></center>

如果想对提交进行签名，可以在本地创建GPG密钥（使用git commit -S）。当然，读者可以自由设置密钥中的名称和电子邮件地址，只需要与git config中配置的电子邮件和用户匹配即可。 只要不修改提交，签名就有效（参见*图15.5*）。

![image-20230212215414107](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-5.png)

<center><p>图15.5-如果电子邮件和名称匹配，则本地签名提交有效</p></center>

但是，即使将**Pretty Good Privacy (PGP)**密钥上传到GitHub配置文件 (https://github.com/settings/gpg/new)，提交也不会被验证，因为GitHub在已验证电子邮件地址的档案中查找密钥 (参见*图15.6*)：

![image-20230212221218979](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-6.png)

> even a valid signed commit is unverified, if the mail address is associated with another GitHub handle   即使提交已经被签名，但如果电子邮件地址与另一个GitHub用户关联，则该提交也不会被验证。   
>
> 图的中文翻译

<center><p>图15.6-来自其他用户的已签名提交不会得到验证</p></center>

这是否意味着必须在本地签署所有提交?本书不这么认为。强制开发人员签名所有提交会拖慢速度。许多ide和工具都不支持签名。保持密钥同步，处理多个电子邮件地址——一切都变得更加痛苦。如果所有开发人员在公司设备上使用相同的电子邮件地址，可能会很好，但通常情况并非如此。人们在远程、不同的机器上以不同的环境工作，并且在同一台机器上以不同的邮箱地址而不是统一的公司代码使用开源软件。这样是得不偿失的，如果攻击者有推送权限，最不用担心的就是伪造的电子邮件地址。

本书建议如下：

- 选择一个依赖于**拉取请求**的工作流，并对服务器上的更改进行合并、压缩或重定，以便它们默认被签名。
- 如果需要为发布保证完整性，请对标签进行签名 (git tag -S) 。由于Git是基于SHA-1或SHA-256的树，因此签名标签将确保所有父级提交没有被修改。

与其要求开发人员本地签名所有提交，减缓团队进度，不如？？要求在构建过程中对代码进行签名，以确保构建过程后没有人篡改您的文件。

### 签署代码

即使签署的是二进制文件而不是代码，对二进制文件进行签名仍被称为代码签名。开发者需要从可信任机构获取证书才能进行此操作。在构建过程中如何为代码签名很大程度上取决于使用的语言以及它的编译方式。

要在GitHub Actions中签署Apple XCode应用程序，开发者可以使用此文档在构建期间安装发布配置文件和Base64编码的证书：https://docs.github.com/en/actions/deployment/deploying-xcode-applications/installing-an-apple-certificate-on-macos-runners-for-xcode-development。不要忘记在与其他团队共享的自托管运行程序上进行清理。在GitHub托管的运行程序上，每个构建都会获得一个纯净的环境。

根据代码签名解决方案，开发者可以在市场上找到多个Authenticode和signtool.exe操作。但由于所有签名解决方案都是基于命令行的，可以像Apple的示例一样将签名证书通过密钥上下文传递给工作流程。

## 动态应用程序安全测试

为加强应用程序安全性，开发者可以将**动态应用程序安全测试 (dynamic application security** **testing, DAST)** 集成到发布工作流中。DAST是一种黑盒测试，模拟对正在运行的应用程序进行真实攻击。

 有许多商业工具和SaaS解决方案（例如**PortSwigger**的**Burp Suit**或**WhiteHat Sentinel**），但分析它们不在本书探讨的范围之内。 

也有一些开源解决方案，例如来自OWASP的**Zed Attack Proxy (ZAP)**（https://www.zaproxy.org/）。它是一个独立的应用程序，可以在Windows、macOS和Linux上运行（请参见https://www.zaproxy.org/download/），可用于攻击Web应用程序。该应用程序允许用户分析Web应用程序、拦截和修改流量，并使用ZAP Spider对网站或其部分进行攻击（参见*图15.7*）：

![image-20230215192553457](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15_7.png)

<center><p>图15.7-OWASP ZAP应用程序</p></center>

OWASP ZAP启动一个浏览器并使用一个**提示显示器 (heads-up display, HUD)** 来在网站顶部显示控件。用户可以使用这些控件来分析站点、使用爬虫运行攻击、拦截请求，？？而无需其他应用程序的协助（参见*图15.8*）：

![image-20230215193034234](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-8.png)

> OWASP ZAP intercepts the traffic OWASP ZAP拦截流量
>
> Heads-up Display(HUD) to analyze and attack the site using the spider ？？通过平视显示器(HUD)使用爬虫分析和攻击站点
>
> Items can be customized  可以自定义项目
>
> 图的中文翻译

<center><p>图15.8-HUD会在被攻击的网站上显示控件</p></center>

即使不是渗透测试人员，作为Web开发人员，开始学习如何使用OW ASP ZAP攻击自己的网站应该很容易。但是为了将安全性前移，应该将扫描集成到工作流中。OWASP ZAP在GitHub市场中有三个Actions（请参见*图15.9*）：

![image-20230215194344374](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-9.png)

<center><p>图15.9-在 GitHub 市场中可用的 OWASP ZAP Actions</p></center>

**Baseline Scan**比**Full Scan**快。**API Scan**可用于扫描**OpenAPI**、**SOAP**或**GraphQL API**。这些Actions的使用很简单：

```
- name: OWASP ZAP Full Scan
  uses: zaproxy/action-full-scan@v0.3.0
  with:
    target: ${{ env.TARGET_URL }}
```

该操作使用GItHub_TOKEN将结果写入GitHub Issue，它还将报告作为构建产物添加。报告有HTML、JSON或Markdown格式（参见*图15.10*）。

![image-20230216144757922](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15_10.png)

> A report is added as a build artifact in HTML,JSON and Markdown 一份报告将以 HTML、JSON 和 Markdown 的形式作为构建产物添加
>
> A GitHub issue is created with a link to the workflow run 同时，该 Action 还会在 GitHub 中创建一个 Issue 并附带一个指向该工作流运行的链接
>
> 图的中文翻译

<center><p>图15.10-OWASP ZAP 扫描结果</p></center>

当然，这仅适用于Web应用程序。还有其他用于其他场景的DAST工具。但是该示例展示了将其纳入到流水线中是多么容易。大多数DAST工具是命令行工具或容器，或者它们已经有了集成，例如OWASP ZAP。

## 加固发布流程安全性

CI/CD 流水线非常复杂，且面临着被攻击的风险。基本上，发布流水线是远程代码执行环境，应谨慎对待（有关一些攻击示例，请参见*Haymore A., Smart I., Gazdag V ., Natesan D., & Fernick J., 2022* ）。

因此，建议开发者在建模流水线时要谨慎，并遵循前人的经验，特别是在构建高度定制的流水线时。与其后悔为时已晚，不如趁早寻求外界的帮助。

### 保护运行器

如果开发者使用的是GitHub托管的运行器，它们的安全工作是由GitHub完成的。这些运行器是临时的，每次执行都会在一个纯净的状态下开始。但是开发者执行的代码可以访问其GitHub中的资源，包括机密信息。请确保GitHub Actions安全（参见*Secure your Actions*部分），并限制GitHub_TOKEN的权限（工作流应以尽可能低的权限运行）。

开发者有责任确保在其环境中运行的自托管的运行器的安全。以下是应遵循的一些规则：

- 不要将自托管运行器用于**公共代码库**。

- 确保运行器是**临时的**。（或者至少在每次运行后进行清理，不要在磁盘或内存中留下产物）

- 保持镜像**轻量化**且**打了最新的补丁**（仅安装所需工具并保持所有内容更新）。

- 不要让所有团队和技术使用**通用运行器**。保持镜像分离和特化。

- 将运行器放在一个**隔离的网络**中(只允许运行器访问他们需要的资源)。

- 只运行**安全的Actions**。

- 将运行器置于您的**安全监控**中，并检查是否有异常进程或网络活动。

最好的解决方案是拥有一个动态扩展的环境（例如 Kubernetes 服务），并使用轻量且已打补丁的镜像运行临时的运行器。

有关自托管和托管运行程序的详细信息，请参见*第7章，运行工作流*。

### 保护Actions

GitHub Actions非常有用，但它们是开发者执行的代码，并被授予访问资源的权限。应该非常谨慎地选择使用哪些Actions，特别是对于自托管的运行器。可信来源（如GitHub、Microsoft、AWS或Google）的Actions不是问题。但即使它们接受拉请求，仍然有可能存在漏洞。Actions的最佳实践如下：

- 经常**检查Action的代码**。同时，查看所有者、贡献者数量、提交数量和日期、点赞数量以及所有这些类型的指标，以确定Action是否属于一个健康的社区。
- 始终使用显式的**提交SHA**引用Action。SHA是不可变的，而标签和分支可能会被修改，导致新？？版本的代码在不知情的情况下执行。
- 如果开发者正在开发分支，**需要**所有外部合作者的**批准**，而不仅仅是第一次贡献者。
- 使用**Dependabot**保持Actions最新。

如果开发者是自托管运行器，应该更加严格的限制可使用的Actions。有两种可能性：

- **仅允许使用本地Actions**，并创建一个从已分析的Action分支的fork，并引用该分支。这需要额外的工作，但可以完全控制开发者使用的Actions。开发者可以将Actions添加到本地市场，以便更轻松地找到（参见*Rob Bos, 2022*）。
-  **允许**从GitHub和特定允许的Actions列表（白名单）中**选择Actions**。开发者可以使用通配符允许同一所有者的所有Actions（例如，Azure/*）。这个做法没有上一个安全，但也不需要太多维护工作。

开发者可以为每个组织或企业政策配置这些选项。Actions是在自己的环境中执行的其他人的代码。它们是可能破坏发布和并引入漏洞的依赖项。开发者应该在速度和安全性之间找到最佳平衡点。

### 保护环境

使用**环境保护规则**并设置**必需审阅者**来审核发布之前的部署（参见*第9章“部署到任何平台”*中的**分阶段部署**）。这可以确保在访问环境的密钥和执行代码之前，发布已经被审核。 

结合**分支保护**和**代码所有者**（参见*第3章“团队合作和协作开发”*）的做法，仅允许特定分支进入开发者的环境。这样，可以确定必要的自动化测试和代码所有者的批准在批准部署时已经完成。

### 尽量使用令牌

与将存储为机密的凭据连接到云提供程序（如Azure、AWS、GCP或HashiCorp）不同，开发者可以使用**OpenID Connect（OIDC）**。 OIDC将交换短期的令牌以进行身份验证，而不是凭据。云提供程序也需要在其端支持OIDC。

使用OIDC，开发者不必在GitHub中存储云凭据，可以更加精细地控制工作流可以访问哪些资源，并且具有？？定期更换的短期令牌，这些令牌将在工作流运行后过期。

*图15.11*展示了OIDC的工作原理概述：

![image-20230216200305636](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-11.png)

<center><p>图15.11-与云服务商的OIDC集成</p></center>

具体步骤如下：

- 创建一个云提供商和 GitHub 之间的 **OIDC 信任**关系。将信任限制为一个组织和存储库，并进一步限制访问环境、分支或拉取请求。

- GitHub OIDC提供程序在工作流运行期间**自动生成一个JSON Web Token**。该令牌包含多个声明，用于为特定工作流作业建立安全且可验证的标识。

- 云服务提供商验证这些声明，并提供一个短期的访问令牌，仅在工作流程任务的生命周期内有效。

- 访问令牌用于通过有权访问资源的标识

开发者可以使用标识直接访问资源，或者使用它从安全保险库（例如**Azure Key Vault**或**HashiCorp Vault**）获取凭据。这样，可以安全地连接到不支持OIDC和自动密钥轮换的服务。

 在GitHub中，可以找到有关为AWS、Azure和GDP配置OIDC的说明（参见https://docs.github.com/en/actions/deployment/security-hardening-your-deployments）。步骤很简单。例如，在Azure中，可以在**Azure Active Directory（AAD）**中创建一个应用程序注册：

```
$ az ad app create --display-name AccelerateDevOps
```

然后，使用注册输出中的应用程序 ID 创建一个服务主体：

```
$ az ad sp create --id <appId>
```

然后，可以在AAD中打开应用程序注册，并在“证书和密码|联合凭据|添加凭据”下添加 OIDC 信任。请根据*图 15.12* 中的说明填写表单。

![image-20230216202025993](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-12.png)

<center><p>图15.12-为应用程序注册创建OIDC信任</p></center>

然后，在订阅级别为服务主体指定一个角色。在门户中打开订阅。在**访问控制(IAM) | 角色分配 | 添加 | 添加角色分配**下，按照向导提示进行操作。选择一个角色（例如，**贡献者**），然后单击**下一步**。选择**用户、组或服务主体**，并选择前面创建的服务主体。

在GitHub中，开发者的工作流需要id-token的写权限:

```
permissions:
      id-token: write
      contents: read
```

在 Azure Login Action中，使用客户端 ID（appId）、租户 ID 和订阅 ID 从 Azure 中检索令牌：

```
- name: 'Az CLI login'
  uses: azure/login@v1
  with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

之后，就可以使用Azure CLI访问资源:

```
- run: az account show
```

也可以使用其他 Azure Actions 并删除身份验证部分，在本例中是发布配置文件（publishing profile）。这些操作将使用登录操作提供的访问令牌（access token）：

```
- name: Run Azure webapp deploy action using OIDC
  uses: azure/webapps-deploy@v2
  with:
    app-name: ${{ env.APPNAME }}
    slot-name: Production
    package: website
```

尽管是不同的云提供商，文档应该可以让开发者快速上手：https://docs.github.com/en/actions/deployment/security-hardening-your-deployments。

### ？？收集安全数据

为了确保从代码到生产的整个流水线的安全，开发者需要实时了解各个层面的情况。不同层面有不同的监控方案(参见*图15.13*):

![image-20230216224840325](D:\文档\书籍翻译\AccelerateDevOps\ch_demo\15-13.png)

> Security 安全
>
> Customer 客户
>
> Code 代码
>
> Service 服务
>
> Infrastructure 基础设施
>
> Security Information&Event Management(SIEM) 安全信息与事件管理
>
> Real User Monitoring(RUM) 真实用户监测
>
> Digital Experience Monitoring(DEM) 数字体验监测
>
> Application Performance Monitoring(APM) 应用程序性能监测
>
> Pipeline Monitoring 流水线检测
>
> Synthetic Transaction Monitoring(STM) 合成事务监测
>
> Application Discovery,Tracing&Diagnostics(ADTD) 应用发现、追踪和诊断
>
> Platform Monitoring 平台监测
>
> Network Performance Monitoring(NPM) 网络性能监测
>
> IT Infrastructure Monitoring(ITIM) IT基础设施监测
>
> Log Monitoring 日志监测
>
> 图中的中文翻译

<center><p>图15.13-不同层面的监控</p></center>

所有这些层都应将数据报告给SIEM系统，以执行分析并使用AI检测异常。许多组织在不同层面收集数据，但由于职责不同而忘记将其纳入监控。为了加固发布安全，开发者应该考虑以下几点：

- 在SIEM解决方案中包含**所有监视来源**和事件。
- 监视**整个流水线**，包括代理和测试环境，以及所有进程和网络活动。
- **记录部署事件**及相应版本。如果在部署后突然运行新进程或打开端口，则需要将这些更改与该部署相关联，以便进行取证。
- 收集**实时应用程序安全数据**，并在仪表盘上显示。这可能包括**异常程序终止、SQL注入尝试、跨站点(XSS)脚本尝试、登录失败（暴力攻击）**或**DDos攻击**，但具体监测哪些数据，要根据产品而定。如果用户的输入包含可疑的字符或元素，为了检测 SQL 注入或 XSS 攻击，需要在对用户输入进行编码之前增加额外的日志记录。

让人们真正了解威胁的严重性的最好方法是展示威胁已经发生或可能发生的实际情况。

### 案例研究

直到现在，**Tailwind Gears**一直雇用外部公司对架构进行**安全审查**、帮助进行**威胁建模**和**风险分析**并在主要版本发布之前进行安全测试。他们迄今为止的大部分投资都用于网络安全，并且从未被攻破。然而，随着使用越来越多的云服务，他们已经意识到必须采取一些措施来使其能够**检测**、**响应**和**恢复**，以加强安全性。

IT部门已经开始使用**Splunk**作为他们的**SIEM**和**ITIM**解决方案，并集成了越来越多的数据源，但直到现在，IT部门仍不能确定他们是否真的能实时检测到正在进行的攻击。Tailwind Gears决定改变他们处理安全问题的方式。他们与其安全伙伴交谈，并计划了首次**红队/蓝队**模拟。模拟场景是一名**内部攻击者**入侵了我们DevOps试点团队的web应用程序。

这个模拟演练历时3天，最终红队通过找到两种方法来攻击生产环境获胜：

- 一次矛头针对另一个团队中的一些开发人员的**鱼叉式网络钓鱼**攻击成功了，并泄露了其中一名开发人员的凭证。使用**BloodHound**，他们发现该开发人员可以访问之前运行GitHub Actions 运行器但还没有完全转移到Kubernetes解决方案的Jenkins服务器。该服务器没有启用MFA，而**mimikatz**可以捕获测试帐户的凭证。测试帐户可以访问测试环境，然后可以捕获那的管理员帐户凭证，该管理员帐户允许从分阶段环境中提取数据（在比拼中算作生产环境）。
- 由于所有开发人员都有对任何存储库的读访问权，因此对Web应用程序的依赖关系进行分析后发现一个容易受到XSS攻击的组件，且尚未修补。该组件是一个搜索控件，允许红队在另一个团队的前端开发人员帮助，在其他用户的上下文中执行脚本。他们在内部GitHub存储库中打开一个议题，并使用GitHub API在每次执行时向该议题发布一条评论作为证明。

这次模拟过程中发现了许多后续需要解决的问题，将在接下来几周内解决。有些事情不涉及我们的DevOps团队，例如为所有内部系统启用MFA，或定期执行钓鱼模拟以提高员工的安全意识。但是也有许多问题涉及到了团队。Tailwind Gears决定将安全性融入到开发过程中。这包括**秘密扫描**、使用Dependabot进行**依赖关系管理**和**代码扫描**等操作。

团队还将与IT部门合作，通过将构建服务器迁移到Kubernetes，实施整个流水线的**安全日志记录**，以及使用**OpenID Connect**和**安全密钥库**来处理机密信息，从而使发布流水线更加安全可靠。

大家都期待着3个月后的下一次红队/蓝队模拟比赛。

## 总结

本章中，读者学习了如何通过扫描容器和IaC，确保代码和配置的一致性，并对整个流水线进行安全加固来保护发布流水线和部署。

在下一章中，本书将讨论软件架构对软件交付性能的影响。

## 延伸阅读

以下是本章的参考资料，读者也可使用它们来获取相关内容的更多信息：

- Kim G., Humble J., Debois P . & Willis J. (2016). The DevOps Handbook: How to Create World-Class Agility, Reliability, and Security in Technology Organizations (1st ed.). IT Revolution Press
- Forsgren N., Humble, J., & Kim, G. (2018). Accelerate: The Science of Lean Software and DevOps: Building and Scaling High Performing Technology Organizations (1st ed.) [E-book]. IT Revolution Press.
- Oladimeji S., Kerner S. M. (2021). SolarWinds hack explained: Everything you need to know. https://whatis.techtarget.com/feature/SolarWinds-hack-explained-Everything-you-need-to-know
- Sudhakar Ramakrishna (2021). New Findings From Our Investigation of SUNBURST. https://orangematter.solarwinds.com/2021/01/11/new-findings-from-our-investigation-of-sunburst/
- Crowdstrike blog (2021). SUNSPOT: An Implant in the Build Process. https://www.crowdstrike.com/blog/sunspot-malware-technical-analysis/
- Eckels S., Smith J. & Ballenthin W . (2020). SUNBURST Additional Technical Details. https://www.mandiant.com/resources/sunburst-additional-technical-details
- Souppaya M., Morello J., & Scarfone K. (2017). Application Container Security Guide: https://doi.org/10.6028/NIST.SP.800-190
- National Telecommunications and Information Administration (NTIA), Software Bill of Materials: https://www.ntia.gov/sbom
- Thomas Claburn (2018). Check your repos... Crypto-coin-stealing code sneaks into fairly popular NPM lib (2m downloads per week): https://www.theregister.com/2018/11/26/npm_repo_bitcoin_stealer/
- Haymore A., Smart I., Gazdag V ., Natesan D., & Fernick J. (2022). 10 real-world stories of how we've compromised CI/CD pipelines: https://research.nccgroup.com/2022/01/13/10-real-world-stories-of-how-weve-compromised-ci-cd-pipelines/
- Rob Bos (2022). Setup an internal GitHub Actions Marketplace: https://devopsjournal.io/blog/2021/10/14/GitHub-Actions-Internal-Marketplace.html

