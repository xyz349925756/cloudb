github actions CI 设置

# GitHub Actions 快速入门

1. 如果 `.github/workflows` 目录不存在，请在 GitHub 的仓库中创建此目录。
2. 在 `.github/workflow` 目录中，创建一个名为 `xxx.yml` 的文件。
3. 将以下 YAML 内容复制到 `github-actions-demo.yml` 文件中：

```yaml
name: GitHub Actions Demo
on: [push]
jobs:
  Explore-GitHub-Actions:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."
      - name: Check out repository code
        uses: actions/checkout@v2
      - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
      - run: echo "🖥️ The workflow is now ready to test your code on the runner."
      - name: List files in the repository
        run: |
          ls ${{ github.workspace }}
      - run: echo "🍏 This job's status is ${{ job.status }}."

```

4.滚动到页面底部，然后选择 **Create a new branch for this commit and start a pull request（为此提交创建一个新分支并开始拉取请求）**。 然后，若要创建拉取请求，请单击 **Propose new file（提议新文件）**。

https://docs.github.com/cn/actions/learn-github-actions/understanding-github-actions

https://docs.github.com/cn/actions/learn-github-actions/workflow-syntax-for-github-actions#onpushpull_requestpaths

参数说明

| 标签                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| name: learn-github-actions                                   | *可选* - 将出现在 GitHub 仓库的 Actions（操作）选项卡中的工作流程名称。 |
| on: [push]                                                   | 指定自动触发工作流程文件的事件。 此示例使用 `push` 事件，这样每次有人推送更改到仓库时，作业都会运行。 您可以设置工作流程仅在特定分支、路径或标记上运行。 |
| jobs:                                                        | 将 `learn-github-actions` 工作流程文件中运行的所有作业组合在一起。 |
| check-bats-version:                                          | 定义存储在 `jobs` 部分的 `check-bats-version` 作业的名称。   |
| runs-on: ubuntu-latest                                       | 配置作业在 Ubuntu Linux 运行器上运行。 这意味着该作业将在 GitHub 托管的新虚拟机上执行。 |
| steps:                                                       | 将 `check-bats-version` 作业中运行的所有步骤组合在一起。 此部分下嵌套的每项都是一个单独的操作或 shell 命令。 |
| - uses: actions/checkout@v2                                  | `uses` 关键字指示作业检索名为 `actions/checkout@v2` 的社区操作的 `v2`。 这是检出仓库并将其下载到运行器的操作，允许针对您的代码运行操作（例如测试工具）。 只要工作流程针对仓库的代码运行，或者您使用仓库中定义的操作，您都必须使用检出操作。 |
| - uses: actions/setup-node@v2       with:         <br />  node-version: '14' | 此步骤使用该操作在运行器上安装指定版本的软件包，从而允许您访问命令。‎`actions/setup-node@v2node`npm |
| - run: npm install -g bats                                   | `run` 关键字指示作业在运行器上执行命令。 在这种情况下，使用 `npm` 来安装 `bats` 软件测试包。 |
| - run: bats -v                                               | 最后，您将运行 `bats` 命令，并且带有输出软件版本的参数。     |
|                                                              |                                                              |
|                                                              |                                                              |

<hr>

## 上下文


上下文是一种访问工作流程运行、运行器环境、作业及步骤相关信息的方式。 上下文使用表达式语法。

${大括号 <context> 大括号}

| 上下文名称 | 类型   | 描述                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| github     | object | 工作流程运行的相关信息                                       |
| env        | object | 包含工作流程，作业或步骤中设置的环境变量                     |
| job        | object | 当前执行的作业相关信息                                       |
| steps      | object | 作业中运行的步骤                                             |
| runner     | object | 运行当前作业的运行程序相关信息                               |
| secrets    | object | 启用对密码的访问权限                                         |
| strategy   | object | 用于访问配置的策略参数及当前作业的相关信息。 策略参数包括 `fail-fast`、`job-index`、`job-total` 和 `max-parallel` |
| matrix     | object | 用于访问为当前作业配置的矩阵参数。 例如，如果使用 `os` 和 `node` 版本配置矩阵构建，`matrix` 上下文对象将包含当前作业的 `os` 和 `node` 版本。 |
| needs      | object | 允许访问定义为当前作业依赖项的所有作业的输出。               |

作为表达式的一部分，您可以使用以下两种语法之一访问上下文信息。

- 索引语法：`github['sha']`
- 属性解除参考语法：`github.sha`

要使用属性解除参考语法，属性名称必须：

- 以 `a-Z` 或 `_` 开头。
- 后跟 `a-Z` `0-9` `-` 或 `_`。

**确定何时使用上下文**

GitHub Actions 包含一个称为上下文的变量集和一个称为默认环境变量的类似变量集。 这些变量预期用于工作流程中的不同点：

  **默认环境变量：**这些变量仅存在于执行作业的运行器上。

  **上下文：**您可以在工作流程的任何时候使用大多数上下文，包括当*默认环境变量*不可用时。 例如，您可以使用带表达式的上下文执行初始处理，然后将作业路由到运行器以供执行；这允许您使用带有条件 `if` 关键字的上下文来确定步骤是否应运行。 作业运行后，您还可以从执行作业的运行器（如 `runner.os`）检索上下文变量。















## 语法

### name

==name==  工作流程的名称

### on

https://docs.github.com/cn/actions/learn-github-actions/workflow-syntax-for-github-actions

要想使用actions，先创建工作目录

==.github/workflows/==  在此文件夹下创建yml或yaml文件

==on==  触发事件 https://docs.github.com/cn/actions/learn-github-actions/events-that-trigger-workflows

   on: push   //单一事件

   on: [push,pull_request]   //使用事件列表

---

   on:

​      #注释

​      branches: 分支

​        - main    主分支

​      pull_request

​        - main   主分支

​      page_build:

​      release:

​        types:

​           -created

**`on.<event_name>.types`**

选择将触发工作流程运行的活动类型。 大多数 GitHub 事件由多种活动触发。 例如，发布资源的事件在发行版 `published`、`unpublished`、`created`、`edited`、`deleted` 或 `prereleased` 时触发.通过 `types` 关键词可缩小触发工作流程运行的活动类型的范围。 如果只有一种活动类型可触发 web 挂钩事件，就没有必要使用 `types` 关键词。

```yml
# Trigger the workflow on release activity
on:
  release:
    # Only use the types keyword to narrow down the activity types that will trigger your workflow.
    types: [published, created, edited]
```

`on.<push|pull_request>.<branches|tags>`

使用 `push` 和 `pull_request` 事件时，您可以将工作流配置为在特定分支或标记上运行。 对于 `pull_request` 事件，只评估基础上的分支和标签。 如果只定义 `tags` 或只定义 `branches`，则影响未定义 Git ref 的事件不会触发工作流程运行。

`branches`、`branches-ignore`、`tags` 和 `tags-ignore` 关键词接受使用 `*`、`**`、`+`、`?`、`!` 等字符匹配多个分支或标记名称的 glob 模式。

