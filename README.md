# Skills for 10X PMs

[![skills.sh](https://skills.sh/b/christinexrz/skills)](https://skills.sh/christinexrz/skills)

For 10X PMs and builders who want to increase leverage in their work by using AI for administrative work, think with greater clarity, and focus on the right things. 


## Install

```bash
npx skills@latest add christinexrz/skills
```

This works with any agent that supports the [skills](https://skills.sh) format (Claude Code, etc.). It pulls every skill in this repo into your agent.

## How /context-dump works
1. **Onboard once** — the first run asks a few questions (your channels, calendar, notes folder, repos, and goals) to customize and set up the skill. No config file to hand-edit.
2. **Gather the week** — pulls the last 7 days of context from Slack, Calendar, Notes, and Git repos — the 4 scattered sources that make up a PM's week — into one readable brief.
3. **Analyze your effectiveness** — checks your week against your goals and the 3 Levels of Product Work (Execution, Strategy, Optics — from Shreyas), shows where your time went, and recommends how you can be more effective.
4. **Review brief** — writes a 5-minute read in HTML and prepends it to the last one, so the file becomes a running memory of your quarter. Repurpose the content for your "optics" work such as leadership updates and self reviews.

The brief is a **styled, self-contained HTML file** — the visual you see on the landing page is the actual output. Each dump is prepended as a new card on top of the last, so the file becomes a **running memory of your quarter**.

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
