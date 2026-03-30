# perf-snowball Improvements Backlog

Improvement ideas captured from `/snowball-retro` sessions and general usage. When working on the plugin, check this file for the next thing to improve.

## How to Use This File

- New entries are appended by the `/snowball-retro` command
- When you pick up an improvement, move it to the "Completed" section with the date
- Prioritize reference updates (datadog-queries.md, local-benchmarking.md) since these evolve most frequently

## Backlog

### Bot traffic detection in sweep triage
- **Source:** Retro 2026-03-30
- **Type:** enhancement
- **Component:** skill
- **Description:** During sweep triage, automatically check Datadog logs for bot vs real-user breakdown (Cloudflare bot score, user agent). Surface this early so the user can correctly assess impact before deep-diving. This session wasted time assuming errors were user-facing before discovering ~85% were bot traffic.

### Impact assessment — silent errors vs user-visible failures
- **Source:** Retro 2026-03-30
- **Type:** enhancement
- **Component:** skill
- **Description:** When investigating an error, guide the user to check whether the error results in a 500/error page or is silently rescued. Query Datadog for HTTP status codes on the affected endpoint alongside the error logs. This session initially assumed a 500 was returned when the error was actually caught by a rescue block.

### Improve initial error volume estimation
- **Source:** Retro 2026-03-30
- **Type:** enhancement
- **Component:** skill
- **Description:** Initial error volume estimates were significantly underestimated. The sweep should query for total error count over the past 7 days with daily breakdown upfront, rather than relying on sampled or partial data. Include guidance to drill down by time-of-day patterns to catch bursty traffic (e.g., crawler schedules).

### Always include timeline in PR descriptions
- **Source:** Retro 2026-03-30
- **Type:** enhancement
- **Component:** reference
- **Description:** Add a "Timeline" section to the snowball PR description template that captures when the issue started, what caused the onset (flag change, deploy, external event), and the pattern of occurrence.

## Completed

_No completed items yet._
