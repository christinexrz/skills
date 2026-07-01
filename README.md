# Skills for 10X PMs

[![skills.sh](https://skills.sh/b/christinexrz/skills)](https://skills.sh/christinexrz/skills)

> Weekly context, on autopilot.

**[View the live demo →](https://christinexrz.github.io/skills/)**

PMs are like context routers. Your week is scattered across Slack, your calendar, meeting notes, and the team's commits. These skills pull the fragmented work context into one short brief, check it against your goals, and evaluate how you can be more effective.

The bet is simple. AI doesn't replace a PM's judgment; it amplifies it. A 10X PM isn't one who works 10x harder. It's one who spends zero minutes on the things a machine should do, and all of them on the things it can't.

These skills are built from real PM workflow, then generalized so anyone can drop them in.

## Install

```bash
npx skills@latest add christinexrz/skills
```

This works with any agent that supports the [skills](https://skills.sh) format (Claude Code, etc.). It pulls every skill in this repo into your agent.

## How it works

1. **Onboard once** — the first run asks a few questions (your channels, calendar, notes folder, repos, and goals) to customize and set up the skill. No config file to hand-edit.
2. **Gather the week** — pulls the last 7 days of context from Slack, Calendar, Notes, and Git repos — the 4 scattered sources that make up a PM's week — into one readable brief.
3. **Analyze your effectiveness** — checks your week against your goals and the 3 Levels of Product Work (Execution, Strategy, Optics — from Shreyas), shows where your time went, and recommends how you can be more effective.
4. **One brief** — writes a 5-minute read and prepends it to the last one, so the file becomes a running memory of your quarter. Repurpose the content for your "optics" work such as leadership updates and self reviews.

## Why use it?

**context-dump** doesn't just aggregate your week. It *grades the week* against your own goals:

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

Slack and calendar steps need the matching MCP server connected; if one isn't available, the skill notes it and continues with the other sources.

## Landing page

A walkthrough of what `context-dump` produces is live at **[christinexrz.github.io/skills](https://christinexrz.github.io/skills/)**, served via GitHub Pages from [`index.html`](./index.html).

---

Built by [Christine Zhu](https://github.com/christinexrz).
