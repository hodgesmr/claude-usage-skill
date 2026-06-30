# claude-usage-skill

A Claude Code plugin with a single Agent Skill, `get-usage`, that reports the user's plan usage and rate limits on demand. The skill shells out to Claude Code's own `/usage` command and pulls the result into context. It reads no credentials and makes no direct API calls.

## Layout

- `.claude-plugin/plugin.json` is the plugin manifest. Its `name` is `claude-usage`.
- `.claude-plugin/marketplace.json` makes the repo its own one-plugin marketplace. The plugin entry `name` must stay equal to `plugin.json`'s `name`, or `claude plugin tag` validation fails.
- `skills/get-usage/SKILL.md` is the whole skill. Its body is one line: `` !`claude -p "/usage"` ``.

## Invariants to keep

- The skill is named `get-usage`, not `usage`, on purpose. A skill named `usage` produces a `/usage` command that collides with Claude Code's built-in `/usage`. Do not rename it to `usage`.
- The slash command is `/claude-usage:get-usage`, formed from the plugin name and the skill name. It changes if either name changes.
- The `allowed-tools` value in the SKILL.md frontmatter must match the command in the body exactly, or invoking the skill prompts for permission instead of running it. If you change the body command, change `allowed-tools` with it.
- The design assumes `claude -p "/usage"` runs `/usage` as a client-side command with no model turn, which is why it is fast and costs almost no tokens. If a future Claude Code routes `/usage` through the model in print mode, that assumption breaks and the skill becomes slow and expensive.
- Keep credential and security framing out of the SKILL.md description. The description loads into context every session, so it should only say what the skill reports and when to invoke it.

## Testing

Load the plugin from disk without installing it and invoke the skill end to end:

```bash
claude --plugin-dir . -p "/claude-usage:get-usage"
```

The usage report should print with no permission prompt. To exercise the full install path, add the repo as a local marketplace, install it, check `claude plugin details claude-usage`, then uninstall and remove the marketplace so no state is left behind.

## Writing style for docs

- No em-dashes. Use commas, periods, or parentheses.
- Plain, direct prose. No meta-narration, no "not just X but Y", no forced rule-of-three, no coined metaphors presented as findings, no aphoristic closing lines.
- Do not hard-wrap the README or this file at a column limit. Write one line per paragraph and let the reader's renderer wrap.
