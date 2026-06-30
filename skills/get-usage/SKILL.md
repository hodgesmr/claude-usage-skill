---
name: get-usage
description: Check the current Claude plan usage and rate limits. Reports the 5-hour session window, the weekly all-models window, and the weekly Sonnet-only window, each with its used percentage and reset time, plus a breakdown of what is driving recent usage. Use this when you need to know how close you are to a usage limit (for example before spawning several subagents or starting long-running background work), when deciding whether to ease off on expensive operations, or when the user asks about plan or rate-limit usage.
allowed-tools: Bash(claude -p "/usage")
compatibility: Requires the Claude Code CLI on PATH, internet access, and a Pro or Max plan
license: BSD-3-Clause
metadata:
  author: Matt Hodges
  version: "0.1.0"
---

Current Claude plan usage and rate limits:

!`claude -p "/usage"`

The block above is Claude Code's own /usage report: the session, weekly (all-models), and weekly (Sonnet-only) windows, each with a used percentage and reset time, followed by a breakdown of what is driving recent usage. The percentages are account-wide; the breakdown is approximate and covers only local sessions on this machine. Use it to judge how much headroom is left before a limit, and ease off on expensive work such as subagents or long background runs when a window is getting full. If the report does not appear, say so rather than guessing a number.