```yaml
包括分支和标记
on:
  push:
    # Sequence of patterns matched against refs/heads
    branches:    
      # Push events on main branch
      - main
      # Push events to branches matching refs/heads/mona/octocat
      - 'mona/octocat'
      # Push events to branches matching refs/heads/releases/10
      - 'releases/**'
    # Sequence of patterns matched against refs/tags
    tags:        
      - v1             # Push events to v1 tag
      - v1.*           # Push events to v1.0, v1.1, and v1.9 tags
      
忽略分支和标记
on:
  push:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      # Do not push events to branches matching refs/heads/mona/octocat
      - 'mona/octocat'
      # Do not push events to branches matching refs/heads/releases/beta/3-alpha
      - 'releases/**-alpha'
    # Sequence of patterns matched against refs/tags
    tags-ignore:
      - v1.*           # Do not push events to tags v1.0, v1.1, and v1.9

排除分支和标记
    上面的取反即可
    
使用肯定和否定模式
on:
  push:
    branches:    
      - 'releases/**'
      - '!releases/**-alpha'
```

**`on.<push|pull_request>.paths`**

使用 `push` 和 `pull_request` 事件时，您可以将工作流程配置为在至少一个文件不匹配 `paths-ignore` 或至少一个修改的文件匹配配置的 `paths` 时运行。 路径过滤器不评估是否推送到标签。

`paths-ignore` 和 `paths` 关键词接受使用 `*` 和 `**` 通配符匹配多个路径名称的 glob 模式。

```yaml
忽略路径
on:
  push:
    paths-ignore:
      - 'docs/**'
      

包括路径
on:
  push:
    paths:
      - '**.js'
      
 排除路径
 on:
  push:
    paths:
      - '!**.js'
      
 使用肯定和否定
 on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

### Git 差异比较

> **注：** 如果您推送超过 1,000 项提交， 或者如果 GitHub 因超时未生成差异，工作流程将始终运行。

GitHub 会针对推送使用双点差异，针对拉取请求使用三点差异，生成已更改文件列表：

- **拉取请求：** 三点差异比较主题分支的最近版本与其中使用基本分支最新同步主题分支的提交。
- **推送到现有分支：** 双点差异可以直接相互比较头部和基础 SHA。
- **推送到新分支：**根据已推送最深提交的前身父项的两点差异。

差异限制为 300 个文件。 如果更改的文件与过滤器返回的前 300 个文件不匹配，工作流程将不会运行。 您可能需要创建更多的特定过滤器，以便工作流程自动运行。

**`on.workflow_call.inputs`**

```yml
on:
  workflow_call:
    inputs:
      username:
        description: 'A username passed from the caller workflow'   #描述
        default: 'john-doe'   #默认用户
        required: false
        type: string
        
jobs:
  print-username:
    runs-on: ubuntu-latest

    steps:
      - name: Print the input name to STDOUT
        run: echo The username is ${{ inputs.username }}
```

**on.workflow_call.<input_id>.type**

**`on.workflow_call.secrets`**

```yml
on:
  workflow_call:
    secrets:
      access-token:
        description: 'A token passed from the caller workflow'
        required: false

