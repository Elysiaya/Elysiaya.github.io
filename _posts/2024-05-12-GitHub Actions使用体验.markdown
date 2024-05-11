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

接下来与上面的标签平级的就是jobs，该标签可以说是最重要的一个标签，用来指定我的工作流具体干什么工作，jobs可以定义多个job，例如上述文件中定义了一个build的job，工作流被触发执行的时候就执行该job。
