# perf-snowball

A Claude Code plugin for weekly Datadog-powered performance analysis. Identifies low-hanging performance improvements, guides implementation with provable local benchmarks, and maintains a running log of all improvements with measurable business impact.

Small gains that snowball into major impact.

## What It Does

- **Weekly sweep:** Pulls the last 7 days of Datadog data (APM traces, slow queries, error rates, frontend metrics) and surfaces 3-5 low-hanging fruit ranked by effort vs. impact
- **Historical validation:** Confirms each issue is a sustained problem (2-4 weeks), not a transient blip
- **Guided fixes:** Investigates issues, suggests specific code changes, and provides local benchmarking instructions to prove the fix works before merging
- **Evidence trail:** Captures Datadog metrics and local benchmark results as durable evidence that survives Datadog data archival
- **Impact reporting:** Generates summary reports translating technical metrics into business language (user wait time saved, requests affected, cost implications) -- ready for performance reviews

## Installation

### 1. Install the plugin

**From GitHub (recommended):**

First, add the marketplace, then install the plugin:

```bash
/plugin marketplace add kaylakantola/perf-snowball
/plugin install perf-snowball@perf-snowball
```

**From a local directory (for development):**

```bash
claude --plugin-dir /path/to/perf-snowball
```

Or add it as a local marketplace for permanent installation:

```bash
/plugin marketplace add /path/to/perf-snowball
/plugin install perf-snowball@perf-snowball
```

After installing, run `/reload-plugins` to activate.

### 2. Set up Datadog credentials

Add these to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export DATADOG_API_KEY="your-api-key"
export DATADOG_APP_KEY="your-app-key"
export DATADOG_SITE="datadoghq.com"  # or your region (datadoghq.eu, us3.datadoghq.com, etc.)
```

You can find your API and App keys in Datadog under **Organization Settings > API Keys / Application Keys**.

### 3. Configure for your project

From your project directory, run:

```
/snowball-setup
```

This will:
- Verify Datadog connectivity
- Detect your tech stack
- Discover available Datadog services and environments
- Write a config file to `~/.claude/snowball-performance-log/config.md`

## Usage

### Weekly Sweep

```
snowball sweep
```

Analyzes the last 7 days of Datadog data and presents 3-5 performance improvement candidates. Each candidate includes Datadog deep links, metrics, effort estimate, and a suggested fix.

**Focus on a specific area:**

```
snowball sweep, focus on frontend
snowball sweep, focus on database queries
snowball sweep, focus on the checkout flow
```

### Check Pending Fixes

```
snowball check
```

Shows fixes you've implemented but haven't logged production results for yet. Includes Datadog links to check current metrics.

### Work on a Fix

After a sweep, select an issue to work on:

```
let's work on #2
```

The plugin will investigate the issue, propose a fix, implement it, run local benchmarks to prove the improvement, and create a PR with Datadog evidence and local benchmark results in the description.

### Log Production Results

After a fix has been live for a few days:

```
snowball log #2
```

Pulls fresh Datadog data, calculates the improvement, and updates the performance log with before/after metrics and business impact.

### Generate Impact Report

```
snowball summary
```

Generates a formatted report of all improvements with:
- Technical metrics (latency reduction, query count reduction, error rate changes)
- Business metrics (requests affected, cumulative user wait time saved, cost implications)
- Cumulative narrative (total improvements, top 3 wins, trend over time)

### Capture Feedback

After a session, run a quick retro to improve the plugin over time:

```
snowball retro
```

## How Data Is Stored

All data lives in `~/.claude/snowball-performance-log/`:

| File | Purpose |
|------|---------|
| `config.md` | Project configuration (repo, stack, Datadog services) |
| `configs/` | Additional project configs (multi-project support) |
| `log.md` | Running tally of all performance improvements |
| `evidence/` | Raw Datadog JSON snapshots and local benchmark results |
| `retro.md` | Session feedback for plugin improvement |

Datadog credentials are **never stored in the plugin** -- they live in your environment variables.

## Portability

This plugin is designed to travel with you:

- **New machine:** Install the plugin, set env vars, run `/snowball-setup`
- **New job:** Update env vars to point at the new Datadog org, run `/snowball-setup` again. Your performance log history is preserved.
- **Different observability tool:** Swap the `.mcp.json` config to point at a different MCP server (e.g., New Relic, Grafana). The workflow skill stays the same.

## Recommended Weekly Routine

1. `snowball check` -- close out any pending fixes from last week
2. `snowball sweep` -- find new things to work on
3. Pick 1-2 issues, implement fixes
4. `snowball retro` -- capture feedback on the session
5. Come back in a few days to `snowball log` the results

## Tech Stack

Initially built for:
- Rails backend
- React frontend
- MySQL database
- Datadog for observability

The reference files in `skills/perf-snowball/references/` contain stack-specific query patterns and benchmarking methods. These can be updated for different stacks.
