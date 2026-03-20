# Lab 0

> Sidere mens eadem mutato. / 繁星纵变，智慧永恒。

## 欢迎来到实验零

现在，欢迎大家来到 OSH 2026 的实验零。这个实验的目的是教会大家使用 OS 课程的工具链，主要包含如下内容：

1. 使用 Linux 作为实验的主要环境
2. 使用 Git 维护自己的代码版本
3. 使用 Markdown 来编写美观、规范的文档
4. 使用 Makefile 来编写编译脚本
5. 使用 C/C++/Rust 编写程序的要求、规范和注意事项

每个小节的最后可能有若干**「必做」**练习需要完成，请注意提交。

???+ info "实验零的要求"

    本次实验（实验零）**不计分**，但是助教会检查大家的完成情况，针对完成的较差的同学给予提醒和帮助。**请确保在截止时间前完成所有的「必做」部分，避免影响接下来的实验**。

<!--<p style="color:red">注意：除了标明「必做」的部分外，实验零的完成情况不计入最终实验得分，但是助教会随机抽查，针对完成的较差的同学给予提醒和帮助。你可以通过在本页面搜索「必做」来查询所有的此类事项。</p>-->

我们推荐你在一开始就构建好自己的开发环境，并将开发环境的描述保存在一个 Markdown 文件中（例如下面的 `ENVIRONMENT.md`）。这对于 Agent 一类的 AI 应用而言非常有用。

**如果你需要使用 AI 工具，请务必阅读 AI 工具指引**。

实验零完成后，你应该创建了一个 `osh-2026-labs` 作业仓库，并且具有这样的结构：

```
osh-2026-labs
- README.md
- lab0
  - ENVIRONMENT.md
  - src
    - [pic] // src 文件夹存放图片，图片名字随意
  - markdown.md
  - linux.md
  - main.cpp
  - bubblesort.cpp
  - bubblesort.hpp
  - Makefile
  - CMakeLists.txt //如完成CMakeLists.txt选作
  - build //如完成CMakeLists.txt选作
    -  ...
- .gitignore 
```
