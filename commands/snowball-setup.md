---
description: Configure perf-snowball for your project and Datadog environment
---

# Snowball Setup

Configure the perf-snowball plugin for the current project. This command verifies Datadog connectivity, detects the tech stack, discovers available services, and writes a config file.

## Step 1: Check for Existing Configuration

Read `~/.claude/snowball-performance-log/config.md` if it exists. If a config already exists, ask the user:

> "I found an existing perf-snowball configuration for [repo-name]. Would you like to:"
> 1. Update the existing config
> 2. Add a new project config (saved to `~/.claude/snowball-performance-log/configs/`)
> 3. Start fresh (replaces existing config)

If adding a new project config, ask for a short project name (kebab-case) to use as the filename.

## Step 2: Verify Datadog Connectivity

Check that the Datadog MCP server is available by attempting a simple query (e.g., list monitors or list metrics). If the connection fails:

1. Check if `DATADOG_API_KEY` and `DATADOG_APP_KEY` environment variables are set
2. If not set, explain how to set them:
   > "Set these environment variables in your shell profile (e.g., `~/.zshrc`):
   > ```
   > export DATADOG_API_KEY="your-api-key"
   > export DATADOG_APP_KEY="your-app-key"
   > export DATADOG_SITE="datadoghq.com"  # or your region
   > ```
   > You can find your API and App keys in Datadog under Organization Settings > API Keys / Application Keys."
3. If `DATADOG_SITE` is not set, help determine the correct value by asking what URL the user sees when logged into Datadog in the browser (e.g., `app.datadoghq.com` = US1, `app.datadoghq.eu` = EU1, `us3.datadoghq.com` = US3, `us5.datadoghq.com` = US5, `ap1.datadoghq.com` = AP1)

If the connection succeeds, report:
> "Datadog connection: Connected (site: [site])"

## Step 3: Identify the Repository

Detect the current repo URL using:
```
git remote get-url origin
```

Confirm with the user:
> "Repo detected: [repo-url]. Is this correct?"

## Step 4: Detect Tech Stack

Inspect the codebase for common indicators:
- `Gemfile` or `Gemfile.lock` -> Ruby/Rails (check for rails gem and version)
- `package.json` -> Node.js / React / Next.js (check dependencies)
- `requirements.txt` or `pyproject.toml` -> Python
- `go.mod` -> Go
- Database config files (e.g., `config/database.yml` for Rails) -> MySQL, PostgreSQL, etc.

Present findings:
> "Stack detected: [framework] [version], [frontend] [version], [database]"

Ask user to confirm or correct.

## Step 5: Discover Datadog Services and Environments

Query the Datadog API to list available APM services and environments. Present them with context:

> **Datadog APM services found:**
> 1. [service-name] ([framework], [request-volume]/day)
> 2. [service-name] ([framework], [request-volume]/day)
> ...
>
> Which services should I monitor? (comma-separated numbers, or "all")

Then:
> **Environments found:** [list]
> Which environment should I analyze? (default: production)

## Step 6: Infrastructure Notes

Ask the user:
> "Any infrastructure context I should know about? (e.g., 'runs on Kubernetes in AWS us-east-1', 'uses Redis for caching'). Feel free to skip if unsure -- you can always add this later by re-running /snowball-setup."

## Step 7: Initialize Performance Log

Create the performance log directory and files if they don't exist:

```bash
mkdir -p ~/.claude/snowball-performance-log/evidence
mkdir -p ~/.claude/snowball-performance-log/configs
```

If `~/.claude/snowball-performance-log/log.md` doesn't exist, create it with:

```markdown
# Performance Snowball Log

Running tally of performance improvements. Each entry tracks a fix from identification through measured impact.

---
```

## Step 8: Write Config File

Write the configuration to `~/.claude/snowball-performance-log/config.md` (or `configs/[project-name].md` if adding an additional project):

```markdown
# perf-snowball Configuration

## Project
- **Repo:** [repo-url]
- **Tech stack:** [framework, frontend, database]
- **Infrastructure:** [user-provided notes or "Not specified"]

## Datadog
- **Site:** [datadog-site]
- **APM services:** [comma-separated service names]
- **Environment:** [environment]

## Setup
- **Configured on:** [date]
- **Last updated:** [date]
```

## Step 9: Confirm Setup

> "Setup complete! Here's your configuration:
>
> - **Repo:** [repo]
> - **Stack:** [stack]
> - **Datadog services:** [services]
> - **Environment:** [environment]
> - **Performance log:** ~/.claude/snowball-performance-log/
>
> Run **snowball sweep** to start your first performance analysis.
> Run **/snowball-setup** again anytime to update this configuration."