jobs:
  pass-secret-to-action:
    runs-on: ubuntu-latest

    steps:  
      - name: Pass the received secret to an action
        uses: ./.github/actions/my-action@v1
        with:
          token: ${da'kuo secrets.access-token }}
```

**on.workflow_call.secrets.<secret_id>**

**on.workflow_call.secrets.<secret_id>.required**

**on.workflow_dispatch.inputs**

```yaml
on: 
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'     
        required: true
        default: 'warning'
      tags:
        description: 'Test scenario tags'
        required: false
```

工作流程调度输入的格式与操作输入相同。触发的工作流程接收 `github.event.input` 上下文中的输入。

**on.schedule**

```yml
on:
  schedule:
    # * is a special character in YAML so you have to quote this string
    - cron:  '30 5,17 * * *'
```

可用的作用域和访问权限值：

> permissions:
>     actions: read|write|none
>     checks: read|write|none
>     contents: read|write|none
>     deployments: read|write|none
>     id_token: read|write|none
>     issues: read|write|none
>     discussions: read|write|none
>     packages: read|write|none
>     pull-requests: read|write|none
>     repository-projects: read|write|none
>     security-events: read|write|none
>     statuses: read|write|none

```yaml
permissions: read-all|write-all  #定义所有可用作用域的读取或写入权限

--------------------------
name: "My workflow"
on: [ push ]
permissions: read-all
jobs:
  ...
```

### `env`

环境变量的 `map` 可用于工作流程中所有作业的步骤。 您还可以设置仅适用于单个作业的步骤或单个步骤的环境变量。

当多个环境变量使用相同的名称定义时，GitHub 会使用最特定的环境变量。

```yml
env:
  SERVER: production
```

**defaults**

**defaults.run**

您可以为工作流程中的所有 [`run`](https://docs.github.com/cn/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepsrun) 步骤提供默认的 `shell` 和 `working-directory` 选项。 您也可以设置只可用于作业的 `run` 默认设置。

```yml
defaults:
  run:
    shell: bash
    working-directory: scripts
```

**concurrency**

Concurrency 确保只有使用相同并发组的单一作业或工作流程才会同时运行。 并发组可以是任何字符串或表达式。

```yml
使用并发和默认行为
concurrency: staging_environment
concurrency: ci-${{ github.ref }}

使用并发取消任何当前作业或运行
concurrency: 
  group: ${{ github.head_ref }}
  cancel-in-progress: true
```

### jobs

工作流程运行包括一项或多项作业。 作业默认是并行运行。 要按顺序运行作业，您可以使用 `<job_id>needs` 关键词在其他作业上定义依赖项。

每个作业在 `runs-on` 指定的运行器环境中运行。

在工作流程的使用限制之内可运行无限数量的作业。

**jobs.<job_id>**

键值 job_id 是一个字符串，其值是作业配置数据的映像。 必须将 <job_id> 替换为 jobs 对象唯一的字符串。 <job_id> 必须以字母或 _ 开头，并且只能包含字母数字字符、- 或 _。

```yml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

**jobs.<job_id>.name** 作业显示在 GitHub 上的名称。

**jobs.<job_id>.needs**

识别在此作业运行之前必须成功完成的任何作业。 它可以是一个字符串，也可以是字符串数组。 如果某个作业失败，则所有需要它的作业都会被跳过，除非这些作业使用让该作业继续的条件表达式。

```yml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

在此示例中，`job1` 必须在 `job2` 开始之前成功完成，而 `job3` 要等待 `job1` 和 `job2` 完成。

```yml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

在此示例中，job3使用 `always()` 条件表达式，因此它始终在 `job1` 和 `job2` 完成后运行，不管它们是否成功。

**jobs.<job_id>.runs-on**

必填。 要运行作业的机器类型。机器可以是 GitHub 托管的运行器或自托管的运行器。

#### **GitHub 托管的运行器**

如果使用 GitHub 托管的运行器，每个作业将在 `runs-on` 指定的虚拟环境的新实例中运行。

可用的 GitHub 托管的运行器类型包括：

| 虚拟环境             | yaml工作流程                     |
| -------------------- | -------------------------------- |
| Windows Server 2022  | windows-2022                     |
| Windows Server 2019  | windows-latest` 或 `windows-2019 |
| Windows Server 2016  | windows-2016                     |
| Ubuntu 20.04         | ubuntu-latest` 或 `ubuntu-20.04  |
| Ubuntu 18.04         | ubuntu-18.04                     |
| macOS Big Sur 11     | macos-11                         |
| macOS Catalina 10.15 | macos-latest` 或 `macos-10.15    |

```yml
runs-on: ubuntu-latest
```

#### 自托管运行器

要为工作指定自托管的运行器，请在工作流程文件中使用自托管运行器标签配置 `runs-on`

所有自托管运行器都有 `self-hosted` 标签。 仅使用此标签将选择任何自托管运行器。

```yml
runs-on: [self-hosted, linux]
```

**jobs.<job_id>.permissions**

```yml
jobs:
  stale:
    runs-on: ubuntu-latest

    permissions:
      issues: write
      pull-requests: write

    steps:
      - uses: actions/stale@v3
```

**jobs.<job_id>.environment**

作业引用的环境。 在将引用环境的作业发送到运行器之前，必须通过所有环境保护规则

```yml
使用单一环境名称
environment: staging_environment

使用环境名称和 URL 的
environment:
  name: production_environment
  url: https://github.com

url可为表达式
environment:
  name: production_environment
  url: ${{ steps.step_id.outputs.url_output }}
```

**jobs.<job_id>.concurrency**

==>>>====在作业级别指定并发时，无法保证在 5 分钟内排队的作业或运行的互相顺序。==

Concurrency 确保只有使用相同并发组的单一作业或工作流程才会同时运行。 并发组可以是任何字符串或表达式。 表达式可以使用除 secrets 上下文以外的任何上下文。

当并发作业或工作流程排队时，如果仓库中使用同一并发组的其他作业或工作流程正在运行，则排队的作业或工作流程将 pending。 在并发组中任何先前挂起的作业或工作流程都将被取消。 如果还要取消同一并发组中任何当前运行的作业或工作流程，请指定 cancel-in-progress: true。

```yml
使用并发和默认行为
concurrency: staging_environment
concurrency: ci-${{ github.ref }}

使用并发取消任何当前作业或运行
concurrency: 
  group: ${{ github.head_ref }}
  cancel-in-progress: true  
```

**jobs.<job_id>.outputs**

作业输出是字符串，当每个作业结束时，在运行器上评估包含表达式的作业输出。 包含密码的输出在运行器上编辑，不会发送至 GitHub Actions,要在依赖的作业中使用作业输出, 您可以使用 `needs` 上下文

```yml
jobs:
  job1:
    runs-on: ubuntu-latest
    # Map a step output to a job output
    outputs:
      output1: ${{ steps.step1.outputs.test }}
      output2: ${{ steps.step2.outputs.test }}
    steps:
      - id: step1
        run: echo "::set-output name=test::hello"
      - id: step2
        run: echo "::set-output name=test::world"
  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - run: echo ${{needs.job1.outputs.output1}} ${{needs.job1.outputs.output2}}
```

**jobs.<job_id>.env**

环境变量的 map 可用于作业中的所有步骤。 您也可以设置整个工作流程或单个步骤的环境变量。当多个环境变量使用相同的名称定义时，GitHub 会使用最特定的环境变量。 例如，步骤中定义的环境变量在步骤执行时将覆盖名称相同的作业和工作流程变量。 为作业定义的变量在作业执行时将覆盖名称相同的工作流程变量。

```yml
jobs:
  job1:
    env:
      FIRST_NAME: Mona
```

**jobs.<job_id>.defaults**

将应用到作业中所有步骤的默认设置的 map。 您也可以设置整个工作流程的默认设置。

**jobs.<job_id>.defaults.run**

为作业中的所有 run 步骤提供默认的 shell 和 working-directory。 此部分不允许上下文和表达式。

```yml
jobs:
  job1:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: scripts
在作业中定义的默认设置将覆盖在工作流程中定义的同名默认设置。        
```

**jobs.<job_id>.if**

您可以使用 if 条件阻止作业在条件得到满足之前运行。 您可以使用任何支持上下文和表达式来创建条件。

在 if 条件下使用表达式时，可以省略表达式语法 (${{ }})，因为 GitHub 会自动将 if 条件作为表达式求值。

### steps

**jobs.<job_id>.steps**

作业包含一系列任务，称为 steps。 步骤可以运行命令、运行设置任务，或者运行您的仓库、公共仓库中的操作或 Docker 注册表中发布的操作。 并非所有步骤都会运行操作，但所有操作都会作为步骤运行。 每个步骤在运行器环境中以其自己的进程运行，且可以访问工作区和文件系统。 因为步骤以自己的进程运行，所以步骤之间不会保留环境变量的更改。 GitHub 提供内置的步骤来设置和完成作业。在工作流程的使用限制之内可运行无限数量的步骤。

```yml
name: Greeting from Mona

on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
      - name: Print a greeting
        env:
          MY_VAR: Hi there! My name is
          FIRST_NAME: Mona
          MIDDLE_NAME: The
          LAST_NAME: Octocat
        run: |
          echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

**jobs.<job_id>.steps[*].id**

步骤的唯一标识符。 您可以使用 id 引用上下文中的步骤。

**jobs.<job_id>.steps[*].if**

您可以使用 if 条件阻止步骤在条件得到满足之前运行。 您可以使用任何支持上下文和表达式来创建条件。

在 if 条件下使用表达式时，可以省略表达式语法 (${{ }})，因为 GitHub 会自动将 if 条件作为表达式求值。

```yml
使用上下文
此步骤仅在事件类型为 pull_request 并且事件操作为 unassigned 时运行。
steps:
 - name: My first step
   if: ${{ github.event_name == 'pull_request' && github.event.action == 'unassigned' }}
   run: echo This event is a pull request that had an assignee removed.
   
使用状态检查功能
my backup step 仅在作业的上一步失败时运行
steps:
  - name: My first step
    uses: octo-org/action-name@main
  - name: My backup step
    if: ${{ failure() }}
    uses: actions/heroku@1.0.0
```

**jobs.<job_id>.steps[*].name**

步骤显示在 GitHub 上的名称

**jobs.<job_id>.steps[*].uses**

选择要作为作业中步骤的一部分运行的操作。 操作是一种可重复使用的代码单位。 您可以使用工作流程所在仓库中、公共仓库中或发布 Docker 容器映像中定义的操作。

强烈建议指定 Git ref、SHA 或 Docker 标记编号来包含所用操作的版本。 如果不指定版本，在操作所有者发布更新时可能会中断您的工作流程或造成非预期的行为。

- 使用已发行操作版本的 SHA 对于稳定性和安全性是最安全的。
- 使用特定主要操作版本可在保持兼容性的同时接收关键修复和安全补丁。 还可确保您的工作流程继续工作。
- 使用操作的默认分支可能很方便，但如果有人新发布具有突破性更改的主要版本，您的工作流程可能会中断。

有些操作要求必须通过 [`with`](https://docs.github.com/cn/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepswith) 关键词设置输入。

```yml
使用版本化操作
steps:
  # Reference a specific commit
  - uses: actions/checkout@a81bbbf8298c0fa03ea29cdc473d45769f953675
  # Reference the major version of a release
  - uses: actions/checkout@v2
  # Reference a specific version
  - uses: actions/checkout@v2.2.0
  # Reference a branch
  - uses: actions/checkout@main
  
使用公共操作  
{owner}/{repo}@{ref}
您可以指定公共 GitHub 仓库中的分支、引用或 SHA
jobs:
  my_first_job:
    steps:
      - name: My first step
        # Uses the default branch of a public repository
        uses: actions/heroku@main
      - name: My second step
        # Uses a specific version tag of a public repository
        uses: actions/aws@v2.0.1
        
在子目录中使用公共操作
{owner}/{repo}/{path}@{ref}
公共 GitHub 仓库中特定分支、引用或 SHA 上的子目录
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/aws/ec2@main
        
使用工作流程所在仓库中操作  
./path/to/dir
包含工作流程的仓库中操作的目录路径。 在使用操作之前，必须检出仓库。
jobs:
  my_first_job:
    steps:
      - name: Check out repository
        uses: actions/checkout@v2
      - name: Use local my-action
        uses: ./.github/actions/my-action

使用 Docker 中枢操作
docker://{image}:{tag}
Docker 中枢上发布的 Docker 映像
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://alpine:3.8

使用 GitHub Packages Container registry
docker://{host}/{image}:{tag}
GitHub Packages Container registry 中的 Docker 映像
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://ghcr.io/OWNER/IMAGE_NAME

使用 Docker 公共注册表操作
docker://{host}/{image}:{tag}
公共注册表中的 Docker 映像。 此示例在 gcr.io 使用 Google Container Registry
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://gcr.io/cloud-builders/gradle

在不同于工作流程的私有仓库中使用操作
您的工作流程必须检出私有仓库，并在本地引用操作。 生成个人访问令牌并将该令牌添加为加密密钥。
将示例中的 PERSONAL_ACCESS_TOKEN 替换为您的密钥名称。
jobs:
  my_first_job:
    steps:
      - name: Check out repository
        uses: actions/checkout@v2
        with:
          repository: octocat/my-private-repo
          ref: v1.0
          token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          path: ./.github/actions/my-private-repo
      - name: Run my action
        uses: ./.github/actions/my-private-repo/my-action
```

创建个人令牌https://docs.github.com/cn/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

加密密码：https://docs.github.com/cn/actions/security-guides/encrypted-secrets

### run

**jobs.<job_id>.steps[*].run**

使用操作系统 shell 运行命令行程序。 如果不提供 `name`，步骤名称将默认为 `run` 命令中指定的文本。

命令默认使用非登录 shell 运行。 您可以选择不同的 shell，也可以自定义用于运行命令的 shell。

每个 `run` 关键词代表运行器环境中一个新的进程和 shell。 当您提供多行命令时，每行都在同一个 shell 中运行。 

```yml
单行命令
- name: Install Dependencies
  run: npm install
  
多行命令
- name: Clean install dependencies and build
  run: |
    npm ci
    npm run build

使用 working-directory 关键词，您可以指定运行命令的工作目录位置
- name: Clean temp directory
  run: rm -rf *
  working-directory: ./temp
```

#### 使用指定 shell

您可以使用 `shell` 关键词覆盖运行器操作系统中默认的 shell 设置。 您可以使用内置的 `shell` 关键词，也可以自定义 shell 选项集。

| 支持的平台    | `shell` 参数 | 描述                                                         | 内部运行命令                                    |
| :------------ | :----------- | :----------------------------------------------------------- | :---------------------------------------------- |
| 所有          | `bash`       | 非 Windows 平台上回退到 `sh` 的默认 shell。 指定 Windows 上的 bash shell 时，将使用 Git for Windows 随附的 bash shel。 | `bash --noprofile --norc -eo pipefail {0}`      |
| 所有          | `pwsh`       | PowerShell Core。 GitHub 将扩展名 `.ps1` 附加到您的脚本名称。 | `pwsh -command ". '{0}'"`                       |
| 所有          | `python`     | 执行 python 命令。                                           | `python {0}`                                    |
| Linux / macOS | `sh`         | 未提供 shell 且 在路径中找不到 `bash` 时的非 Windows 平台的后退行为。 | `sh -e {0}`                                     |
| Windows       | `cmd`        | GitHub 将扩展名 `.cmd` 附加到您的脚本名称并替换 `{0}`。      | `%ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}""`. |
| Windows       | `pwsh`       | 这是 Windows 上使用的默认 shell。 PowerShell Core。 GitHub 将扩展名 `.ps1` 附加到您的脚本名称。 如果自托管的 Windows 运行器没有安装 *PowerShell Core*，则使用 *PowerShell Desktop* 代替。 | `pwsh -command ". '{0}'"`.                      |
| Windows       | `powershell` | PowerShell 桌面。 GitHub 将扩展名 `.ps1` 附加到您的脚本名称。 | `powershell -command ". '{0}'"`.                |

```yml
使用 bash 运行脚本
steps:
  - name: Display the path
    run: echo $PATH
    shell: bash

