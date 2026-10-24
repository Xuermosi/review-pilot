# review-pilot

> 面向 Git 变更与 Pull Request 的可审计代码审查 Agent。

[English](README.md) · [评测报告](docs/benchmarks/aacr-2026-10-23.md) · [快速开始](docs/quickstart.md)

`review-pilot` 将代码变更转为可审阅、可追溯的结果。它支持本地 staged diff、
GitHub Pull Request、GitLab Merge Request 和 AACR-Bench case，并将它们统一
交由同一审查引擎处理。因此，规则、外部工具、LLM 输出与最终报告共用一套
证据模型，而不是由几段行为各异的脚本拼接而成。

## 为什么做这个项目

一个 AI 代码审查工作流是否可靠，最终要回答三个问题：**改了什么、为何被
标记、这条 Finding 的依据是什么？** `review-pilot` 围绕这三个问题设计。

- **一条审查主链路。** 入口适配器把本地 diff、PR/MR 或 benchmark case
  统一为 `ReviewInput`，再由 `ReviewEngine` 执行。
- **先建立证据，再生成结论。** 确定性规则与可选 Semgrep Finding 默认收敛
  到变更行；Context Pack 会记录选择了哪些仓库上下文、哪些被省略以及
  Token 预算。
- **在边界内使用 LLM。** LLM 以结构化 Finding 的方式参与；可选的
  ReviewUnit、动态只读工具、Snippet Location 和 Reflection 提供不同策略，
  但不会改变报告契约。
- **有独立校验步骤。** Evidence Guard 会在输出前检查 LLM Finding 是否能由
  实际 diff 与选定上下文支撑。
- **面向真实工作流交付。** 同一结果可输出为 Markdown、JSON、CI Artifact、
  幂等 PR/MR 摘要评论、本地 Hook 结果或飞书通知。

## 支持的能力

### 输入与入口

- 本地 staged 变更，以及与指定 base ref 的差异审查。
- GitHub Pull Request，包括 GitHub Actions 的 `pull_request` 事件。
- GitLab Merge Request 与 GitLab CI 运行。
- 用于可复现策略评测的 AACR-Bench case。

### 审查与证据链

- 内置路径、内容、测试和变更规模规则。
- 可选 Semgrep 集成，默认把工具结果过滤到本次变更行。
- 仓库上下文选择，以及带 Token 预算、可审计的 Context Pack。
- OpenAI-compatible Provider，以及用于本地开发和测试的确定性 fake
  provider。
- 结构化 Finding 规范化、合并、严重性阈值与报告渲染。
- Evidence Guard、可选 Snippet Location 与可选 Reflection 质量过滤。

### 自动化与交付

- 面向本地和 CI Artifact 的 Markdown、JSON 报告。
- GitHub/GitLab 摘要评论：重复运行时会更新已有的工具评论，而非重复刷屏。
- 托管 `pre-commit` 与 `pre-push` Hook。
- 基于最终报告的可选飞书 Webhook 通知。
- `doctor`、配置检查、Prompt 预览、Context Pack 生成、工具注册表与输出
  校验等命令，帮助定位工作流问题。

## 架构与主链路

项目把平台相关逻辑放在边缘，将审查策略集中在核心引擎。因此，本地运行、
CI 运行和 benchmark 运行能够遵循同一条可对照的执行路径。

1. **准备输入**：入口适配器读取 staged diff、PR/MR workspace 或
   benchmark case，创建 `ReviewInput`。
2. **构建证据**：diff 解析、确定性规则和上下文选择共同生成审查证据与带
   上限的 Context Pack。
3. **执行审查**：可选 Provider 接收结构化 Prompt；根据配置，引擎可将工作
   切分为 ReviewUnit，或开放受限的只读上下文工具。
4. **校验与合并**：Finding 会被规范化、经 Evidence Guard 校验、可选地经过
   Reflection，再与确定性 Finding 合并。
