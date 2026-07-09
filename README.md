# Skills for 10X PMs

[![skills.sh](https://skills.sh/b/christinexrz/skills)](https://skills.sh/christinexrz/skills)

> Weekly context, on autopilot.

A small collection of agent skills for product managers. Each one automates the busywork that eats a PM's week — reconstructing what happened, chasing context across tools, writing the recap — so the time you keep goes to the part only you can do: deciding what matters next.

The bet is simple. AI doesn't replace a PM's judgment; it amplifies it. A 10X PM isn't one who works 10x harder. It's one who spends zero minutes on the things a machine should do, and all of them on the things it can't.

These skills are built from real PM workflow, then generalized so anyone can drop them in.

## Install

```bash
npx skills@latest add christinexrz/skills
```

This works with any agent that supports the [skills](https://skills.sh) format (Claude Code, etc.). It pulls every skill in this repo into your agent.

## Why use it?

A PM's context is scattered. Decisions happen in Slack, time goes to meetings, your own thinking lives in notes, and the work itself lands in git. By Friday it's a blur, and Monday starts with a scramble to reconstruct the week before you can plan the next one.

**context-dump** compiles all of it into a single, scannable weekly brief — in one command.

It doesn't just aggregate. It *grades the week* against your own goals:

- **LNO** — how much of your time was Leverage vs. Neutral vs. Overhead.
- **Goal alignment** — which objectives got attention and which got none.
- **3 Levels of Product Work** — are you balancing Optics, Execution, and Impact, or neglecting one?

Then it proposes concrete next actions and notices patterns a thoughtful colleague would ("you've iterated on the same PRD four times this week — want me to draft it?").

The brief is a **styled, self-contained HTML file** — the visual you see on the landing page is the actual output. Each dump is prepended as a new card on top of the last, so the file becomes a **running memory of your quarter** you can open in any browser, not a one-off you lose track of.

## Reference

- **[context-dump](./skills/context-dump/SKILL.md)** — Weekly context dump across Slack, calendar, dated notes, and one or more git repos, with built-in PM-framework analysis. On first run it asks a few questions and saves your setup, so every run after is a single command.
- More coming soon.

## Configuration

The first time you run a skill it onboards you with a few questions (which Slack channels, calendar source, notes folder, repos, your goals) and writes the answers to a per-user `config.local.md` next to the skill. That file is **gitignored and never shared** — it's just yours. See [`config.example.md`](./skills/context-dump/config.example.md) for the format, and delete `config.local.md` anytime to re-onboard.

That first run also checks whether the skill (or your configured output file) lives inside a git repo, and if so adds `config.local.md` and the output HTML file to that repo's `.gitignore` if they're missing — so personal data doesn't end up committed by accident. Worth a quick manual check of your `.gitignore` if you're installing into an existing tracked project.

Slack and calendar steps need the matching MCP server connected; if one isn't available, the skill notes it and continues with the other sources.

## Landing page

A walkthrough of what `context-dump` produces lives at [`index.html`](./index.html) (publishable via GitHub Pages).

---

Built by [Christine Zhu](https://github.com/christinexrz).