使用 Windows cmd 运行脚本
steps:
  - name: Display the path
    run: echo %PATH%
    shell: cmd
    
使用 PowerShell Core 运行脚本
steps:
  - name: Display the path
    run: echo ${env:PATH}
    shell: pwsh

使用 PowerShell 桌面运行脚本
steps:
  - name: Display the path
    run: echo ${env:PATH}
    shell: powershell
    
运行 python 脚本
steps:
  - name: Display the path
    run: |
      import os
      print(os.environ['PATH'])
    shell: python
```

#### 自定义 shell

您可以使用 command […options] {0} [..more_options] 将 shell 值设置为模板字符串。 GitHub 将字符串的第一个用空格分隔的词解释为命令，并在 {0} 处插入临时脚本的文件名。

```yml
此示例中使用的命令 perl 必须安装在运行器上
steps:
  - name: Display the environment variables and their values
    run: |
      print %ENV
    shell: perl {0}
```

#### 文件系统

GitHub 在虚拟机上的特定目录中执行操作和 shell 命令。 虚拟机上的文件路径不是静态的。 使用环境变量 GitHub 提供 `home`、`workspace` 和 `workflow` 目录的构建文件路径。

| 目录                  | 环境变量            | 描述                                                         |
| :-------------------- | :------------------ | :----------------------------------------------------------- |
| `home`                | `HOME`              | 包含用户相关的数据。 例如，此目录可能包含登录凭据。          |
| `workspace`           | `GITHUB_WORKSPACE`  | 在此目录中执行操作和 shell 命令。 操作可以修改此目录的内容，后续操作可以访问这些修改。 |
| `workflow/event.json` | `GITHUB_EVENT_PATH` | 触发工作流程的 web 挂钩事件的 `POST` 有效负载。 每当操作执行时，GitHub 都会重写此变量，以隔离操作之间的文件内容。 |

#### Docker 容器文件系统

在 Docker 容器中运行的操作在 `/github` 路径下有静态目录。 但强烈建议使用默认环境变量在 Docker 容器中构建文件路径。

GitHub 保留 `/github` 路径前缀，并为操作创建三个目录。

- `/github/home`
- `/github/workspace` - **注：**GitHub Actions 必须由默认 Docker 用户 (root) 运行。 确保您的 Dockerfile 未设置 `USER` 指令，否则您将无法访问 `GITHUB_WORKSPACE`。
- `/github/workflow`

#### 退出代码和错误操作首选项

至于内置的 shell 关键词，我们提供由 GitHub 托管运行程序执行的以下默认值。 在运行 shell 脚本时，您应该使用这些指南。

- `bash`/`sh`：

  - 使用 `set -eo pipefail` 的快速失败行为：`bash` 和内置 `shell` 的默认值。 它还是未在非 Windows 平台上提供选项时的默认值。

  - 您可以向 shell 选项提供模板字符串，以退出快速失败并接管全面控制权。 例如 `bash {0}`。

  - sh 类 shell 使用脚本中最后执行的命令的退出代码退出，也是操作的默认行为。 运行程序将根据此退出代码将步骤的状态报告为失败/成功。

    

- `powershell`/`pwsh`

  - 可能时的快速失败行为。 对于 `pwsh` 和 `powershell` 内置 shell，我们将 `$ErrorActionPreference = 'stop'` 附加到脚本内容。

  - 我们将 `if ((Test-Path -LiteralPath variable:\LASTEXITCODE)) { exit $LASTEXITCODE }` 附加到 powershell 脚本，以使操作状态反映脚本的最后一个退出代码。

  - 用户可随时通过不使用内置 shell 并提供类似如下的自定义 shell 选项来退出：`pwsh -File {0}` 或 `powershell -Command "& '{0}'"`，具体取决于需求。

    

- `cmd`

  - 除了编写脚本来检查每个错误代码并相应地响应之外，似乎没有办法完全选择快速失败行为。 由于我们默认不能实际提供该行为，因此您需要将此行为写入脚本。
  - `cmd.exe` 在退出时带有其执行的最后一个程序的错误等级，并且会将错误代码返回到运行程序。 此行为在内部与上一个 `sh` 和 `pwsh` 默认行为一致，是 `cmd.exe` 的默认值，所以此行为保持不变。

### with

**`jobs.<job_id>.steps[*].with`**

输入参数的 `map` 由操作定义。 每个输入参数都是一个键/值对。 输入参数被设置为环境变量。 该变量的前缀为 `INPUT_`，并转换为大写。

```yml
定义 hello_world 操作所定义的三个输入参数（first_name、middle_name 和 last_name）。 这些输入变量将被 hello-world 操作作为 INPUT_FIRST_NAME、INPUT_MIDDLE_NAME 和 INPUT_LAST_NAME 环境变量使用

jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/hello_world@main
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat 
```

**jobs.<job_id>.steps[*].with.args**

string 定义 Docker 容器的输入。 GitHub 在容器启动时将 args 传递到容器的 ENTRYPOINT。 此参数不支持 array of strings

```yml
steps:
  - name: Explain why this job ran
    uses: octo-org/action-name@main
    with:
      entrypoint: /bin/echo
      args: The ${{ github.event_name }} event triggered this step.
```

args 用来代替 Dockerfile 中的 CMD 指令。 如果在 Dockerfile 中使用 CMD，请遵循按偏好顺序排序的指导方针：

1.在操作的自述文件中记录必要的参数，并在 CMD 指令的中忽略它们。
2.使用默认值，允许不指定任何 args 即可使用操作。
3.如果操作显示 --help 标记或类似项，请将其用作默认值，以便操作自行记录。

`jobs.<job_id>.steps[*].with.entrypoint`

覆盖 Dockerfile 中的 Docker ENTRYPOINT，或在未指定时设置它。 与包含 shell 和 exec 表单的 Docker ENTRYPOINT 指令不同，entrypoint 关键词只接受定义要运行的可执行文件的单个字符串。

```yml
steps:
  - name: Run a custom command
    uses: octo-org/action-name@main
    with:
      entrypoint: /a/different/executable
entrypoint 关键词旨在用于 Docker 容器操作，但您也可以将其用于未定义任何输入的 JavaScript 操作。
```

**jobs.<job_id>.steps[*].env**

设置供步骤用于运行器环境的环境变量。 您也可以设置整个工作流程或某个作业的环境变量。当多个环境变量使用相同的名称定义时，GitHub 会使用最特定的环境变量。例如，步骤中定义的环境变量在步骤执行时将覆盖名称相同的作业和工作流程变量。 为作业定义的变量在作业执行时将覆盖名称相同的工作流程变量。

公共操作可在自述文件中指定预期的环境变量。 如果要在环境变量中设置密码，必须使用 `secrets` 上下文进行设置。

```yml
steps:
  - name: My first action
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      FIRST_NAME: Mona
      LAST_NAME: Octocat
```

**jobs.<job_id>.steps[*].continue-on-error**

防止步骤失败时作业也会失败。 设置为 `true` 以允许在此步骤失败时作业能够通过。

**jobs.<job_id>.steps[*].timeout-minutes**

终止进程之前运行该步骤的最大分钟数

**jobs.<job_id>.timeout-minutes**

在 GitHub 自动取消运行之前可让作业运行的最大分钟数。 默认值：360如果超时超过运行器的作业执行时限，作业将在达到执行时限时取消。

### strategy

**jobs.<job_id>.strategy**

策略创建作业的构建矩阵。 您可以定义要在其中运行每项作业的不同变种。

**jobs.<job_id>.strategy.matrix**

您可以定义不同作业配置的矩阵。 矩阵允许您通过在单个作业定义中执行变量替换来创建多个作业。 例如，可以使用矩阵为多个受支持的编程语言、操作系统或工具版本创建作业。 矩阵重新使用作业的配置，并为您配置的每个矩阵创建作业。

作业矩阵在每次工作流程运行时最多可生成 256 个作业。 此限制也适用于自托管运行器。

您在 `matrix` 中定义的每个选项都有键和值。 定义的键将成为 `matrix` 上下文中的属性，您可以在工作流程文件的其他区域中引用该属性。 例如，如果定义包含操作系统数组的键 `os`，您可以使用 `matrix.os` 属性作为 `runs-on` 关键字的值，为每个操作系统创建一个作业。

定义 `matrix` 事项的顺序。 定义的第一个选项将是工作流程中运行的第一个作业

```yml
运行多个版本的 Node.js
strategy:
  matrix:
    node: [10, 12, 14]
steps:
  # Configures the node version used on GitHub-hosted runners
  - uses: actions/setup-node@v2
    with:
      # The Node.js version to configure
      node-version: ${{ matrix.node }}

此示例通过设置三个 Node.js 版本阵列的 node 键创建三个作业的矩阵。 为使用矩阵，示例将 matrix.node 上下文属性设置为 setup-node 操作的输入参数 node-version。 因此，将有三个作业运行，每个使用不同的 Node.js 版本.setup-node 操作是在使用 GitHub 托管的运行器时建议用于配置 Node.js 版本的方式。


使用多个操作系统运行
您可以创建矩阵以在多个运行器操作系统上运行工作流程。 您也可以指定多个矩阵配置。 此示例创建包含 6 个作业的矩阵：
在 os 阵列中指定了 2 个操作系统
在 node 阵列中指定了 3 个 Node.js 版本
定义操作系统矩阵时，必须将 runs-on 的值设置为您定义的 matrix.os 上下文属性。
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    os: [ubuntu-18.04, ubuntu-20.04]
    node: [10, 12, 14]
steps:
  - uses: actions/setup-node@v2
    with:
      node-version: ${{ matrix.node }}
      
