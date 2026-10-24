# review-pilot

> 面向 Git 变更与 Pull Request 的可审计代码审查。

[English](README.md) · [评测摘要](docs/benchmarks/aacr-2026-10-23.md)

## 一览

- 本地、GitHub、GitLab 和评测输入共用一个审查引擎。
- 规则与可选 Semgrep 提供确定性证据。
- Context Pack 和只读工具控制 LLM 上下文范围。
- Evidence Guard 在生成报告前校验 Finding。
- 支持 Markdown / JSON、Hook、CI Artifact 和飞书摘要。

## 评测

固定 AACR-Bench 30-case 样本，五种策略，终版均为 30/30。

- baseline：F1 0.3593（该样本最高）
- ReviewUnit：Recall 0.3884（噪声更高）

这是固定样本工程评测，不是公开榜单或通用性能结论。
[查看条件、指标与边界 →](docs/benchmarks/aacr-2026-10-23.md)

## 快速开始

```bash
python -m pip install -e '.[dev]'
review-pilot --help
review-pilot doctor
review-pilot review --staged --provider fake
python -m pytest
```

本地演示见 [examples/final-demo](examples/final-demo)。

## 主链路怎么走

1. 入口适配器将 Git 变更、PR/MR 或 AACR case 组织成 `ReviewInput`。
2. `ReviewEngine` 执行规则、上下文、LLM 审查和证据校验。
3. 结果输出为 Markdown、JSON、CI Artifact 或平台摘要。

## 可复核性与原始产物

仓库只提交精简、可读的实验报告。原始 workspace、缓存、日志和逐例
结果保留在本地审计，并由 `.gitignore` 排除。

## 许可证

复用或分发前请查看仓库中的许可证信息。
