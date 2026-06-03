# Claude Code Status Line (ericraymond fork)

A custom status line for [Claude Code](https://claude.com/claude-code) that displays model info,
token usage, cost, git context, and rate limits in a single compact line. Runs as an external shell
command — no slowdown, no extra tokens.

Forked from [daniel3303/ClaudeCodeStatusLine](https://github.com/daniel3303/ClaudeCodeStatusLine).

## Screenshot

![Status Line Screenshot](screenshot.png)

## What it shows

| Segment | Description |
|---------|-------------|
| **Model (effort)** | Model name with reasoning effort inline, e.g. `Sonnet 4.6 (high)` |
| **Tokens** | Used / total context tokens (% used) |
| **dir@branch** | Current folder, git branch, and staged/unstaged counts |
| **Cost** | Session cost with per-message delta, e.g. `$0.42 (+$0.05)` |
| **5h** | 5-hour rate limit usage percentage and reset time |
| **7d** | 7-day rate limit usage percentage and reset time |
| **Extra** | Extra usage credits spent / limit (if enabled) |
| **Update** | Appears when a new version is available (checked every 24h) |

Usage percentages are color-coded: green (<50%) → yellow (≥50%) → orange (≥70%) → red (≥90%).

## Changes from upstream

- Layout: `Model (effort) | tokens (%) | dir@branch | cost (+delta) | 5h | 7d`
- Session cost display with per-message delta
- Effort level shown inline next to model name instead of as a separate segment
- 5h/7d hidden when no usage data (no placeholder dashes by default)
- `awk` injection fixed: shell vars passed via `-v` instead of string interpolation
- All env vars use `CLAUDECODE_STATUSLINE_` prefix

## Installation

```bash
mkdir -p ~/.claude/statusline
curl -o ~/.claude/statusline/statusline.sh \
  https://raw.githubusercontent.com/ericraymond/ClaudeCodeStatusLine/main/statusline.sh
chmod +x ~/.claude/statusline/statusline.sh
```

Then wire it into Claude Code:

```bash
jq '.statusLine = {"type":"command","command":"~/.claude/statusline/statusline.sh"}' \
  ~/.claude/settings.json > /tmp/sl.json && mv /tmp/sl.json ~/.claude/settings.json
```

Restart Claude Code.

### Updating

```bash
cp /path/to/this/repo/statusline.sh ~/.claude/statusline/statusline.sh
```

Or re-run the `curl` command above. No `settings.json` changes needed.

## Configuration

All options are set via environment variables (e.g. in `~/.zshrc` or `~/.bashrc`).

| Variable | Default | Description |
|----------|---------|-------------|
| `CLAUDECODE_STATUSLINE_SHOW_COST` | `true` | Show session cost and per-message delta |
| `CLAUDECODE_STATUSLINE_SHOW_CLI_VERSION` | `false` | Show Claude CLI version (e.g. `v2.1.161`) |
| `CLAUDECODE_STATUSLINE_CHECK_UPDATES` | `true` | Check GitHub for new releases every 24h |
| `CLAUDECODE_STATUSLINE_SHOW_USAGE_PLACEHOLDERS` | `false` | Show `5h -` / `7d -` when no usage data |

When an update is available, a second line appears below the status bar. The check hits
`api.github.com` once per 24h and fails silently if unreachable.

Example — hide cost, show CLI version:

```bash
export CLAUDECODE_STATUSLINE_SHOW_COST=false
export CLAUDECODE_STATUSLINE_SHOW_CLI_VERSION=true
```

## Requirements

- Claude Code with OAuth authentication (Pro/Max subscription for rate-limit and extra-usage data)
- `git` in `PATH`
- macOS / Linux: `jq` and `curl`

## Caching

Usage data is cached for 60 seconds at `/tmp/claude/statusline-usage-cache-<hash>.json`. Release
checks are cached for 24 hours. Both caches are shared across concurrent Claude Code instances to
avoid rate limits.

## License

MIT

## Credits

Original by [Daniel Oliveira](https://github.com/daniel3303/ClaudeCodeStatusLine).
