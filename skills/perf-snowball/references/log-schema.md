# Performance Log Schema

## Log Location

`~/.claude/snowball-performance-log/log.md`

## Entry Schema

Each fix gets one entry. Entries start as "In Progress" and transition to "Completed" when production results are captured.

### In Progress Entry

```markdown
## Fix #[number] -- [short description]
- **Status:** In Progress
- **Date started:** [YYYY-MM-DD]
- **PR:** [full PR URL]
- **Datadog links:** [APM Trace](url) | [additional relevant views](url)
- **Before (Datadog):** [metric: value, metric: value]
- **Before (local benchmark):** [description of local test and results]
- **How to measure impact:** [exact metric to watch] at [exact Datadog link]
- **Evidence:** `evidence/fix-[number]-before.json`, `evidence/fix-[number]-local-benchmark.md`
```

### Completed Entry

```markdown
## Fix #[number] -- [short description]
- **Status:** Completed
- **Date started:** [YYYY-MM-DD]
- **Date completed:** [YYYY-MM-DD]
- **PR:** [full PR URL]
- **Datadog links:** [APM Trace](url) | [additional relevant views](url)
- **Before (Datadog):** [metric: value, metric: value]
- **Before (local benchmark):** [description of local test and results]
- **After (Datadog):** [metric: value, metric: value]
- **Impact:** [percentage improvement, volume affected]
- **Business impact:** [translated to user-facing or cost terms]
- **Evidence:** `evidence/fix-[number]-before.json`, `evidence/fix-[number]-after.json`, `evidence/fix-[number]-local-benchmark.md`
```

## Fix Numbering

Fixes are numbered sequentially starting from 1. To determine the next number, count existing entries in `log.md`.

## Evidence Files

### Datadog Snapshots (`evidence/fix-[number]-before.json`, `evidence/fix-[number]-after.json`)

Raw JSON responses from Datadog API queries at the time of capture. These are the source of truth for metrics, surviving Datadog data archival (typically 15 days for logs/traces, 15 months for metrics).

Store the full API response including:
- Query parameters used
- Time range queried
- All returned data points
- Timestamp of capture

### Local Benchmark Results (`evidence/fix-[number]-local-benchmark.md`)

Structured markdown capturing local performance test results:

```markdown
# Local Benchmark: Fix #[number] -- [short description]

## Test Method
[Tool/technique used, e.g., "Benchmark.measure in rails console"]

## Before
[Exact commands run and their output]

## After
[Exact commands run and their output]

## Reproduction Steps
[Step-by-step instructions anyone can follow to reproduce this benchmark]
```

## Config File Location

Primary: `~/.claude/snowball-performance-log/config.md`
Additional: `~/.claude/snowball-performance-log/configs/[project-name].md`

Auto-detect which config to use by matching the current working directory's git remote URL to the repo URL stored in each config file. If ambiguous, ask the user.

## Summary Report Format

When generating a summary report from `log.md`, organize as follows:

### Technical Metrics
- Total latency reduction (sum of p95/p99 improvements across all fixes)
- Total database query time and count reduction
- Error rate reductions

### Business Metrics
- Requests affected per day (sum across all fixed endpoints)
- Cumulative user wait time saved daily
- User-facing page load improvements
- Infrastructure cost implications
- Reliability improvements (fewer timeouts, retries)

### Cumulative Narrative
- Total number of improvements: [count]
- Time range: [first fix date] to [last fix date]
- Top 3 biggest wins (ranked by business impact)
- Trend: [improving/stable/declining] based on severity of issues found over time