在组合中包含附加值
您可以将额外的配置选项添加到已经存在的构建矩阵作业中。 例如，如果要在作业使用 windows-latest 和 node 的版本 8 运行时使用 npm 的特定版本，您可以使用 include 指定该附加选项。
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    os: [macos-latest, windows-latest, ubuntu-18.04]
    node: [8, 10, 12, 14]
    include:
      # includes a new variable of npm with a value of 6
      # for the matrix leg matching the os and version
      - os: windows-latest
        node: 8
        npm: 6

包括新组合
您可以使用 include 将新作业添加到构建矩阵中。 任何不匹配包含配置都会添加到矩阵中。 例如，如果您想要使用 node 版本 14 在多个操作系统上构建，但在 Ubuntu 上需要一个使用节点版本 15 的额外实验性作业，则可使用 include 指定该额外作业。
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    node: [14]
    os: [macos-latest, windows-latest, ubuntu-18.04]
    include:
      - node: 15
        os: ubuntu-18.04
        experimental: true

从矩阵中排除配置
您可以使用 exclude 选项删除构建矩阵中定义的特定配置。 使用 exclude 删除由构建矩阵定义的作业。 作业数量是您提供的数组中所包括的操作系统 (os) 数量减去所有减项 (exclude) 后的叉积。
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    os: [macos-latest, windows-latest, ubuntu-18.04]
    node: [8, 10, 12, 14]
    exclude:
      # excludes node 8 on macOS
      - os: macos-latest
        node: 8
```

**注意：**所有 `include` 组合在 `exclude` 后处理。 这允许您使用 `include` 添加回以前排除的组合。

#### 在矩阵中使用环境变量

可以使用 `include` 键为每个测试组合添加自定义环境变量。 然后，您可以在后面的步骤中引用自定义环境变量。

在此示例中， `node-version` 的矩阵条目每个都被配置为对 `site` 和 `datacenter` 环境变量使用不同的值。 `Echo site details` 步骤然后使用 `env: ${{ matrix.env }}` 引用自定义变量：

```yml
name: Node.js CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
       include:
         - node-version: 10.x
           site: "prod"
           datacenter: "site-a"
         - node-version: 12.x
           site: "dev"
           datacenter: "site-b"
    steps:
      - name: Echo site details
        env:
          SITE: ${{ matrix.site }}
          DATACENTER: ${{ matrix.datacenter }}
        run: echo $SITE $DATACENTER
```

**jobs.<job_id>.strategy.fail-fast**

设置为 true 时，如果任何 matrix 作业失败，GitHub 将取消所有进行中的作业。 默认值：true

**jobs.<job_id>.strategy.max-parallel**

使用 matrix 作业策略时可同时运行的最大作业数。 默认情况下，GitHub 将最大化并发运行的作业数量，具体取决于 GitHub 托管虚拟机上可用的运行程序。

```yml
strategy:
  max-parallel: 2
```



### continue

**jobs.<job_id>.continue-on-error**

防止工作流程运行在作业失败时失败。 设置为 `true` 以允许工作流程运行在此作业失败时通过。

```yml
防止特定失败的矩阵作业无法运行工作流程
runs-on: ${{ matrix.os }}
continue-on-error: ${{ matrix.experimental }}
strategy:
  fail-fast: false
  matrix:
    node: [13, 14]
    os: [macos-latest, ubuntu-18.04]
    experimental: [false]
    include:
      - node: 15
        os: ubuntu-18.04
        experimental: true
您可以允许作业矩阵中的特定任务失败，但工作流程运行不失败。 例如， 只允许 node 设置为 15 的实验性作业失败，而不允许工作流程运行失败。        
```

**jobs.<job_id>.container**

用于运行作业中尚未指定容器的任何步骤的容器。 如有步骤同时使用脚本和容器操作，则容器操作将运行为同一网络上使用相同卷挂载的同级容器。

若不设置 container，所有步骤将直接在 runs-on 指定的主机上运行，除非步骤引用已配置为在容器中运行的操作。

```yml
jobs:
  my_job:
    container:
      image: node:14.16
      env:
        NODE_ENV: development
      ports:
        - 80
      volumes:
        - my_docker_volume:/volume_mount
      options: --cpus 1
      
只指定容器映像时，可以忽略 image 关键词
jobs:
  my_job:
    container: node:14.16
```

**jobs.<job_id>.container.image**

要用作运行操作的容器的 Docker 镜像。

**jobs.<job_id>.container.credentials**

如果映像的容器注册表需要身份验证才能拉取映像，可以使用 `credentials` 设置 `username` 和 `password` 的 `map`。 凭据与您提供给 [`Docker 登录`](https://docs.docker.com/engine/reference/commandline/login/) 命令的值相同。

```yml
container:
  image: ghcr.io/owner/image
  credentials:
     username: ${{ github.actor }}
     password: ${{ secrets.ghcr_token }}
```

**jobs.<job_id>.container.env**

设置容器中环境变量的 `map`

**jobs.<job_id>.container.ports**

设置要在容器上显示的端口 `array`

**jobs.<job_id>.container.volumes**

设置要使用的容器卷的 `array`。 您可以使用卷分享作业中服务或其他步骤之间的数据。 可以指定命名的 Docker 卷、匿名的 Docker 卷或主机上的绑定挂载。

要指定卷，需指定来源和目标路径：<source>:<destinationPath>

`<source>` 是主机上的卷名称或绝对路径，`<destinationPath>` 是容器中的绝对路径

```yml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

**jobs.<job_id>.container.options**

附加 Docker 容器资源选项 docker create :https://docs.docker.com/engine/reference/commandline/create/#options

> 不支持 `--network` 选项

### services

**jobs.<job_id>.services**

> 如果您的工作流程使用 Docker 容器操作或服务容器，则必须使用 Linux 运行器：
>
> - 如果您要使用 GitHub 托管的运行器，则必须使用 Ubuntu 运行器。
> - 如果您要使用自托管运行器，则必须使用 Linux 机器作为运行器，并且必须安装 Docker。

用于为工作流程中的作业托管服务容器。 服务容器可用于创建数据库或缓存服务（如 Redis）。 运行器自动创建 Docker 网络并管理服务容器的生命周期。

如果将作业配置为在容器中运行，或者步骤使用容器操作，则无需映射端口来访问服务或操作。 Docker 会自动在同一个 Docker 用户定义的桥接网络上的容器之间显示所有端口。 您可以直接引用服务容器的主机名。 主机名自动映射到为工作流程中的服务配置的标签名称。

如果配置作业直接在运行器机器上运行，且您的步骤不使用容器操作，则必须将任何必需的 Docker 服务容器端口映射到 Docker 主机（运行器机器）。 您可以使用 localhost 和映射的端口访问服务容器。

```yml
使用 localhost
services:
  nginx:
    image: nginx
    # Map port 8080 on the Docker host to port 80 on the nginx container
    ports:
      - 8080:80
  redis:
    image: redis
    # Map TCP port 6379 on Docker host to a random free port on the Redis container
    ports:
      - 6379/tcp

此示例创建分别用于 nginx 和 redis 的两项服务。 指定 Docker 主机端口但不指定容器端口时，容器端口将随机分配给空闲端口。 GitHub 在 ${{job.services.<service_name>.ports}} 上下文中设置分配的容器端口。 在此示例中，可以使用 ${{ job.services.nginx.ports['8080'] }} 和 ${{ job.services.redis.ports['6379'] }} 上下文访问服务容器端口。
```