5. **交付结果**：Reporter 写出 Markdown / JSON；对应的适配器还可发布 CI
   Artifact、PR/MR 摘要或飞书卡片。

## 快速开始

环境要求：Python 3.11+；审查本地变更时需要位于 Git 仓库中。

```bash
python -m pip install -e '.[dev]'
review-pilot doctor
```

### 只运行确定性检查

先在 Git 仓库中 stage 一处或多处变更，再执行：

```bash
review-pilot review --staged --no-ai --format markdown --output review-report.md
```

### 使用 OpenAI-compatible Provider

通过环境变量配置 Provider。密钥不应写入仓库、截图或日志。

```bash
export REVIEW_PILOT_LLM_PROVIDER=openai-compatible
export REVIEW_PILOT_LLM_MODEL=deepseek-v4-pro
export REVIEW_PILOT_LLM_BASE_URL=https://api.deepseek.com
export REVIEW_PILOT_API_KEY='<your-api-key>'
export REVIEW_PILOT_LLM_TIMEOUT_SECONDS=120

review-pilot review --staged \
  --provider openai-compatible \
  --format markdown \
  --output review-report.md
```

常用辅助命令：

```bash
# 查看配置与 LLM 状态；不会输出密钥。
review-pilot config validate
review-pilot llm doctor

# 先生成可审计的上下文包，再调用 Provider。
review-pilot context-pack --staged --output context-pack.json

# 当最终结果达到设定严重性时，让 CI 返回失败。
review-pilot review --staged --no-ai --fail-on P1
```

完整本地演示见 [examples/final-demo](examples/final-demo)。GitHub Actions
Artifact、PR 摘要评论和飞书通知的配置见[快速开始](docs/quickstart.md)。

## 配置与安全边界

项目配置从 `.review-pilot.toml` 读取，可通过 `review-pilot config show`
或 `review-pilot config validate` 查看。

- API Key 与 Webhook 应使用环境变量或 CI Secret，不进入版本库。
- 外部工具 Finding 默认限制在变更行；只有显式传入
  `--include-out-of-diff` 才保留 diff 外结果。
- 动态上下文工具是只读的，并受审查配置的调用边界限制。
- `--fail-on` 是工作流控制项：当最高严重性达到设定等级时，命令会返回
  exit code 1。
- LLM 用于辅助审查；它不能替代确定性检查和证据校验。

## 评测快照

2026-10-23 至 24 日，项目在固定的 AACR-Bench 30-case 样本上，对五种审查
策略进行了评测，使用 DeepSeek V4.1 Flash 1017 部署。每个终版组均完成
30/30 个 case。

- Baseline：Precision 0.4688、Recall 0.2913、F1 0.3593、Noise Rate 0.5313。
- ReviewUnit：Recall 0.3884，为五种策略中最高；F1 0.3463，Noise Rate
  也升至 0.6875。
- Dynamic Context：F1 0.2721。
- Snippet Location：F1 0.2472。
- Reflection：F1 0.2857；本次运行使用 774,106 个 reflection token。

这些数字只描述固定样本上的工程评测，不是公开排行榜结果，也不构成对不同
仓库、模型或 Prompt 的通用性能承诺。完整的固定样本、实验条件、全部指标、
Provider 连续性说明与解读边界，见[评测报告](docs/benchmarks/aacr-2026-10-23.md)。

## 测试

安装完成后运行：

```bash
python -m pytest
```

仓库包含单元测试、集成风格测试和端到端测试，覆盖 diff 解析、上下文选择、
规则执行、LLM schema、Provider 输出、证据校验、CLI、Hook、平台适配器、
报告和 benchmark 工具。

## 可复核性与原始产物

仓库仅提交精简、可读的实验报告与配置摘要。原始 benchmark workspace、
克隆仓库、缓存、日志和逐例运行产物保留在本地审计，并由 `.gitignore`
排除。它们体积较大且依赖运行环境，因此不提交到版本库。

## 许可证

复用或分发前请查看仓库中的许可证信息。
