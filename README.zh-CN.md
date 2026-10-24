# review-pilot

> 面向 Git 变更与 Pull Request 的可审计代码审查。

[English](README.md) · [评测摘要](docs/benchmarks/aacr-2026-10-23.md)

## 一览

`review-pilot` 将代码变更转为可审阅、可追溯的结果。本地 staged diff、
GitHub Pull Request、GitLab Merge Request 和 AACR-Bench case 均复用同一
套审查引擎，而不是维护多条彼此割裂的审查链路。

## 能力概览

- **统一输入。** 入口适配器把 Git 变更、PR、MR 和评测 case 归一为
  `ReviewInput`，交给 `ReviewEngine` 执行。
- **先构建证据。** 规则与可选 Semgrep 结果会收敛到变更行；Context Pack
  同时记录 Token 预算与未纳入的上下文。
- **在边界内使用 LLM。** ReviewUnit 分包和只读上下文工具帮助聚焦分析，
  不允许审查器改动目标仓库。
- **校验每一条 Finding。** Evidence Guard 会在生成报告前检查 LLM Finding
  是否有足够证据支撑。
- **适配日常工作流。** 支持 Markdown / JSON、CI Artifact、平台摘要、
  托管 Git Hook 与飞书通知。

## 审查主链路

1. 入口适配器将 Git 变更、PR/MR 或 AACR case 准备为 `ReviewInput`。
2. `ReviewEngine` 执行规则、上下文收集、可选工具、LLM 审查、证据校验和
   Finding 合并。
3. Reporter 输出 Markdown / JSON、CI Artifact 或面向平台的摘要。

这样入口适配器保持轻量，审查策略则集中在一条可追溯的执行链路中。

## 评测

项目基于固定的 AACR-Bench 30-case 样本，对五种策略进行了评测，使用
DeepSeek V4.1 Flash 1017 部署；各终版组均完成 30/30 个 case。

- baseline：F1 0.3593（该样本最高）
- ReviewUnit：Recall 0.3884（噪声更高）

这些数字只描述该固定样本上的工程评测，不是公开榜单，也不构成对不同
仓库或模型的通用性能承诺。

[查看条件、指标与边界 →](docs/benchmarks/aacr-2026-10-23.md)

## 快速开始

```bash
python -m pip install -e '.[dev]'
review-pilot --help
review-pilot doctor
review-pilot review --staged --provider fake
python -m pytest
```

本地演示见 [examples/final-demo](examples/final-demo)；完整命令可运行
`review-pilot --help` 查看。

## 可复核性与原始产物

仓库只提交精简、可读的实验报告和配置摘要。原始 workspace、缓存、
日志和逐例结果保留在本地审计，并由 `.gitignore` 排除，避免把克隆仓库
与环境相关运行产物提交进版本库。

## 许可证

复用或分发前请查看仓库中的许可证信息。
