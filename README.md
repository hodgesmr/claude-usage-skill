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

## What's in here

| Path | Role |
|------|------|
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.claude-plugin/marketplace.json` | Lets this repo serve as its own one-plugin marketplace |
| `skills/get-usage/SKILL.md` | The skill: runs `claude -p "/usage"` and pulls the report into context |
| `README.md` | This file |

## Install

As a plugin. The repo is `claude-usage-skill`; the plugin it ships is named `claude-usage`.

```bash
claude plugin marketplace add hodgesmr/claude-usage-skill
claude plugin install claude-usage@hodgesmr
```

Or from inside Claude Code, run `/plugin marketplace add hodgesmr/claude-usage-skill` then `/plugin install claude-usage@hodgesmr`. Restart and the skill is live.

For local development before the repo is pushed:

```bash
claude --plugin-dir .
```

## Requirements

- The `claude` CLI on `PATH`, since the skill calls it.
- A Pro or Max plan, which is what `/usage` reports on.

## Notes

The skill runs when invoked, not continuously. Each call starts a short-lived `claude` subprocess of about 1.5 seconds, which is fine for checking when it matters rather than every turn. Each invocation is itself a small session that nudges the counters it reads, by a negligible amount.

The rate-limit percentages are account-wide. The breakdown of what is driving usage is approximate and covers only local sessions on this machine, so it does not count other devices or claude.ai.

It depends on `claude -p "/usage"` returning the report as a client-side command with no model turn, which is how it behaves today. If a future version sent `/usage` through the model in print mode, the skill would get slower and start costing tokens. The fix would be one line in `SKILL.md`.

## Uninstall

```bash
claude plugin uninstall claude-usage@hodgesmr
```
