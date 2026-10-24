# review-pilot

[English](README.md) · [评测摘要](docs/benchmarks/aacr-2026-10-23.md)

`review-pilot` 是一个可审计的代码 Review Agent，支持本地 staged diff、GitHub Pull Request、GitLab Merge Request 与 AACR-Bench 样本。不同入口只负责准备输入，审查决策统一由同一份引擎执行。

## 核心能力

- 使用统一的 `ReviewInput` / `ReviewEngine` 执行本地、GitHub、GitLab 与评测输入。
- 运行确定性规则与可选 Semgrep，并将工具结果过滤至本次变更行。
- 构建带 Token 预算和 omitted-context 记录的可审计 Context Pack。
- 对 LLM Finding 做证据校验；可选 ReviewUnit 分包与受限只读上下文工具。
- 输出 Markdown / JSON 报告，并支持托管 Git Hook、CI Artifact、幂等 PR/MR 摘要评论和飞书通知。

## 评测快照

2026-10-23 至 24 日，项目在固定的 AACR-Bench 30-case 样本上，完成了五种审查策略的全量运行；每个终版组均为 30/30 completed。该实验使用 DeepSeek V4.1 Flash 1017 部署：baseline 的 F1 最高，为 0.3593；ReviewUnit 的 Recall 最高，为 0.3884，但噪声率更高。

这是一项固定样本工程评测，不是公开排行榜结果，也不代表对所有仓库或模型的通用性能承诺。详细的复现条件、消融指标和边界见[评测摘要](docs/benchmarks/aacr-2026-10-23.md)。

## 快速开始

```bash
python -m pip install -e '.[dev]'
review-pilot --help
review-pilot doctor
review-pilot review --staged --provider fake
python -m pytest
```

本地演示见 [examples/final-demo](examples/final-demo)；完整命令列表可运行 `review-pilot --help` 查看。

## 主链路

```text
本地 staged diff / GitHub PR / GitLab MR / AACR case
        -> 入口适配器 -> ReviewInput -> ReviewEngine
        -> 规则 / 上下文 / 工具 / LLM / Evidence Guard / 合并
        -> Markdown / JSON / CI Artifact / 平台摘要
```

## 可复核性与原始产物

仓库只提交精简、可读的实验报告；原始 benchmark workspace、仓库 cache、日志和逐例结果保留在本地审计，但因包含克隆仓库和运行产物，已由 `.gitignore` 排除，避免将大体积与环境相关文件提交到版本库。

## 许可证

复用或分发前请查看仓库中的许可证信息。
