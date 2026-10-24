# review-pilot

> Auditable code review for Git changes and pull requests.

[中文](README.zh-CN.md) · [Benchmark](docs/benchmarks/aacr-2026-10-23.md)

## At a glance

- One engine for local, GitHub, GitLab, and benchmark inputs.
- Rules and optional Semgrep produce deterministic evidence.
- Context Packs and read-only tools bound LLM context.
- Evidence Guard checks findings before reports are created.
- Markdown/JSON reports, hooks, CI artifacts, and Feishu summaries.

## Evaluation

Fixed 30-case AACR-Bench evaluation, five strategies, all final groups 30/30.

- Baseline: F1 0.3593 (best in this sample)
- ReviewUnit: Recall 0.3884 (higher noise)

This is an engineering evaluation on a fixed sample, not a public leaderboard.
[Conditions, metrics, and caveats →](docs/benchmarks/aacr-2026-10-23.md)

## Quick start

```bash
python -m pip install -e '.[dev]'
review-pilot --help
review-pilot doctor
review-pilot review --staged --provider fake
python -m pytest
```

Try [examples/final-demo](examples/final-demo) for a local demo.

## How it flows

1. An adapter turns a Git change, PR/MR, or AACR case into `ReviewInput`.
2. `ReviewEngine` runs rules, context gathering, LLM review, and evidence checks.
3. Reports are emitted as Markdown, JSON, CI artifacts, or platform summaries.

## Reproducibility and raw artifacts

The repository includes a concise benchmark report. Raw workspaces, caches, logs,
and case files stay local for audit and are excluded by `.gitignore`.

## License

See the repository license information before reuse or redistribution.
