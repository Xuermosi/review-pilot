# review-pilot

> Auditable code review for Git changes and pull requests.

[中文](README.zh-CN.md) · [Benchmark](docs/benchmarks/aacr-2026-10-23.md)

## At a glance

`review-pilot` turns a code change into a reviewable, traceable result. Local
staged diffs, GitHub pull requests, GitLab merge requests, and AACR-Bench cases
share the same review engine instead of maintaining separate review pipelines.

## What it does

- **Unifies inputs.** Adapters normalize Git changes, PRs, MRs, and benchmark
  cases into one `ReviewInput` for `ReviewEngine`.
- **Builds evidence first.** Rules and optional Semgrep findings are scoped to
  changed lines, while Context Packs retain a token budget and omitted-context
  record.
- **Uses LLMs within boundaries.** ReviewUnit partitioning and read-only
  context tools help focus analysis without allowing the reviewer to modify
  the target repository.
- **Checks every finding.** Evidence Guard validates that an LLM finding is
  supported before it reaches the final report.
- **Fits common workflows.** Results can be emitted as Markdown, JSON, CI
  artifacts, platform summaries, managed Git hooks, or Feishu notifications.

## Review flow

1. An adapter prepares a Git change, PR/MR, or AACR case as `ReviewInput`.
2. `ReviewEngine` runs rules, context collection, optional tools, LLM review,
   evidence checks, and finding merge.
3. The selected reporter produces Markdown/JSON output, CI artifacts, or a
   platform-facing summary.

This keeps input adapters thin and concentrates the review policy in one
auditable execution path.

## Evaluation

The project ran five strategies on a fixed 30-case AACR-Bench sample with a
DeepSeek V4.1 Flash 1017 deployment. Every final group completed all 30 cases.

- Baseline: F1 0.3593 (best in this sample)
- ReviewUnit: Recall 0.3884 (higher noise)

The numbers describe this fixed-sample engineering evaluation. They are not a
public leaderboard result or a general-performance claim across repositories
and models.

[Conditions, metrics, and caveats →](docs/benchmarks/aacr-2026-10-23.md)

## Quick start

```bash
python -m pip install -e '.[dev]'
review-pilot --help
review-pilot doctor
review-pilot review --staged --provider fake
python -m pytest
```

Try [examples/final-demo](examples/final-demo) for a local demo. Run
`review-pilot --help` to explore the command surface.

## Reproducibility and raw artifacts

The repository includes a concise benchmark report and configuration summary.
Raw workspaces, caches, logs, and case files stay local for audit and are
excluded by `.gitignore` because they contain cloned repositories and runtime
output.

## License

See the repository license information before reuse or redistribution.
