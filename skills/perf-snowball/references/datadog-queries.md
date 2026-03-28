# Datadog Queries for Performance Analysis

Common queries and patterns for identifying performance issues via the Datadog MCP server. This reference is tailored for a Rails + React + MySQL stack and should be updated when changing jobs or stacks.

## Sweep Queries

### APM: Slowest Endpoints (Last 7 Days)

Use the Datadog MCP server to query APM traces for the configured service(s) and environment. Focus on:
- Endpoints with highest p95/p99 latency
- Endpoints with highest request volume (these have the most user impact even with moderate latency)
- Endpoints where latency has increased compared to the previous 7-day period

Sort by: impact score = (p95 latency) * (daily request volume)

### Database: Slow Queries

Query Datadog DBM (Database Monitoring) for:
- Queries with highest average execution time
- Queries with highest total time (execution time * frequency)
- Queries with full table scans or poor execution plans

For Rails specifically, look for:
- N+1 query patterns: multiple similar SELECT statements within a single request trace
- Missing index indicators: queries on tables where execution time is disproportionate to result size
- Unoptimized ActiveRecord queries: `SELECT *` patterns that could use `.select()` or `.pluck()`

### Error Rates

Query for:
- Endpoints with error rates above 1% (sustained over 7 days)
- Error rate increases compared to previous period
- 5xx errors that correlate with high latency (often indicates timeouts)

### Frontend (RUM)

If Datadog RUM is available for the configured services:
- Pages with worst Core Web Vitals (LCP, FID/INP, CLS)
- Pages with highest load times
- JavaScript errors by frequency and affected user count

### Resource Metrics

If infrastructure metrics are available:
- Services with consistently high CPU utilization (>70% p95)
- Services with memory growth patterns (potential leaks)
- Services with high container restart counts

## Historical Validation Queries

For each candidate issue, verify it's sustained (not a blip) by querying 2-4 weeks of data:
- Compare current week metrics to each of the 3 prior weeks
- Flag if the issue appeared only in the most recent week (likely transient)
- Note if the issue is getting worse over time (trending)

## Deep-Dive Queries

When investigating a specific issue:
- Pull the full trace for the slowest instances of an endpoint
- Get the database query breakdown within those traces
- Check related services for cascading latency
- Look at recent deployments that correlate with performance changes

## Constructing Datadog Deep Links

When presenting issues, construct direct links to the relevant Datadog views. URL patterns:

### APM Trace View
`https://app.{site}/apm/traces?query=service:{service}%20resource_name:{endpoint}&env={env}&start={unix_start}&end={unix_end}`

### APM Service Page
`https://app.{site}/apm/service/{service}?env={env}`

### DBM Query View
`https://app.{site}/databases/queries?search={query_fragment}&env={env}`

### RUM Performance
`https://app.{site}/rum/performance?query=@view.url_path:{path}`

### Dashboards
`https://app.{site}/dashboard/{dashboard_id}`

Replace `{site}` with the configured Datadog site (e.g., `datadoghq.com`), `{service}`, `{env}`, etc. with values from the config file.