**jobs.<job_id>.services.<service_id>.image**

要用作运行操作的服务容器的 Docker 镜像

**jobs.<job_id>.services.<service_id>.credentials**

如果映像的容器注册表需要身份验证才能拉取映像，可以使用 `credentials` 设置 `username` 和 `password` 的 `map`。 凭据与您提供给 [`Docker 登录`](https://docs.docker.com/engine/reference/commandline/login/) 命令的值相同。

```yml
services:
  myservice1:
    image: ghcr.io/owner/myservice1
    credentials:
      username: ${{ github.actor }}
      password: ${{ secrets.ghcr_token }}
  myservice2:
    image: dockerhub_org/myservice2
    credentials:
      username: ${{ secrets.DOCKER_USER }}
      password: ${{ secrets.DOCKER_PASSWORD }}
```

**jobs.<job_id>.services.<service_id>.env**

在服务容器中设置环境变量的 `map`。

**jobs.<job_id>.services.<service_id>.ports**

设置要在服务容器上显示的端口 `array`

**jobs.<job_id>.services.<service_id>.volumes**

设置要使用的服务容器卷的 `array`。 您可以使用卷分享作业中服务或其他步骤之间的数据。 可以指定命名的 Docker 卷、匿名的 Docker 卷或主机上的绑定挂载。

要指定卷，需指定来源和目标路径：`<source>:<destinationPath>`.

`<source>` 是主机上的卷名称或绝对路径，`<destinationPath>` 是容器中的绝对路径。

```yml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

**jobs.<job_id>.services.<service_id>.options**

附加 Docker 容器资源选项。不支持 `--network` 选项

### uses

**jobs.<job_id>.uses**

作为作业运行的可重用工作流文件的位置和版本

{owner}/{repo}/{path}/{filename}@{ref}

{ref} 可以是 SHA、发布标签或分支名称。 使用提交 SHA 是最安全的稳定性和安全性。

```yml
jobs:
  call-workflow-1:
    uses: octo-org/this-repo/.github/workflows/workflow-1.yml@172239021f7ba04fe7327647b213799853a9eb89
  call-workflow-2:
    uses: octo-org/another-repo/.github/workflows/workflow-2.yml@v1
```

**jobs.<job_id>.with**

与jobs.<job_id>.steps[*].with 不同的是，您通过jobs.<job_id>.with 传递的输入不能用作被调用工作流中的环境变量。 相反，您可以使用输入上下文来引用输入。

```yml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    with:
      username: mona
```

**jobs.<job_id>.with.<input_id>**

由输入的字符串标识符和输入的值组成的对。 标识符必须与被调用工作流中由 on.workflow_call.inputs.<inputs_id> 定义的输入名称匹配。 该值的数据类型必须与被调用工作流中的 on.workflow_call.<input_id>.type 定义的类型相匹配。

允许的表达式上下文：github 和需求。

**jobs.<job_id>.secrets**

当作业用于调用可重用工作流时，您可以使用机密来提供传递到被调用工作流的机密映射。

您传递的任何机密都必须与被调用工作流中定义的名称相匹配。

```yml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    secrets:
      access-token: ${{ secrets.PERSONAL_ACCESS_TOKEN }} 
```

**jobs.<job_id>.secrets.<secret_id>**

由秘密的字符串标识符和秘密的值组成的对。 标识符必须与被调用工作流中由 on.workflow_call.secrets.<secret_id> 定义的秘密名称相匹配。

允许的表达上下文：github、需求和秘密。

**过滤器模式备忘清单**

您可以在路径、分支和标记过滤器中使用特殊字符。

- `*`： 匹配零个或多个字符，但不匹配 `/` 字符。 例如，`Octo*` 匹配 `Octocat`。
- `**`： 匹配零个或多个任何字符。
- `?`：匹配零个或一个前缀字符。
- `+`: 匹配一个或多个前置字符。
- `[]` 匹配列在括号中或包含在范围内的一个字符。 范围只能包含 `a-z`、`A-Z` 和 `0-9`。 例如，范围 `[0-9a-z]` 匹配任何数字或小写字母。 例如，`[CB]at` 匹配 `Cat` 或 `Bat`，`[1-2]00` 匹配 `100` 和 `200`。
- `!`：在模式开始时，它将否定以前的正模式。 如果不是第一个字符，它就没有特殊的意义。

字符 `*`、`[` 和 `!` 是 YAML 中的特殊字符。 如果模式以 `*`、`[` 或 `!` 开头，必须用引号括住模式。

```yml
# Valid
- '**/README.md'

# Invalid - creates a parse error that
# prevents your workflow from running.
- **/README.md
```

**匹配分支和标记的模式**

| 模式                              | 描述                                                         | 示例匹配                                                     |
| :-------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `feature/*`                       | `*` 通配符匹配任何字符，但不匹配斜杠 (`/`)。                 | `feature/my-branch`<br/>`feature/your-branch`                |
| `feature/**`                      | `**` 通配符匹配任何字符，包括分支和标记名称中的斜杠 (`/`)。  | `feature/beta-a/my-branch`<br/>`feature/your-branch`<br/>`feature/mona/the/octocat` |
| `main``releases/mona-the-octocat` | 匹配分支或标记名称的确切名称。                               | `main``releases/mona-the-octocat`                            |
| `'*'`                             | 匹配所有不包含斜杠 (`/`) 的分支和标记名称。 `*` 字符是 YAML 中的特殊字符。 当模式以 `*` 开头时，您必须使用引号。 | `main``releases`                                             |
| `'**'`                            | 匹配所有分支和标记名称。 这是不使用 `branches` or `tags` 过滤器时的默认行为。 | `all/the/branches``every/tag`                                |
| `'*feature'`                      | `*` 字符是 YAML 中的特殊字符。 当模式以 `*` 开头时，您必须使用引号。 | `mona-feature``feature``ver-10-feature`                      |
| `v2*`                             | 匹配以 `v2` 开头的分支和标记名称。                           | `v2``v2.0``v2.9`                                             |
| `v[12].[0-9]+.[0-9]+`             | 将所有语义版本控制分支和标记与主要版本 1 或 2 匹配           | `v1.10.1``v2.0.0`                                            |

**匹配文件路径的模式**

路径模式必须匹配整个路径，并从仓库根开始。

| 模式                        | 匹配描述                                                     | 示例匹配                                                     |
| :-------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `'*'`                       | `*` 通配符匹配任何字符，但不匹配斜杠 (`/`)。 `*` 字符是 YAML 中的特殊字符。 当模式以 `*` 开头时，您必须使用引号。 | `README.md``server.rb`                                       |
| `'*.jsx?'`                  | `?` 个字符匹配零个或一个前缀字符。                           | `page.js``page.jsx`                                          |
| `'**'`                      | The `**` 通配符匹配任何字符，包括斜杠 (`/`)。 这是不使用 `path` 过滤器时的默认行为。 | `all/the/files.md`                                           |
| `'*.js'`                    | `*` 通配符匹配任何字符，但不匹配斜杠 (`/`)。 匹配仓库根目录上的所有 `.js` 文件。 | `app.js``index.js`                                           |
| `'**.js'`                   | 匹配仓库中的所有 `.js` 文件。                                | `index.js``js/index.js``src/js/app.js`                       |
| `docs/*`                    | 仓库根目录下 `docs` 根目录中的所有文件。                     | `docs/README.md``docs/file.txt`                              |
| `docs/**`                   | 仓库根目录下 `/docs` 目录中的任何文件。                      | `docs/README.md``docs/mona/octocat.txt`                      |
| `docs/**/*.md`              | `docs` 目录中任意位置具有 `.md` 后缀的文件。                 | `docs/README.md``docs/mona/hello-world.md``docs/a/markdown/file.md` |
| `'**/docs/**'`              | 仓库中任意位置 `docs` 目录下的任何文件。                     | `docs/hello.md``dir/docs/my-file.txt``space/docs/plan/space.doc` |
| `'**/README.md'`            | 仓库中任意位置的 README.md 文件。                            | `README.md``js/README.md`                                    |
| `'**/*src/**'`              | 仓库中任意位置具有 `src` 后缀的文件夹中的任何文件。          | `a/src/app.js``my-src/code/js/app.js`                        |
| `'**/*-post.md'`            | 仓库中任意位置具有后缀 `-post.md` 的文件。                   | `my-post.md``path/their-post.md`                             |
| `'**/migrate-*.sql'`        | 仓库中任意位置具有前缀 `migrate-` 和后缀 `.sql` 的文件。     | `migrate-10909.sql``db/migrate-v1.0.sql``db/sept/migrate-v1.sql` |
| `*.md``!README.md`          | 模式前使用感叹号 (`!`) 对其进行否定。 当文件与模式匹配并且也匹配文件后面定义的否定模式时，则不包括该文件。 | `hello.md`*Does not match*`README.md``docs/hello.md`         |
| `*.md``!README.md``README*` | 按顺序检查模式。 否定前一个模式的模式将重新包含文件路径。    | `hello.md``README.md``README.doc`                            |























<hr>

```yml
name: 构建
on: push
jobs:
  runs-on:  ubuntu-latest
  
  steps: 
```





# 变量

GitHub 设置适用于工作流程运行中每个步骤的默认环境变量。 环境变量区分大小写。 在操作或步骤中运行的命令可以创建、读取和修改环境变量。

要设置自定义环境变量，您需要在工作流程文件中指定变量。您可以使用jobs.<job_id>.step[*].env 、jobs.<job_id>.env和env关键字定义步骤、作业或整个工作流程的环境变量。

```yml
jobs:
  weekday_job:
    runs-on: ubuntu-latest
    env:
      DAY_OF_WEEK: Mon
    steps:
      - name: "Hello world when it's Monday"
        if: ${{ env.DAY_OF_WEEK == 'Mon' }}
        run: echo "Hello $FIRST_NAME $middle_name $Last_Name, today is Monday!"
        env:
          FIRST_NAME: Mona
          middle_name: The
          Last_Name: Octocat
```

要在工作流程文件中使用环境变量的值，您应该使用 env 上下文。 如果要在运行器中使用环境变量的值，您可以使用运行器操作系统的正常方法来读取环境变量。

https://docs.github.com/cn/actions/learn-github-actions/contexts#env-context

## 默认环境变量

强烈建议操作使用环境变量访问文件系统，而非使用硬编码的文件路径。 GitHub 设置供操作用于所有运行器环境中的环境变量。

| 环境变量             | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `CI`                 | 始终设置为 `true`。                                          |
| `GITHUB_WORKFLOW`    | 工作流程的名称。                                             |
| `GITHUB_RUN_ID`      | 仓库中每个运行的唯一编号。 如果您重新执行工作流程运行，此编号不变。 |
| `GITHUB_RUN_NUMBER`  | 仓库中特定工作流程每个运行的唯一编号。 此编号从 1（对应于工作流程的第一个运行）开始，然后随着每个新的运行而递增。 如果您重新执行工作流程运行，此编号不变。 |
| `GITHUB_JOB`         | 当前作业的 [job_id](https://docs.github.com/cn/actions/reference/workflow-syntax-for-github-actions#jobsjob_id)。 |
| `GITHUB_ACTION`      | 操作唯一的标识符 (`id`)。                                    |
| `GITHUB_ACTION_PATH` | 您的操作所在的路径。 您可以使用此路径访问与操作位于同一仓库中的文件。 此变量仅在复合操作中才受支持。 |
| `GITHUB_ACTIONS`     | 当 GitHub Actions 运行工作流程时，始终设置为 `true`。 您可以使用此变量来区分测试是在本地运行还是通过 GitHub Actions 运行。 |
| `GITHUB_ACTOR`       | 发起工作流程的个人或应用程序的名称。 例如 `octocat`。        |
| `GITHUB_REPOSITORY`  | 所有者和仓库名称。 例如 `octocat/Hello-World`。              |
| `GITHUB_EVENT_NAME`  | 触发工作流程的 web 挂钩事件的名称。                          |
| `GITHUB_EVENT_PATH`  | 具有完整 web 挂钩事件有效负载的文件路径。 例如 `/github/workflow/event.json`。 |
| `GITHUB_WORKSPACE`   | GitHub 工作空间目录路径，初始为空白。 例如 `/home/runner/work/my-repo-name/my-repo-name`。 [actions/checkout](https://github.com/actions/checkout) 操作将在此目录内检出文件，默认情况下是仓库的副本。 |
| `GITHUB_SHA`         | 触发工作流程的提交 SHA。 例如 `ffac537e6cbbf934b08745a378932722df287a53`。 |
| `GITHUB_REF`         | 触发工作流程的分支或标记参考。 例如 `refs/heads/feature-branch-1`。 如果分支或标记都不适用于事件类型，则变量不会存在。 |
| `GITHUB_REF_NAME`    | The branch or tag name that triggered the workflow run.      |
| `GITHUB_HEAD_REF`    | Only set for pull request events. 头部分支的名称。           |
| `RUNNER_TOOL_CACHE`  | 包含 GitHub 托管运行器预安装工具的目录路径。                 |

https://docs.github.com/cn/actions/using-github-hosted-runners/about-github-hosted-runners#supported-software

GitHub 在虚拟机上的特定目录中执行操作和 shell 命令。 虚拟机上的文件路径不是静态的。 使用环境变量 GitHub 提供 `home`、`workspace` 和 `workflow` 目录的构建文件路径。

| 目录                  | 环境变量            | 描述                                                         |
| :-------------------- | :------------------ | :----------------------------------------------------------- |
| `home`                | `HOME`              | 包含用户相关的数据。 例如，此目录可能包含登录凭据。          |
| `workspace`           | `GITHUB_WORKSPACE`  | 在此目录中执行操作和 shell 命令。 操作可以修改此目录的内容，后续操作可以访问这些修改。 |
| `workflow/event.json` | `GITHUB_EVENT_PATH` | 触发工作流程的 web 挂钩事件的 `POST` 有效负载。 每当操作执行时，GitHub 都会重写此变量，以隔离操作之间的文件内容。 |





















