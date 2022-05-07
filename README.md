# DevOps 鉴宝活动说明

获取对DevOps理念和技术的深刻理解，可以通过实操示例项目的方式。我们这个项目，用 DevOps 工具考古鉴宝的直播方式，请各路实战大拿给社区贡献了一些列有趣有料的Codelab，并用直播的方式进行生动的讲解。

"DevOps鉴宝活动“的目标是：面向社区公开的向所有人收集并分享DevOps工具链上各种工具的实操技能。

相关的 DevOps 工具应该包含，但不仅限于以下分类。

- 软件仓库 | code – 管理软件版本的工具–目前使用最广泛的是Git。
- 构建工具 | build – 有些软件在打包或使用前需要编译，传统的构建工具包括Make、Ant、Maven和MSBuild。
- 持续集成工具 | ci – 在配置好以后，每次将代码提交到存储库中时，它都会对软件进行构建、部署和测试。这通常可以提高软件质量和上市时间。这个市场上最流行的工具是 Jenkins、Travis、TeamCity和Bamboo。
- 代码分析/审查工具 | verify – 这些工具可以查找代码中的错误，检查代码格式和质量，以及测试覆盖率。这些工具因编程语言而异。SonarQube是这个领域的一个流行工具，还有其他各种 “轻量的 “工具。
- 配置管理 | config – 配置管理工具和数据库通常存储所有关于你的硬件和软件项目的信息，以及提供一个脚本和/或模板系统，用于自动化常见任务。在这个领域似乎有很多玩家。传统的玩家是Chef、Puppet和Salt Stack。
- 部署工具 | deploy – 这些工具有助于软件的部署。许多CI工具也是CD（持续部署）工具，它们协助软件的部署。传统上在Ruby语言中，Capistrano工具被广泛使用；在Java语言中，Maven被很多人使用。所有的编排工具也都支持某种形式的部署。
- 编排工具 | release – 这些工具配置、调度和管理计算机系统和软件。它们通常将 “自动化 “和 “工作流 “作为其服务的一部分。Kubernetes是一个非常流行的编排工具，它专注于容器。Terraform是一个非常流行的编排工具，它的关注点更广，包括云编排。另外，每个云提供商都有自己的一套工具（CloudFormation、GCP Deployment Manager, 和ARM）。
- 监控工具 | monitor - 这些工具允许监控硬件和软件。通常，它们包括监控代理程序，用于监视进程和日志文件，以确保系统的健康。Nagios是一种流行的监控工具。
- 测试工具 | test - 测试工具用于管理测试，以及测试自动化，包括性能和负载测试等。

![devops tool chain](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2021-04-02-1-wTp-r9QJvF-DXGZDZUHmbA.jpeg)

## 学习清单

以下是所有社区 DevOps 鉴宝活动鉴宝人们分享的codelab 操练直播内容，请大家根据需求学习打卡。社区也欢迎你提交学习过程的延伸学习视频，分享你对某个DevOps 工具某部分知识的解读。

### code


### build     


### test


### verify


### ci        


### 配置管理

* 【第一季】第1集 ：配置管理神器 Ansible - 新手入门指南 [【Codelab】](config/lab01-ansible/readme.md) - [【B 站视频】](https://www.bilibili.com/video/BV1Uv4y1K7u9)


### deploy    


### release   


### monitor   



## 贡献说明

### 目录结构

通过创建 PR 的方式，在以上分类目录中，按编号创建一个新的文件夹，用于容纳一个独立lab的所有相关代码。必须包含清晰的 readme 文件，在 readme 文件中清晰概要的描述相关技术知识要点，不能有意无意的包含任何产品市场宣传的内容。

```sh
├── README.md
├── build
│   └── lab01-ansible
│       └── readme.md
├── ci
├── code
├── config
├── deploy
├── monitor
├── release
├── test
└── verify
```

### 参与贡献过程

本代码库面向所有社区朋友开放。为了让大家能够顺利的参与贡献，请参考以下参与流程。

1. 在代码库的 issue 列表中，用模板创建一个新 issue，简单描述 codelab 的内容，必须包含工具、技术、开发语言和难度级别等，让其他人更清晰的了解知识的范围。
2. Issue 的沟通和确认阶段，社区评审同学和提议者在 issue 的评论区完成讨论确认过程。
3. 进入 codelab 代码的 PR 提交阶段，请创建符合规则的分支名称 【 分类 - 编号  - 工具名称】，例如： code-lab01-git 
4. 进入 PR 搞了确认，社区评审同学会进行评审和测试，并提出修改建议。
5. 提交 PR 的合并请求，本代码库的维护同学审批 MR ，代码正式纳入主干。
6. 进入 Codelab 的学习和宣传阶段，Codelab的提交者和其他鉴宝人，用线上直播的方式演示整个 codelab 的操作过程，并将录播视频在 B 站分享给大家。
7. 关闭 issue 。
8. Codelab 的 readme 文件内容和 B 站视频 会分享给社区的所有人。

## 鸣谢

第一期：

* Red Hat & 极狐 赞助了直播抽奖礼品
* JetBrains 赞助鉴宝人礼品

第二期：

* Elastc & 极狐 赞助了直播抽奖礼品
* JetBrains 赞助鉴宝人礼品

第三期：


第四期：


第五期：

## 行为准则

本项目是面向所有社区参与者的共创学习项目，请任何参与者遵守[《中国DevOps社区行为规范》](https://devopschina.org/zh-hans/code-of-conduct)
