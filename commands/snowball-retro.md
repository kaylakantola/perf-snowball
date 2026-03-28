---
description: Capture what worked and what to improve after a snowball session
---

# Snowball Retro

Run after a weekly sweep or fix session to capture feedback on the plugin workflow itself. This builds up an improvement backlog over time.

## Process

Ask the user three questions, one at a time:

1. **"What worked well in this session?"**
   - Examples: "The Datadog queries found good candidates", "Local benchmarking was easy to follow", "The PR description format was helpful"

2. **"What was frustrating or could be better?"**
   - Examples: "The sweep missed frontend issues", "I had to manually construct the Datadog link", "It didn't detect the right service automatically"

3. **"Any new ideas for the plugin?"**
   - Examples: "Support for New Relic", "Auto-detect N+1 patterns in code", "Weekly Slack summary"

After collecting responses, append an entry to `~/.claude/snowball-performance-log/retro.md`:

```markdown
## Retro -- [YYYY-MM-DD]

**What worked:**
[user response]

**What to improve:**
[user response]

**Ideas:**
[user response]
```

If `retro.md` doesn't exist yet, create it with the header:

```markdown
# Snowball Retro Log

Feedback from weekly sessions to guide plugin improvements.

---
```

Then, analyze the feedback and determine if any items are concrete, actionable improvements to the plugin. If so, append them to `IMPROVEMENTS.md` in the plugin repo (at `${CLAUDE_PLUGIN_ROOT}/IMPROVEMENTS.md`). Each improvement entry:

```markdown
### [short title]
- **Source:** Retro [YYYY-MM-DD]
- **Type:** [bug / enhancement / new-feature / reference-update]
- **Component:** [skill / command / reference / mcp-config]
- **Description:** [what to change and why]
```

Finally, report:
> "Retro saved to `~/.claude/snowball-performance-log/retro.md`. [N] improvement ideas added to the plugin backlog."
