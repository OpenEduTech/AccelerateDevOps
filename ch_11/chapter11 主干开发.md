# 第11章 主干开发

与加速工程速度高度相关的功能之一是**主干开发**(也称为**TBD**)。高效的团队在任何时候都不超过三个活跃分支，并且他们的分支在合并到主分支之前的生命周期很短(不到一天)(Forsgren N.、Humble J.和Kirn G.2018 年，第 98 页)。不幸的是，TBD 不是 git **工作流**，而是自 80 年代以来一直在使用的分支模型。它的定义不明确，有很大的解释空间，尤其是在与 GitHub 一起使用时。此外，我个人发现，仅切换到基于主干的工作流程不会对性能有很大的改善，只有具有高度复杂工作流程的大型团队在陷入**合并困境**时才会真正产生很大的影响。对于大多数团队来说，更多的是？？功能发布控制（feature flags）和持续集成/持续部署(**CI/CD**)等不同功能的组合，以及基于主干的工作流，这会产生很大的影响。

在本章中，我将解释基于主干的工作流的好处以及它与其他分支工作流的区别，还将把我认为最好的git工作流来介绍给您，以加速您的软件交付。

本章涵盖以下主题：

- 主干开发
- 为什么应该避免复杂分支
- 其他git 工作流
- 使用MyFlow加速
- 案例分析

## 主干开发

