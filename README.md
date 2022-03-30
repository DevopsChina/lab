# DevOps lab

获取对DevOps理念和技术的深刻理解，可以通过实操示例项目的方式。我们用这个项目收集各种有趣有料的学习型项目。

"DevOps实验室“的目标是收集和分享各种深度、技术DevOps工具链的实操练习。

包含，但不仅限于一下分类。

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

以下是所有社区 DevOps lab 贡献者分享的实操练习内容，请大家根据需求学习打卡，欢迎提交学习过程的视频分享，分享你对某个lab的知识解读。

### code


### build     


### test


### verify


### ci        


### config


### deploy    


### release   


### monitor   



## 贡献说明

### 目录结构

在以上分类目录中，按编号创建一个新的文件夹，用于容纳一个独立lab的所有相关代码。必须包含清晰的 readme 文件，在 readme 文件中清晰概要的描述相关技术知识要点，不能有意无意的包含任何产品市场宣传的内容。

```sh
├── README.md
├── build
│   └── lab01
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

本代码库面向所有社区朋友开放。为了让大家能够顺利的参与贡献，请参考一下工作流程。

1. 在议题列表创建一个新 issue，简单描述lab的内容，必须包含工具、技术、开发语言和难度级别等，让其他人更清晰的了解知识的范围。
2. 进入议题沟通和确认阶段，社区评审同学和提议者在 issue 的评论区讨论确认这个议题。
3. 进入lab代码提交阶段，请创建符合规则的分支名称 【 分类 - 编号 - lab 名称】，例如： code-lab01-git-for-beginer 
4. 进入邀请测试阶段，社区会使用公众号等方式，在社区中征集测试学习人员；lab提交者和测试者在分支上沟通反馈建议和可能的问题。
5. 提交合并请求，本代码库的维护同学审批 MR ，代码正式纳入主干。更新本文的学习清单，将其加入合适的位置。
6. 进入 Lab 的学习和宣传阶段，Lab的提交者或者社区中的其他志愿者，用线上直播的方式演示整个lab的操作过程，或者提交自行录制的lab操作流程视频。
7. 关闭 issue 

## 行为准则

本项目是面向所有社区参与者的共创学习项目，请任何参与者遵守[《中国DevOps社区行为规范》](https://devopschina.org/zh-hans/code-of-conduct)


rtmp://live-push.bilivideo.com/live-bvc/

?streamname=live_630300453_61835441&key=77452e490b9d015dd03f4d3478224510&schedule=rtmp&pflag=1

?streamname=live_630300453_61835441&key=77452e490b9d015dd03f4d3478224510&schedule=rtmp&pflag=1