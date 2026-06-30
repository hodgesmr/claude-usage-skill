# claude-usage-skill

A Claude Code plugin with one Agent Skill. It lets Claude check its own plan usage and rate limits on demand, so it can see how close it is to a limit and ease off on expensive work when a window is filling up.

It works by calling Claude Code's own `/usage` report through the CLI. Claude Code handles its own authentication, so the skill reads no credentials. There is no status line and no background process, and it writes nothing to your settings that you have to maintain.

## How it works

The skill's body is a single line:

```
!`claude -p "/usage"`
```

When Claude invokes the skill, that runs the CLI without an interactive session, and `/usage` prints the live report into the turn:

```
Current session: 3% used · resets Jun 30 at 12:09pm
Current week (all models): 6% used · resets Jul 2 at 8:59pm
Current week (Sonnet only): 0% used · resets Jul 2 at 8:59pm

What's contributing to your limits usage?
Last 24h · 206 requests · 4 sessions
  80% of your usage came from subagent-heavy sessions
  ...
```

That covers the session window, the weekly all-models window, and the weekly Sonnet-only window, each with a reset time in your local zone and a breakdown of what is driving usage. It returns in about 1.5 seconds. Because `/usage` is rendered by the client rather than the model, running it costs almost no model tokens.

The skill leaves model invocation on, so its description loads into context each session and Claude can reach for it when usage is relevant. You can also run it yourself as `/claude-usage:get-usage`.

## Install

You can install this plugin either through Claude Desktop's Code tab, or directly in your terminal if you use Claude Code.

```bash
# First, tell Claude where to find this plugin
claude plugin marketplace add hodgesmr/claude-usage-skill

# Next, install the plugin
claude plugin install claude-usage@claude-usage-skill
```

_Note: You may need to restart the app or start a new Code session for the new features to fully load._

**Updating:**
If you need to update to a newer version in the future, run these commands:

```bash
claude plugin marketplace update claude-usage-skill
claude plugin update claude-usage@claude-usage-skill
```

## Usage

Once installed you can do:

```text
❯ what is your current plan usage?
```

And Claude can now report back: 

```text
I'll check your current plan usage.

⏺ Skill(claude-usage:get-usage)
Successfully loaded skill · 1 tool allowed

Here's your current plan usage (you're on a Claude subscription):

┌────────────────────────────┬──────┬─────────────────────┐
│           Window           │ Used │       Resets        │
├────────────────────────────┼──────┼─────────────────────┤
│ Current session (5h)       │ 13%  │ Jun 30, 12:09pm CDT │
├────────────────────────────┼──────┼─────────────────────┤
│ Current week (all models)  │ 7%   │ Jul 2, 8:59pm CDT   │
├────────────────────────────┼──────┼─────────────────────┤
│ Current week (Sonnet only) │ 0%   │ Jul 2, 9:00pm CDT   │
└────────────────────────────┴──────┴─────────────────────┘

You've got plenty of headroom across all three windows.

What's driving usage recently:
- Last 24h: 320 requests across 7 sessions — 89% from subagent-heavy sessions, 59% at >150k context
- Last 7d: 1,047 requests across 14 sessions — 82% at >150k context, 74% from long (8+ hour) sessions

The main cost drivers are large-context, long-running, subagent-heavy sessions. Nothing to worry about right now given how low all three windows are, but that's where to ease off if a window starts filling up.
```

You can use this on-demand, or chain it together with other Skills or agents to intelligently tune your usage of high-token operations like workflows, Agent Teams, etc.

## Requirements

- The `claude` CLI on `PATH`, authenticated, since the skill calls it.
- A Pro or Max plan, which is what `/usage` reports on.

## Notes

The skill runs when invoked, not continuously. Each call starts a short-lived `claude` subprocess of about 1.5 seconds, which is fine for checking when it matters rather than every turn. Each invocation is itself a small session that nudges the counters it reads, by a negligible amount.

The rate-limit percentages are account-wide. The breakdown of what is driving usage is approximate and covers only local sessions on this machine, so it does not count other devices or claude.ai.

It depends on `claude -p "/usage"` returning the report as a client-side command with no model turn, which is how it behaves today. If a future version sent `/usage` through the model in print mode, the skill would get slower and start costing tokens. The fix would be one line in `SKILL.md`.

## Uninstall

```bash
claude plugin uninstall claude-usage@claude-usage-skill
```

## License

BSD 3-Clause. See [LICENSE](LICENSE).
