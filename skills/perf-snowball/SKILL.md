---
name: perf-snowball
description: This skill should be used when the user says "snowball sweep", "snowball check", "snowball log", "snowball summary", "snowball report", or any phrase containing "snowball" in the context of performance analysis, optimization tracking, or impact reporting. Do NOT activate unless the word "snowball" appears in the user's message.
version: 0.1.0
---

# perf-snowball

A weekly performance analysis workflow that connects to Datadog, identifies low-hanging performance improvements, guides implementation with provable local benchmarks, and maintains a running log of improvements with measurable business impact. Small gains that snowball into major impact.

## Prerequisites

Run `/snowball-setup` before first use to configure Datadog connectivity and project context. Configuration is stored at `~/.claude/snowball-performance-log/config.md`.

## Workflow Commands

### snowball sweep

Run the weekly performance analysis. Accepts an optional focus area.

**Invocations:** "snowball sweep", "run my snowball sweep", "snowball sweep, focus on [area]"

**Process:**

1. Read config from `~/.claude/snowball-performance-log/config.md` (or detect the correct config from `configs/` by matching the current repo's git remote URL). If no config found, prompt to run `/snowball-setup`.

2. Query Datadog for the last 7 days of data using the configured services and environment. Consult `references/datadog-queries.md` for query patterns. Pull:
   - APM traces: slowest endpoints, p95/p99 latency
   - Database: slow queries, N+1 patterns
   - Error rates: sustained elevated rates
   - Frontend RUM data (if available)
   - Resource metrics (if available)

3. If a focus area is provided, scope queries accordingly:
   - "frontend" -> RUM, page load, Core Web Vitals
   - "database" / "queries" -> DBM, slow queries, N+1s
   - "API" / "latency" / "backend" -> APM endpoint traces
   - Any other text -> use as a filter on service/endpoint names

4. Identify 3-5 candidates ranked by effort vs. impact. For each candidate, validate it is a sustained issue by checking 2-4 weeks of historical data. Filter out anything that appears to be a transient blip (only present in the most recent week).

5. Present each candidate with:
   - Description of the issue
   - Duration (how long it has been happening, with trend)
   - Metrics (p95 latency, query count, error rate, etc.)
   - Request volume (daily requests affected)
   - Datadog deep links (construct URLs per `references/datadog-queries.md`)
   - "How to measure impact" section with exact metric and Datadog link
   - Effort estimate (small / medium / large)
   - Suggested fix (specific code change with file path and line reference)

### snowball check

Check on pending fixes that need results logged.

**Invocations:** "snowball check", "snowball, what do I need to check up on?"

**Process:**

1. Read `~/.claude/snowball-performance-log/log.md`
2. Find all entries with **Status: In Progress**
3. For each, show:
   - Fix name and number
   - Days since fix was started
   - PR link
   - Before metrics
   - Datadog link to check current state
4. Ask if the user wants to log results for any of them

### Selecting and Implementing a Fix

When the user selects an issue from the sweep (e.g., "let's work on #2"):

1. **Deep investigation:**
   - Pull full historical trend for this specific issue from Datadog
   - Read the relevant code in the current repo
   - Check git log for recent changes that might have caused or worsened it
   - Present findings and propose a concrete implementation plan

2. **Implement the fix:**
   - Make the code changes
   - Run existing tests to confirm nothing breaks

3. **Prove it locally:**
   - Consult `references/local-benchmarking.md` for the appropriate method
   - Run the benchmark before AND after (or capture before-state first if needed)
   - Save results to `~/.claude/snowball-performance-log/evidence/fix-[number]-local-benchmark.md`

4. **Log the in-progress entry:**
   - Determine the next fix number from `log.md`
   - Capture current Datadog metrics as the "before" baseline
   - Save raw Datadog API response to `~/.claude/snowball-performance-log/evidence/fix-[number]-before.json`
   - Append an "In Progress" entry to `log.md` following the schema in `references/log-schema.md`

5. **Create PR:**
   - Include a "Performance Fix" section in the PR description:

   ```
   ## Performance Fix
   **Issue:** [description]
   **Impact:** [request volume]/day, [key metric] [value]
   **Evidence:** [Datadog deep link(s)]

   ### Local Benchmark Results
   **Before:** [local benchmark results]
   **After:** [local benchmark results]
   **How to reproduce locally:**
   [step-by-step commands]

   ### Production Verification
   **Expected improvement:** [target metric value]
   **How to verify:** [Datadog link to check ~2 days after merge]
   ```

   - Remind the user to check results in a few days

### snowball log

Log production results for a completed fix.

**Invocations:** "snowball log #[number]", "snowball, log results for fix #[number]"

**Process:**

1. Read the specified fix entry from `log.md`
2. Pull fresh Datadog data for the relevant metrics (same queries used for the "before" baseline)
3. Save raw results to `~/.claude/snowball-performance-log/evidence/fix-[number]-after.json`
4. Calculate impact:
   - Percentage improvement for each metric
   - Translate to business terms (requests affected * improvement = user wait time saved, etc.)
5. Update the entry in `log.md` from "In Progress" to "Completed" with after metrics, impact, and business impact
6. Present the results to the user

### snowball summary

Generate an impact report from the performance log.

**Invocations:** "snowball summary", "snowball, show my impact report", "snowball report"

**Process:**

1. Read all completed entries from `~/.claude/snowball-performance-log/log.md`
2. Generate a report with three sections (see `references/log-schema.md` for full format):

   **Technical Metrics:**
   - Total latency reduction across all fixes (sum of p95 improvements)
   - Total database query time/count reduction
   - Error rate reductions

   **Business Metrics:**
   - Total requests affected per day across all fixes
   - Cumulative user wait time saved daily
   - Page load improvements
   - Infrastructure cost implications
   - Reliability improvements

   **Cumulative Narrative:**
   - Total improvements made: [count]
   - Period: [first fix date] to [last fix date]
   - Top 3 biggest wins (ranked by business impact)
   - Overall trend

3. Format the report for easy sharing (clean markdown, suitable for pasting into a performance review, Slack message, or document)

### snowball retro

Capture feedback after a sweep or fix session to improve the plugin over time.

**Invocations:** "snowball retro", "snowball, what should we improve?"

**Process:** Run `/snowball-retro` command. This collects feedback on what worked, what didn't, and new ideas. Feedback is saved to `~/.claude/snowball-performance-log/retro.md` and actionable items are added to the plugin's `IMPROVEMENTS.md` backlog.

## Additional Resources

### Reference Files

Consult these for detailed guidance:
- **`references/datadog-queries.md`** -- Query patterns for Datadog APM, DBM, RUM, metrics; deep link URL construction
- **`references/local-benchmarking.md`** -- Methods for proving fixes locally (Rails benchmarks, EXPLAIN ANALYZE, Lighthouse, React Profiler)
- **`references/log-schema.md`** -- Performance log entry schema, evidence file formats, summary report structure
