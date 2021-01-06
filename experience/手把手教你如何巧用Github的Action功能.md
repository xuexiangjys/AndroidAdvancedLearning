
# 手把手教你如何巧用Github的Action功能

## 概念

[GitHub Actions](https://github.com/features/actions) 是 [GitHub](https://github.com/) 于2018年10月推出的持续集成服务。

那么何谓持续集成呢？

### 持续集成

持续集成（Continuous integration），也就是我们经常说的CI。它是一种软件开发实践，可以让团队在持续的基础上收到反馈并进行改进，不必等到开发后期才寻找和修复缺陷，常运用于软件的敏捷开发中。Jenkins就是我们常用的持续集成平台工具。

理解了持续集成的概念之后，下面我简单讲一下使用持续集成的好处：

* 提高效率，减少了重复性工作：一些重复性的工作写成脚本交给持续集成服务执行。
* 减少了人工带来的错误：机器通过预先写好的脚本执行犯错的几率比人工低很多。
* 减少等待的时间：一套完备的持续集成服务涵盖了开发、集成、测试、部署等各个环节。
* 提高产品质量：很多大公司在代码提交后都会有一套代码检视脚本（俗称门禁）来检查代码的提交是否符合规范，从而从源头遏制问题的产生。

### Actions

相比较持续集成这个大概念，GitHub推出的 Actions 就显得非常轻量和巧妙了。Actions就相当于持续集成中的某个特定功能的脚本，通过多个actions的自由组合，便可实现自己特定功能的持续集成服务。

同时，Github为了方便大家使用 Actions，还专门做了一个 [Actions市场](https://github.com/marketplace?type=actions), 真的是非常方便！

![](https://img.rruu.net/image/5ff5f152d64ce)

GitHub Actions 有一些自己的术语:

* 1.`workflow（工作流程）`：持续集成一次运行的过程，就是一个`workflow`。
* 2.`job（任务）`：一个`workflow`由一个或多个`jobs`构成，含义是一次持续集成的运行，可以完成多个任务。
* 3.`step（步骤）`：每个`job`由多个`step`构成，一步步完成。
* 4.`action（动作）`：每个`step`可以依次执行一个或多个命令（action）。

### workflow文件

GitHub Actions 的配置文件叫做`workflow`文件，存放在代码仓库的`.github/workflows`目录, 如下图所示：

![](https://img.rruu.net/image/5ff5f4a84125b)

`workflow`文件采用`YAML`格式，文件名可以任意取，但是后缀名统一为`.yml`，比如上图的`package.yml`。

`workflow`文件的配置字段非常多，详见[官方文档](https://help.github.com/en/articles/workflow-syntax-for-github-actions) 。下面是一些基本字段：

* 1.`name`: workflow的名称。如果省略该字段，默认为当前workflow的文件名。
* 2.`on`: 触发workflow的条件，通常是某些事件，例如：`release`、`push`、`pull_request`等。详细内容可以参照 [官方文档](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows) 。
* 3.`jobs`: workflow文件的主体内容，表示要执行的一项或多项任务。
    * `jobs.<job_id>.name`: `job_id`是任务的id，`name`是任务的描述。
    * `jobs.<job_id>.runs-on`: `runs-on`运行所需要的虚拟机环境,它是必填字段。
    * `jobs.<job_id>.needs`: `needs`指定当前任务的依赖关系，即运行顺序。
    * `jobs.<job_id>.steps`: `steps`指定每个任务的运行步骤，可以包含一个或多个步骤。
     
## Actions的应用
















