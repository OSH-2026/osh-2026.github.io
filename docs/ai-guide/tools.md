# 二：工具

## 概述

本节介绍基于大语言模型的编程辅助工具，如 IDE 插件、CLI 工具等。

## 常用工具

本课程推荐把工具链分成四层：**IDE/Agent（开发与审阅）→ 模型路由（统一接入）→ Agent 框架（自动化工作流）→ 可观测性（证据与复现）**。

## 2.1 IDE 与 Agent

### 推荐策略：Cursor 为主，Trae 为备选（强隐私隔离）

- **Cursor（推荐主力）**
  - 适合作为课程统一 IDE：支持团队/课程共享规则、Composer/Agent、代码库索引、Review 工作流，便于统一教学与助教支持。
  - 文档：[https://cursor.com/docs](https://cursor.com/docs)
  - Rules：[https://cursor.com/docs/context/rules](https://cursor.com/docs/context/rules)

- **Trae（备选，需隔离使用）**
  - 社区里常被当成 Cursor 替代；但 2025 下半年出现过关于隐私/遥测的争议报道。
  - **课程支持建议**：只允许在隔离容器/无敏感仓库环境使用；避免接触含 token、私钥、学校/企业内部代码的目录。
  - 官网：[https://www.trae.ai/](https://www.trae.ai/)
  - 文档：[https://docs.trae.ai/](https://docs.trae.ai/)

## 2.2 模型接入：OpenRouter 作为统一“模型路由层”

在教学场景里，建议用 OpenRouter 作为统一接入层（同一套 API/Key、按任务切换模型、便于成本与能力管理）。OpenRouter 的模型页面通常会标注发布时间、上下文长度、价格与基准，适合做“分层选择”。

- OpenRouter 官网：[https://openrouter.ai/](https://openrouter.ai/)
- Quickstart：[https://openrouter.ai/docs/quickstart](https://openrouter.ai/docs/quickstart)
- API Reference：[https://openrouter.ai/docs/api/reference/overview](https://openrouter.ai/docs/api/reference/overview)
- Models 指南：[https://openrouter.ai/docs/guides/overview/models](https://openrouter.ai/docs/guides/overview/models)


## 2.3 Agent 框架：OpenClaw（或同类）与“安全默认值”

OpenClaw（或类似框架）适合做“OS 实验助手”：

- 自动生成实验报告骨架
- 跑测试/收集结果
- 整理 trace/log
- 生成 patch 与提交说明（由同学/助教审阅决策）

但这类框架也会放大安全风险，因此要设置安全默认值。

- OpenClaw Install：[https://docs.openclaw.ai/install](https://docs.openclaw.ai/install)
- GitHub Repo：[https://github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)

## 2.4 eBPF：可观测性与可验证的课程核心工具

在“让 agent 像人一样操作系统”的叙事中，eBPF 的关键价值是“hook 万物”，把行为变成**可观测、可审计、可回放**的证据：

- **观测**：调度、内存、IO、Binder/系统调用、GPU 驱动路径等
- **约束与证明**：为 agent 的每一步提供可审计证据（trace/log）
- **回放与复现**：trace → 最小复现用例 → 自动化验证

课程落地建议：**每个实验至少产出一份 trace**（perf / ftrace / eBPF 三选一即可）。

参考：

- Kernel BPF 文档入口：[https://docs.kernel.org/bpf/](https://docs.kernel.org/bpf/)
- BPF Verifier：[https://docs.kernel.org/bpf/verifier.html](https://docs.kernel.org/bpf/verifier.html)
- eBPF.io 实践知识库：[https://docs.ebpf.io/](https://docs.ebpf.io/)

## 2.5 其他常用参考入口（源码/平台）

- **GitHub**
  - 官网：[https://github.com/](https://github.com/)
  - Docs：[https://docs.github.com/](https://docs.github.com/)

- **Android（AOSP）**
  - 官网：[https://source.android.com/](https://source.android.com/)
  - 文档入口：[https://source.android.com/docs](https://source.android.com/docs)
  - Setup：[https://source.android.com/docs/setup](https://source.android.com/docs/setup)

- **Linux Kernel**
  - kernel.org：[https://www.kernel.org/](https://www.kernel.org/)
  - Linux Kernel Docs：[https://docs.kernel.org/](https://docs.kernel.org/)
  - kernel.org/doc：[https://www.kernel.org/doc/](https://www.kernel.org/doc/)