主干开发是一种源代码控制分支模型，开发人员将小而频繁的更新合并到单个分支(通常称为主干，但在 git 中，这通常称为主分支)并抵制创建其他长期开发分支。(参阅 https://trunkbaseddevelopment.com)。

基本思想是主分支始终处于干净状态，以便任何开发人员在任何时候都可以基于成功构建的主分支创建新分支。

为了保持分支处于干净状态，开发人员必须采取多种措施来确保只有不会破坏任何内容的代码才能被合并到主分支，如下所述:

- 从主分支获取最新的变更
- 执行清洁测试
- 运行所有测试
- 与你的团队有高度的凝聚力(结对编程或代码审查)

正如您所看到的，这是为受保护的主分支和CI构建的拉取（PR）请求而预先确定的，该 CI 构建具有 PR 触发器并且能够构建和测试您的变更。然而，一些团队更喜欢手动完成这些步骤并在没有分支保护的情况下直接推送到main分支。对于实践结对编程的高凝聚力小型同地域团队，这可能非常有效，但需要很多准则进行约束。在复杂的环境或异步工作的分布式团队中，我总是建议使用分支保护和 PR。

## 为什么应该避免复杂分支

当我们谈论分支时，我们经常使用长期的（long-lived）和短期的（short-lived）这两个术语，它们指的是时间。我发现这在某种程度上具有误导性。分支是关于变化的，而变化很难及时？？衡量。开发人员可以编写 8 小时的代码同时进行大量重构，并尝试在 1 天内合并这个非常复杂的分支。如果他们做到了及时衡量，这个分支仍然会被认为是短期（short-lived）的。相反，如果他们有一个分支只变更了一行，例如，更新了代码依赖的包，但由于团队必须解决一些关于变更的架构问题，导致该分支持续3周保持开放 ，即使在 main分支上只进行了非常简单的变更，但从时间角度上来说它也是长期（long-lived）的。

时间似乎不是区分分支好坏的最佳衡量标准，而应是复杂度和时间的结合。

在您尝试合并变更之前，基于基础分支所创建的分支中的变更越多，将这些变更与其他分支的变更合并的难度就越大。复杂性可能来自一个非常复杂的合并，或者来自许多开发人员合并许多小的变更。为了避免合并，许多团队试图在合并之前完成一个功能（feature）。当然，这会导致更复杂的变更，使得其他功能难以合并——也就是所谓的合并地狱。因此，在发布之前，所有功能都必须集成到新版本中。

为了避免合并地狱，您应该定期拉取主分支的最新版本。只要您可以顺利地合并或变基，分支的集成就不是问题，但是如果您的变更过于复杂，当您合并变更时，其他开发人员可能会出现问题。这就是为什么您应该在变更超过一定的复杂度之前将其合并。复杂程度在很大程度上取决于您修改的代码，您需要考虑以下几点:

- 您是在使用现有代码还是新代码?
- 是有很多依赖关系的复杂代码，还是简单代码?
- 您使用的是孤立的代码还是高内聚的代码?
- 有多少人在同时更改代码?
- 是否同时对很多代码进行重构?

人们倾向于使用时间而非复杂性来进行？？衡量的原因——我认为是复杂性没有好的衡量标准。所以，作为一个经验法则:如果您在完成一个更为复杂的功能，至少应该以每天一次的频率将变更合并到主分支，但如果变更很简单，那么让您的分支/PR 长时间开放是没有问题的。请记住，这与时间无关，而与复杂度有关!

## 其他git工作流

对于使用GitHub的DevOps工作团队，我认为最有效的工作流是git，在我们进一步研究git前，我想介绍一下当前最流行的工作流。

### Gitflow

Gitflow 仍然是最流行的工作流之一。它由 Vincent Driessen于2010年推出(参见https://nvie.com/posts/a-successful-git-branching-mode1/)并流行开来。**Gitflow**有一个很好的海报，它对如何解决 git 中的问题进行了描述性的介绍，例如如何使用标签发布和处理合并后删除的分支(见图 11.1)：

![fig11-1](./chapter11.assets/fig11-1.png)

main                                         主分支

Tags                                          标签

hotfix for production             生产修补程序

Branch hotfix                          修补程序分支

Backwards integration of bugfix into develop        将错误修复程序向后集成到开发分支中       

Start new release 2.0             启动新版本2.0

Only bugfixes                          仅错误修复程序

develop                                     开发分支

Feature for next release        下一版本的功能

Start new release 2.1             启动2.1新版本

Bugfixes from release branches get integrated into develop regularly     发布分支的错误修复程序定期集成到开发分支中

Figure 11.1 — Gitflow overview        图11.1 GitFlow 概览



如果您每隔几个月将软件发送给不同的客户，想要将一些功能？？绑定（bundle）到单独许可的新的主要版本，并且需要持续多年地维护多个版本，那么 Gitflow非常有用。在2010年，这几乎是所有软件的通用发布流程，但在复杂的环境中，该工作流会引发一些问题。它不是基于主干的，并且有多个长期存在的分支。在复杂环境中，这些分支之间的集成可能导致合并地狱。随着 DevOps 和 CI/CD 实践的兴起，GitFlow名声渐差。

如果您想通过DevOps加速软件交付，Gitflow不是适合您的分支工作流！但它的许多概念可以在其他工作流中找到。

### GitHub flow

GitHub flow非常注重与PR的合作。首先，您创建一个具有描述性名字的分支并进行第一次变更，然后，您创建一个PR并通过代码的审阅意见与审阅者合作。一旦PR准备就绪，它会在合并到主分支之前被传送到生产环境。（参见图11.2）

![fig11-2](./chapter11.assets/fig11-2.png)

Create branch                创建分支

Commit changes            提交变更

Create pull request        创建拉取请求

Collaborate                      合作

Deploy(ship it)                 部署（传输）

Merge                                合并

Figure 11.2 — GitHub flow        图11.2 GitHub流

GitHub Flow是基于主干的，且非常流行。不包含部署PR的基本部分是大多数其他工作流的基础。而问题就在于部署。将每个 PR 部署到生产中会造成瓶颈，并且不能很好地扩展。

GitHub 本身使用 ChatOps 和部署？？列车（trains）来解决这个问题(Aman Gupta，2015 年)，但这对我来说似乎有点矫枉过正。只有在生产环境中被正式有效的更改才被合到主分支中，这个观点是有说服力的，但是在复杂的环境下，这个目标基本无法达成。您将需要相当长的时间来查看在生产环境中隔离运行的变更，以真正确保它们不具破坏性，但在这段时间内，该瓶颈会阻止其他团队或团队成员合并他们的更改。我认为在具有快速失败和前滚原则的DevOps世界中，最好在隔离环境中验证PR，并在使用主分支的push触发器合并 PR 后将它们部署到生产环境中。如果这些变更对生产环境产生了破坏，您仍然可以部署上一个有效的版本(回滚)，或者修复错误并立即部署修复(前滚）。您不需要干净的主分支来执行这些选项中的任何一个。

我不喜欢 GitHub Flow 的另一个原因是它对用户、分支和 PR 的数量不是很明确。一个功能分支可能意味着多个人向同一个功能分支提交代码。我不常看到这种情况发生，但仅从文档来看，它并没有明确说明。

### Release flow

发布流基于GitHub流，但它不是连续部署PR，而是添加单向发布分支。分支不会合并，并且修复遵循上游优先原则：错误在基于main分支的一个分支中修复后，更改被拣选到发布分支中的一个分支里（Edward Thomson，2018）。这样，就不可能忘记对main分支进行错误修复了（参见图11.3）:

![fig11-3](./chapter11.assets/fig11-3.png)

main                                     主分支

Create release branch      创建发布分支

Create bugfix branch        创建错误修复分支

Cherry- picking  to a new branch from release    从发布分支拣选到一个新的分支

Create release branch      创建发布分支

release /1.6                         版本1.6

release /1.7                         版本1.7

Figure 11.3 — Release flow        图11.3 发布流

发布流不是CD！创建发布仍然是一个必须单独触发的过程。如果您必须维护软件的不同版本，发布流程是一种很好的方式。条件允许的情况下，您应该尽量做到持续部署。

### GitLab flow

GitLab流程也基于GitHub流程。它添加了环境分支（例如开发，暂存，预生产以及生产），并且每次部署都是在合并到这些环境时发生的（参见图11.4）。

![fig11-4](./chapter11.assets/fig11-4.png)

production                           生产分支

development                       开发分支

main                                     主分支

Deploy to development environment       部署到开发环境

Deploy to production environment           部署到生产环境

Figure 11.4 — GitLab environment branches        图11.4 GitLab环境分支

由于变更仅流向下游，因此您可以确保所有变更在所有环境中都进行了测试。GitLab 流程也遵循上游优先的原则。如果您在其中一个环境中发现错误，您可以创建一个基于main分支的功能分支，并拣选该更改，使其对所有环境生效。错误修复在 GitLab 流程中的工作方式与在发布流中的工作方式相同。

如果您没有支持多种环境的流水线 一 例如 GitHub Actions，GitLab 流可能会提供一种不错的方法来自动执行您对环境的批准和部署。就个人而言，如果您在上游修复错误，将失去环境代码分开的价值。我更喜欢一次构建代码，然后按顺序将输出部署到所有环境。但在某些情况下，例如，对于直接从存储库部署的静态网站来说，此工作流可能有意义。

## 使用MyFlow加速

如你所见，git工作流只是针对不同使用场景的解决方案的集合。主要区别在于它们是否是基于主干的以及它们是否明确说明某些事情。当发现所有工作流都存在缺陷时，可以创建了自己的工作流：MyFlow。

MyFlow 是一个基于 PR 的轻量级、基于主干的工作流。它不是新发明！许多团队已经采用这种方式工作。如果你专注于与 PR 协作，这是一种非常自然的分支和合并方式。我只是给它取了一个名字，我可以预见人们能容易地使用它。

### 主分支

由于 MyFlow 是基于主干的，因此只有一个称为 main 的主分支，并且它应该始终处于干净状态。主分支应该始终被构建，并且应该能够随时将其发布到生产环境中。这就是为什么您应该使用分支保护规则来保护 main分支。一个好的分支保护规则至少包括以下标准:

- 合并前需要至少两次 PR 审查
- 推送新提交时取消过时的 PR 批准
- 需要代码所有者的审查
- 合并前需要通过状态检查，包括 CI构建、测试执行、代码分析和静态代码分析
- 将管理员包含在限制中
- 允许强制推送

使用CI构建的自动化程度越高，保持分支处于干净状态的可能性就越大。

其他所有分支总是从main分支派生出来，由于它是默认分支，当您创建新分支时无需指定源分支，简化操作的同时消除了错误源。

### 私有主题分支

图11.5展示了Myflow的基本概念：

![fig11-5](./chapter11.assets/fig11-5.png)

main                              主分支

Create branch users/<user>/<topic>         创建分支users/<user>/<topic>

Create draft pull request                  创建初步的拉取请求

Ready for review                                 准备接受审查

Collaborate                                          合作

Validate                                                 验证

Merge and deploy using CI/CD(ship it)    使用CI/CD合并及部署（传输）

Figure 11.5 — Basics of Myflow        图11.5 Myflow基础

私有主题分支可用于处理新功能、文档、错误、基础架构以及仓库中的其他所有内容。它们是私有的，这意味着它们只属于一个特定的用户。其他团队成员可以跳转到该分支来测试解决方案，但不允许他们直接将变更推送到该分支。相反，他们必须使用 PR 中的建议来向 PR 的作者提出变更建议。

为了表明分支是私有的，我建议使用 users/* 或 private/* 等命名约定，使其显而易见。我还建议在名称中包含问题或错误的标识符(ID)，使其稍后在提交消息中更容易被引用。一个好的约定应该是这样的：

```
users/<username>/<id>_<topic>
```

要开始研究一个新主题，您可以创建一个新分支，如下所示：

```
$ git switch -c <branch> main
```

示例如下：

```
$ git switch -c users/kaufm/42_new-feature main
> Switched to a new branch 'users/kaufm/42_new-feature'
```

创建您的第一个修改并将其推送到服务器，修改什么并不重要——您只需在文件中添加一个空白即可。无论如何您后续都可以对它进行重写。示例如下：

```
$ git add .
$ git commit
$ git push --set-upstream origin <branch>
```

现在，对上述示例添加更多信息：

```
$ git add .
$ git commit -m "New feature #42"
$ git push --set-upstream origin users/kaufm/42_new-feature
```

> 注意
>
> 请注意，我是用**GitHub命令行界面（**GitHub CLI**） (https://
>
> cli.github.com/)与PR进行交互，因为我发现它比使用web**用户界面**（**UI**）的屏幕快照更易理解。您也可以使用web UI执行相同的操作。

创建一个PR并将其标记为draft，如下所示：

```
$ gh pr create --fill --draft
```

这样，团队就知道您正在研究这个主题。快速查看开放的PR列表应该能让您很好地了解团队当前正在研究的主题。

> 注意
>
> 您可以在提交时更改时省略-m参数，并在默认编辑器中添加多个提交消息。第一行时PR的标题；消息的其余部分是正文。您还可以在创建PR时设置标题（--title 或 -t）和正文(--body 或 -b）来代替--fill。

您现在可以开始研究主题，并且可以使用git的全部功能。比如，如果您想对之前的提交添加变更，你可以使用--amend选项，如下所示：

```
$ git commit --amend
```

或者，您想将最后三个提交合并为一个，您可以运行如下命令

```
$ git reset --soft HEAD~3
$ git commit
```

如果您想将一个分支中的所有提交合并为一个，您可以运行如下命令：

```
$ git reset --soft main
$ git commit
```

或者，如果您想完全自由地重排和压缩所有提交，您可以使用交互式变基，如下所示：

```
$ git rebase -i main
```

要将更改推送到服务器，请使用如下命令：

```
$ git push origin +<branch>
```

这是之前的示例，填充了分支名：

```
$ git push origin +users/kaufm/42_new-feature
```

请注意分支名称前的 + 加号。这会导致强制推送，但仅限于特定分支。如果你没有弄乱你的分支历史，你可以执行一个普通的 git 推送操作，如果你的分支得到很好的保护并且你知道自己在做什么，那么正常的强制推送可能会更方便，如下所示:

```
$ git push -f
```

如果您已经需要帮助或需要队友对您的代码提出意见，您可以在PR 的评论中提及他们。如果他们想提出变更，可以使用PR 评论中的suggestion功能。这样，您就可以应用他们提出的变更，并且可以确保在执行此操作之前仓库中的状态是干净的。

当您觉得工作已准备就绪，可以将PR的状态从draft修改为ready，并且激活自动合并，如下所示：

```
$ gh pr ready
$ gh pr merge --auto --delete-branch --rebase
```

> 注意
>
> 请注意，我指定了--rebase作为合并方式，对于青睐良好而简洁的提交历史的小型团队来说，这是一种很好的合并策略。如果您更偏爱--squash或--merge，请相应地调整策略。

您的审阅者仍然可以在他们的评论中提出建议，并且您可以继续协作。但是一旦所有批准和所有自动检查完成，PR 将自动合并并删除分支。自动检查在 pull_request 触发器上运行，包括在隔离环境中安装应用程序，以及运行各种测试。

如果您的 PR 已被合并并且分支已被删除，您可以清理本地环境，如下所示：

```
$ git switch main
$ git pull --prune
```

这会将您当前的分支更改为main，从服务器拉取已更改的分支，并删除已经在服务器上完成删除的本地分支。

### 发布

一旦您的更改合并到 main分支上，main分支上的推送触发器将开始部署到生产环境，与您使用的方法是基于环境还是基于环无关。

如果必须维护多个版本，可以将标签与 GitHub 发布一起使用(如我在第 8章“使用 GitHub 包管理依赖项”中展示的那样)。在工作流中使用release触发器并部署应用程序，并使用 GitVersion 自动生成版本号，如下所示:

```
$ gh release create <tag> --notes "<release notes>"
```

示例如下：

```
$ gh release create v1.1 --notes "Added new feature"
```

您还可以利用发布说明的自动生成。遗憾的是，此功能无法通过CLI使用， 您必须使用UI创建发布才能工作。

由于我们无论如何都遵循上游优先原则来修复错误，因此如果我们不必执行修补程序，为每个发布创建发布分支是没有切实好处的。创建发布时生成的标签就够用了。

### 修补程序

如果您必须为旧发布提供修补程序，可以跳转到该标签并创建一个新的修补分支，如下所示：

```
$ git switch -c <hotfix-branch> <tag>
$ git push --set-upstream origin <branch>
```

示例如下：

```
$ git switch -c hotfix/v1.1.1 v1.1
$ git push --set-upstream origin hotfix/1.1.1
```

现在，切换回main分支并修复正常主题分支中的错误（如 users/kaufm/666_fix-bug）。然后，将这条修补提交拣选到修补分支中，如下所示：

```
$ git switch <hotfix-branch>
$ git cherry-pick <commit SHA>
$ git push
```

您可以使用要拣选的提交的安全哈希算法（SHA），或者，如果该分支是最近一次提交的，您也可以使用分支的名称，如下所示：

```
$ git switch hotfix/v1.1.1
$ git cherry-pick users/kaufm/42_fix-bug
$ git push
```

这将拣选主题分支的最近一次提交。图11.6展示了针对旧发布的修补程序是如何工作的：

![fig11-6](./chapter11.assets/fig11-6.png)

Tag: v1.1 Release v1.1                       标签：v1.1.  版本 v1.1

Only create branch if you need it!  hotfix/v1.1.1     仅在需要时创建分！ 修补程序 v1.1.1

Cherry-pick the fix                               拣选修补程序

Tag: v1.1.1 Release v1.1.1                  标签：v1.1.1  版本 v1.1.1

Tag: v1.2.1 Release v1.2.1                  标签：v1.2.1  版本 v1.2.1

Tag: v1.2 Release v1.2                        标签：v1.2.  版本 v1.2

Fix upstream first in a normal topic branch       在正常主题分支中首先修复上游

Figure 11.6 — Performing hotfixes on older releases       图11.6 对旧版本执行修补程序

您也可以先把修补程序合并到main分支中，然后从main分支中拣选这条提交。这确保了代码符合您所有的分支策略。这取决于您的环境有多复杂，以及主分支和修补程序分支之间的差异有多大。

### 自动化
如果您的工作流具有命名约定，那么您会经常使用某些特定的命令序列。为了减少拼写错误并简化您的工作流程，您可以使用 git **aliases**。执行此操作的最佳方法是在您选择的编辑器中编辑您的 . gitconfig文件，如下所示：

```
$ git config --global --edit
```

如果alias尚不存在，则添加一个部分（section），[alias]，并添加一个别名：

```
[alias]
 mfstart = "!f() { \
 git switch -c users/$1/$2_$3 && \
 git commit && \
 git push --set-upstream origin users/$1/$2_$3 && \
 gh pr create --fill --draft; \
 };f"
```

这个别名成为mfstart，将会被调用来指定用户名、问题ID以及主题，如下所示：

```
$ git mfstart kaufm 42 new-feature
```

它切换到一个新分支并提交索引中的当前更改，将它们推送到服务器，并创建一个 PR。

您可以引用单个参数($1、$2、...)或使用$@引用所有参数。如果要独立于退出代码链接命令，则必须使用 ;终止命令。如果您希望下一个命令仅在第一个命令成功时执行，您可以使用&&。请注意，您必须以反斜杠（\）结束每一行，这个字符也可用来转义引号。

您可以添加 if 语句来拆分逻辑，如下所示：

```
mfrelease = "!f() { \
 if [[ -z \"$1\" ]]; then \
 echo Please specify a name for the tag; \
 else \
 gh release create $1 --notes $2; \
 fi; \
};f"
```

或者，您可以将值存储在变量中以供后续使用，如本例所示——您最近一次提交所指向的分支名称为：

```
mfhotfix = "!f() { \
 head=$(git symbolic-ref HEAD --short); \
 echo Cherry-pick $head onto hotfix/$1 && \
 git switch -c hotfix/$1 && \
 git push --set-upstream origin hotfix/$1 && \
 git cherry-pick $head && \
 git push; \
};f"
```

这些只是示例，自动化在很大程度上取决于您工作方式的细节，但它是一个非常强大的工具，可以帮助您提高效率。

## 案例学习

随着发布过程的自动化就绪，两个试点团队已经注意到生产力的巨大提升。**交付时间**和**部署频率**的指标显著增加。

使用 git 的团队从 Bitbucket 迁移到 GitHub 之前，他们使用 **Gitflow** 作为他们的分支工作流。由于他们的web 应用程序可以使用分阶段部署工作流来持续发布，因而转向具有PR和私有分支的**基于主干**的工作流，并在合并后使用 CI/CD 工作流(**MyFlow**)部署到主分支。为了频繁集成，他们决定使用？？**功能发布控制**。由于该公司需要在云端和本地进行功能管理，因此他们决定使用Unleash（是一个功能管理解决方案，用户自定义用户划分的规则，以便控制如何启用新功能）。该团队可以使用软件即服务(SaaS)，并目可以立即开始使用它，而无需等待本地解决方案。

从 **Team Foundation Server (TFS)**迁移的第二个团队已经习惯了复杂的分支工作流程，其中包含长期发布、服务包、修补程序分支和集成了所有功能的开发分支。由于软件安装在硬件产品上，多个版本并行保持稳定状态，还有多个版本需要持续多年的维护。这意味着该软件不能持续发布。团队选择**发布流**来管理发布和修补程序。对于开发，他们还使用了带有 PR 的私有分支和基于主干的方法。由于产品未连接到互联网，该团队依赖其配置系统来获取功能标志，他们曾经使用此技术来测试硬件上的新功能，现在对其进行扩展来更为频繁地集成更改。

## 总结

git工作流彼此之间并没有太大区别，而且大多数工作流都是建立在其他工作流之上的。更重要的是遵循快速失败和前滚的原则，而不是教条化地对待某一个工作流。所有工作流都只是最佳实践的集合，您应该只取您所需。

重要的是更改的大小和合并的频率。

应始终遵循以下规则：

- 始终基于主分支派生主题分支(**基于主干**)
- 如果您正在处理复杂的功能，请确保至少**每天**提交**一次**（使用？？功能发布控制）。
- 如果你的改动很简单，只需要改动几行代码，你可以让你的 PR 开放更长时间。但是请检查您没有太多**开放的 PR**。

有了这些规则，你实际使用的工作流程就没那么重要了。选择对你有用的东西。

在本章中，您了解了 TBD的好处以及如何将其与 git 工作流一起使用以提高工程速度。

在下一章中，我将解释如何使用左移测试来提高质量，更加自信地完成发布。

## 延伸阅读

您可以参阅以下参考资料来获取本章所涵盖主题的更多信息：

- *Forsgren N.*, *Humble, J.*, and *Kim, G.* (2018). *Accelerate: The Science of Lean Software* *and DevOps: Building and Scaling High Performing Technology Organizations* (1st ed.) [E-book]. IT Revolution Press.

- Trunk-based development: https://trunkbaseddevelopment.com
- Gitflow: *Vincent Driessen* (2010), *A successful Git branching model*: https://nvie.com/posts/a-successful-git-branching-model/
- *GitLab flow*: https://docs.gitlab.com/ee/topics/gitlab_flow.html
- *Edward Thomson* (2018). *Release Flow: How We Do Branching on the VSTS Team*: https://devblogs.microsoft.com/devops/release-flow-how-we-do-branching-on-the-vsts-team/
- *Aman Gupta* (2015). *Deploying branches to GitHub.com*: https://github.blog/2015-06-02-deploying-branches-to-github-com/
- *GitHub flow*: https://docs.github.com/en/get-started/quickstart/github-flow
- GitHub CLI: https://cli.github.com/
