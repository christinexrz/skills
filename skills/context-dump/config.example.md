---
skill: context-dump-shared
configured: true
# generated: <stamped on first run, e.g. 2026-06-30>
---

# Context Dump — Configuration

This file tells the `context-dump-shared` skill where to look. The skill writes
your real config to `config.local.md` (NOT this file) on first run. You can also
copy this file to `config.local.md` and fill it in by hand.

`config.local.md` is per-user and should never be shared or committed. Delete it
to re-run onboarding from scratch.

Every source below is optional. Omit a section (or leave it empty) and the skill
simply skips it in the output.

## Output
# Where the weekly dump is written — a single self-contained HTML file. The skill
# prepends each new dump as a styled card on top of the previous ones (after the
# `<!-- DUMPS:START -->` marker) so the file becomes a running memory — it never
# overwrites past dumps.
path: ~/context-dump.html

## Slack channels
# Channels to scan for the last 7 days. Names or raw IDs both work — the skill
# resolves names to IDs with slack_search_channels. Omit to skip Slack.
- "#your-team-channel"
- "#another-channel"
- "C0XXXXXXX"

## Calendar
# Which calendar MCP server to read meetings from.
#   outlook | google | none
source: outlook

## Notes
# A local folder of dated notes/journal files. Omit to skip.
folder: /absolute/path/to/notes
# Filename pattern for the dated files. Default: YYYY-MM-DD.md
pattern: YYYY-MM-DD.md

## Repos
# Zero or more git repos to summarize recent activity from. Omit to skip.
- path: /absolute/path/to/repo
  # Optional. Ticket IDs matching this prefix are extracted from commit messages.
  ticket_prefix: ABC-
  # Optional. Group commits under these headings. If omitted, the skill falls
  # back to grouping by conventional-commit type (feat / fix / chore / etc.).
  categories:
    - Infrastructure
    - UI
    - Fixes

## Goals
# Your current objectives, used by the Goal Alignment analysis. Paste them here,
# or point to a doc with `path: /absolute/path/to/goals.md`.
- "Objective 1: ..."
- "Objective 2: ..."
- "Objective 3: ..."
