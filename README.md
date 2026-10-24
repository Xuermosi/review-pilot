# review-pilot

[中文文档](README.zh-CN.md) · [Benchmark summary](docs/benchmarks/aacr-2026-10-23.md)

`review-pilot` is an auditable code-review agent for local staged changes, GitHub pull requests, GitLab merge requests, and AACR-Bench cases. It keeps platform adapters thin and routes every input through one review engine.

## Highlights

- One `ReviewInput` / `ReviewEngine` execution path for local, GitHub, GitLab, and benchmark inputs.
- Deterministic rules and optional Semgrep findings, filtered to changed lines.
- Auditable Context Packs with a token budget and omitted-context record.
- Evidence-checked LLM findings; optional ReviewUnit partitioning and bounded read-only context tools.
- Markdown/JSON reports, managed Git hooks, CI artifacts, idempotent PR/MR summary comments, and Feishu notifications.

## Benchmark snapshot

On 2026-10-23–24, the project completed five AACR-Bench strategy runs on a fixed 30-case sample using a DeepSeek V4.1 Flash 1017 deployment. Every final group completed 30/30 cases. In this sample, the baseline achieved the highest F1 (0.3593); ReviewUnit had the highest recall (0.3884) with a higher noise rate.

This is a fixed-sample engineering evaluation, not a public leaderboard result or a claim of general performance. See the [reproducibility conditions, ablation metrics, and caveats](docs/benchmarks/aacr-2026-10-23.md).

## Quick start

```bash
python -m pip install -e '.[dev]'
review-pilot --help
review-pilot doctor
review-pilot review --staged --provider fake
python -m pytest
```

For a local demo, see [examples/final-demo](examples/final-demo). For the command-line surface, run `review-pilot --help`.

## Core flow

```text
local staged diff / GitHub PR / GitLab MR / AACR case
        -> adapter -> ReviewInput -> ReviewEngine
        -> rules / context / tools / LLM / evidence guard / merge
        -> Markdown / JSON / CI artifact / platform summary
```

## Reproducibility and raw artifacts

The repository intentionally stores the concise benchmark report rather than raw benchmark workspaces, caches, logs, and case files. Raw artifacts are retained locally for audit but are excluded by `.gitignore` because they include cloned repositories and runtime output.

## License

See the repository license information before reuse or redistribution.
