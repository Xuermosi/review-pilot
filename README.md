# review-pilot

> An auditable code-review agent for Git changes and pull requests.

[中文文档](README.zh-CN.md) · [Benchmark report](docs/benchmarks/aacr-2026-10-23.md) · [Quick start](docs/quickstart.md)

`review-pilot` turns a code change into a reviewable, traceable result. It accepts
local staged changes, GitHub pull requests, GitLab merge requests, and AACR-Bench
cases, then routes them through one review engine. Rules, external tools, LLM
output, and final reports therefore share one evidence model.

## Why this project

An AI code-review workflow is useful only when a developer can answer three
questions: **what changed, why was it flagged, and what evidence supports the
finding?** `review-pilot` is designed around those questions.

- **One review path.** Adapters normalize a local diff, PR/MR, or benchmark case
  into `ReviewInput`, which is processed by `ReviewEngine`.
- **Evidence before prose.** Deterministic rules and optional Semgrep findings
  are scoped to changed lines. Context Pack records selected context, omissions,
  and the available token budget.
- **Bounded LLM assistance.** The LLM produces structured findings; optional
  ReviewUnit grouping, dynamic read-only tools, snippet location, and reflection
  support different strategies without changing the report contract.
- **A verification step.** Evidence Guard checks LLM findings against the actual
  diff and selected context before they are reported.
- **Workflow-oriented output.** The same result can become Markdown, JSON, a CI
  artifact, an idempotent PR/MR summary comment, a local hook outcome, or a
  Feishu notification.

## What it supports

### Inputs and entry points

- Local staged changes and comparisons against a base ref.
- GitHub pull requests, including GitHub Actions `pull_request` events.
- GitLab merge requests and GitLab CI execution.
- AACR-Bench cases for reproducible strategy evaluation.

### Review and evidence pipeline

- Built-in path, content, test, and size rules.
- Optional Semgrep integration, with findings filtered to changed lines by
  default.
- Repository context selection and a token-budgeted, auditable Context Pack.
- OpenAI-compatible LLM provider support plus a deterministic fake provider for
  local development and tests.
- Structured finding normalization, merging, severity thresholds, and report
  rendering.
- Evidence Guard, optional snippet location, and an optional reflection quality
  filter.

### Automation and delivery

- Markdown and JSON reports for local use or CI artifacts.
- GitHub/GitLab summary comments that update an existing tool-owned comment
  instead of posting duplicates.
- Managed `pre-commit` and `pre-push` hooks.
- Optional Feishu webhook notification based on a generated report.
- `doctor`, configuration inspection, prompt preview, context-pack generation,
  tool inspection, and output validation commands for debugging a workflow.

## Architecture and flow

Platform-specific logic lives at the edges, while review policy stays in the
core engine. A local run, CI run, and benchmark run can therefore follow the
same comparable execution path.

1. **Prepare input** — an adapter reads a staged diff, PR/MR workspace, or
   benchmark case and creates `ReviewInput`.
2. **Build evidence** — diff parsing, deterministic rules, and context
   selection produce review evidence and a bounded Context Pack.
3. **Review** — an optional provider receives a structured prompt. Depending on
   configuration, the engine can split work into ReviewUnits or expose bounded,
   read-only context tools.
4. **Validate and merge** — findings are normalized, checked by Evidence Guard,
   optionally reflected on, and merged with deterministic findings.
5. **Deliver results** — a reporter writes Markdown/JSON and the relevant
   adapter can publish CI artifacts, a PR/MR summary, or a Feishu card.

## Quick start

Requirements: Python 3.11+ and a Git repository when reviewing local changes.

```bash
python -m pip install -e '.[dev]'
review-pilot doctor
```

### Run deterministic checks locally

Stage one or more changes in a Git repository, then run:

```bash
review-pilot review --staged --no-ai --format markdown --output review-report.md
```

### Run with an OpenAI-compatible provider

Configure the provider in environment variables. Keep the key outside the
repository, screenshots, and logs.

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

Useful variants:

```bash
# Inspect configuration and LLM setup without exposing secrets.
review-pilot config validate
review-pilot llm doctor

# Prepare an auditable context pack before calling a provider.
review-pilot context-pack --staged --output context-pack.json

# Make CI fail when a finding reaches the chosen severity.
review-pilot review --staged --no-ai --fail-on P1
```

For an end-to-end local example, see [examples/final-demo](examples/final-demo).
For GitHub Actions artifacts, PR summary comments, and Feishu notifications,
see the [quick start guide](docs/quickstart.md).

## Configuration and safety boundaries

Project configuration is loaded from `.review-pilot.toml`; use
`review-pilot config show` or `review-pilot config validate` to inspect it.

- API keys and webhooks belong in environment variables or CI secrets, never in
  the repository.
- External tool findings remain limited to changed lines unless
  `--include-out-of-diff` is explicitly requested.
- Dynamic context tools are read-only and bounded by the review configuration.
- `--fail-on` changes the command exit code when the highest resulting severity
  meets the selected threshold.
- An LLM can assist the review, but it does not replace deterministic checks or
  evidence validation.

## Evaluation snapshot

On 2026-10-23–24, the project evaluated five review strategies on a fixed
30-case AACR-Bench sample using a DeepSeek V4.1 Flash 1017 deployment. Every
final group completed 30/30 cases.

- Baseline: Precision 0.4688, Recall 0.2913, F1 0.3593, Noise Rate 0.5313.
- ReviewUnit: Recall 0.3884, the highest of the five strategies; F1 0.3463 and
  a higher Noise Rate 0.6875.
- Dynamic Context: F1 0.2721.
- Snippet Location: F1 0.2472.
- Reflection: F1 0.2857; it used 774,106 reflection tokens in this run.

These figures are a fixed-sample engineering evaluation, not a public
leaderboard result or a general-performance claim across repositories, models,
or prompts. The [benchmark report](docs/benchmarks/aacr-2026-10-23.md) records
the fixed sample, conditions, complete metrics, provider continuity note, and
interpretation limits.

## Testing

Run the test suite after installation:

```bash
python -m pytest
```

The repository includes unit, integration-style, and end-to-end coverage for
diff parsing, context selection, rule execution, LLM schemas, provider output,
evidence checks, CLI behavior, hooks, platform adapters, reports, and benchmark
utilities.

## Reproducibility and raw artifacts

This repository commits a concise benchmark report and configuration summary.
Raw benchmark workspaces, cloned repositories, caches, logs, and per-case
runtime artifacts remain local for audit and are excluded by `.gitignore`.
They are intentionally not committed because they are large and environment
dependent.

## License

See the repository license information before reuse or redistribution.
