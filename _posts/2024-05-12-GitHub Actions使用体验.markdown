---
layout:       post
title:        "GitHub Actions使用体验"
subtitle:     "GitHub Actions使用体验"
author:       "ZhangXu"
header-style: text
catalog:      true
tags:
    - GitHub
    - Actions
    - 部署
---

> GitHub Actions使用体验，这是一种可以自动执行的定时脚本任务，可以根据用户的需要生成编译文件。在c或者c#中可以用来生成多平台的二进制可执行文件，在web中可以自动部署到相应的服务器中

## GitHub Actions
在 GitHub Actions 的仓库中自动化、自定义和执行软件开发工作流程。 您可以发现、创建和共享操作以执行您喜欢的任何作业（包括 CI/CD），并将操作合并到完全自定义的工作流程中。
总之就是可以根据自己的需求定制一套软件的部署流程，很好用。。。
## GitHub Actions文件
在代码仓库界面的Actions菜单中可以选择新建一个GitHub Actions文件，其形式为*.yaml文件，有固定的语法规则，创建完成的文件在仓库的.github/workflows文件夹中，GitHub检测到该文件存在就会自动执行相应的*.yaml文件。下面是一段GitHub Actions文件代码示例，作用可以将.net项目编译并保存。

```yaml
name: .NET Build & Test

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    strategy:
      matrix:
        configuration: [Debug, Release]
    runs-on: windows-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup
        uses: actions/setup-dotnet@v1
      - name: Build
        run: dotnet build --configuration $env:Configuration
        env:
          Configuration: ${{ matrix.configuration }}
      - name: Test
        run: dotnet test --configuration $env:Configuration
        env:
          Configuration: ${{ matrix.configuration }}
      - name: 查看当前工作目录与显示工作目录的文件
        run: pwd | ls | tree
      - name: Upload dist
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.configuration }}
          path: ./ConsoleApp_githubActions/bin/${{ matrix.configuration }}/net8.0/*.exe
```

可以看出，上面的文件存在一定的层级关系，以键值对的形式存在。第一行的name后跟:（冒号），并空一格，后面的值就是本次Actions任务的名称，一般可以不写，但是一般都要写一个。

与name同级的就是on，on用来定义哪些事件可以用来触发该工作流。可以定义单个或多个可以触发工作流的事件，或设置时间计划。最常用的就是`on: push `,当有推送到仓库的任何分支时就会触发改工作流。也可以使用多个事件，例如`on: [push, fork]`。如果指定多个事件，仅需发生其中一个事件就可触发工作流。 如果触发工作流的多个事件同时发生，将触发多个工作流运行。

与上面的标签平级的就是jobs，该标签可以说是最重要的一个标签，用来指定我的工作流具体干什么工作，jobs可以定义多个job，例如上述文件中定义了一个build的job，工作流被触发执行的时候就执行该job。每个job中包含多个steps。

上述示例文件中创建了一个名为build的job，该job定义了strategy策略、runs-on定义操作系统，steps定义该脚本的每一步内容。其中定义了一个matrix矩阵，在该矩阵中定义的元素会被依次执行，后面的内容可以用`${{ matrix.configuration }}`调用。例如本次定义了debug模式和release模式，这样在后面的编译过程中就不用连续定义两个job了。

steps中的每一中name确定操作的名称，该名称会在Actions执行过程中显示在信息框内，便于查看Actions执行的情况，如果出现错误，也可以便于分析和定位。

```yaml
- name: Checkout
  uses: actions/checkout@v2
```
该step用于将GitHub仓库的代码签出到本地，一般每个脚本都会执行该段代码。
```yaml
- name: Upload dist
  uses: actions/upload-artifact@v4
  with:
    name: ${{ matrix.configuration }}
    path: ./ConsoleApp_githubActions/bin/${{ matrix.configuration }}/net8.0/*.exe
```
Artifact是GitHub Action存在的一个特殊仓库，可以将编译完成的文件上传到这里，一般可以用于保存编译完成的二进制文件，操作日志等需要保存的信息，Artifact内保存的文件不是永久的，注意及时下载。

最后，任何技术都要用起来才能真正掌握。GitHub Actions也很难说是一项技术，看起来她更像是一种GitHub给开发者提供的一项福利，用于编译部署发布自己的内容。在这里感谢他们。
