---
name: context-dump
description: Generate a weekly context dump for a PM — a consolidated summary of the last 7 days across Slack, calendar meetings, dated notes, and one or more git repos, plus PM-framework analysis (LNO, goal alignment, 3 levels of product work). Use when the user runs /context-dump or asks for a weekly summary / context dump. On first run it onboards the user with a few questions and saves their setup so future runs are one command.
user-invocable: true
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion, mcp__claude_ai_Slack__slack_read_channel, mcp__claude_ai_Slack__slack_search_channels, mcp__claude_ai_Microsoft_365__outlook_calendar_search, mcp__claude_ai_Microsoft_365__read_resource, mcp__claude_ai_Google_Calendar__list_events
---

# Weekly Context Dump

Generate a consolidated summary of the last 7 days across the sources a PM cares about — Slack, calendar meetings, dated notes/journal, and git repos — then layer on lightweight PM analysis. The skill is driven entirely by a per-user config file, so it works for anyone who installs it.

**Sources are optional.** Whatever the user didn't configure is simply skipped. Calendar and Slack steps depend on the relevant MCP server being connected; if a tool isn't available, note it in the output rather than erroring out.

The config lives in this skill's own directory: `config.local.md` (next to this `SKILL.md`). The format is documented in `config.example.md`.

## Step 0: Check Config / Onboard

First, locate this skill's directory (the folder containing this `SKILL.md` — wherever the installer placed it, e.g. `~/.claude/skills/context-dump/` or `.github/skills/context-dump/`) and check whether `config.local.md` exists there.

- **If `config.local.md` exists:** read it, state in one line what's configured (e.g. "Loaded config: 5 Slack channels, Google calendar, notes folder, 1 repo, 3 goals"), then proceed to Step 1.

- **If `config.local.md` does NOT exist:** run onboarding. Do not error trying to read a missing file. Use `AskUserQuestion` to collect the inputs below (batch related questions; always offer a "skip this source" choice). Read `config.example.md` first so the saved file matches that format.

  Onboarding questions:
  1. **Output path** — where to write the weekly dump. This is a self-contained HTML file. Default suggestion: `~/context-dump.html`.
  2. **Slack channels** — which channels (names or IDs) to scan, or skip Slack. Mention that names get resolved to IDs automatically.
  3. **Calendar source** — Outlook, Google, or none.
  4. **Notes/journal folder** — absolute path to a folder of dated files, plus the filename pattern (default `YYYY-MM-DD.md`), or skip.
  5. **Git repo(s)** — one or more repo paths. For each, optionally a ticket prefix (e.g. `GUX-`) and custom commit categories. Or skip.
  6. **Goals/objectives** — paste the user's current objectives, or a path to a goals doc, for the goal-alignment analysis.

  Then write `config.local.md` in this skill's directory using the structure from `config.example.md`, stamping `generated:` with today's date. Confirm what was saved, and tell the user they can edit `config.local.md` anytime or delete it to re-onboard. Then proceed to Step 1.

## Step 1: Gather Slack Highlights

If no Slack channels are configured, skip this section. Otherwise use the `slack_read_channel` MCP tool to read the last 7 days of messages from each configured channel. If a channel is given by name and can't be found, use `slack_search_channels` to resolve it to an ID first.

Synthesize all channels into a **single unified summary** of the most important things that happened across Slack this week. Do NOT break out per-channel sections. Write 5-10 bullets covering the key decisions, blockers, announcements, and action items, noting the source channel inline when relevant (e.g., "migration scope confirmed (in #team-channel)").

Focus on what matters for a PM tracking team progress. Skip routine messages (oncall rotations, PR review requests, OOO notices).

## Step 2: Gather Calendar Meetings

Branch on the configured calendar source:
- **outlook** — use `outlook_calendar_search` to find all meetings from the last 7 days. If a meeting has detailed notes or an agenda attached, use `read_resource` for more context.
- **google** — use `list_events` to find all meetings from the last 7 days.
- **none** — skip this section.

Produce a **high-level summary in a few bullets** covering:
- The types/themes of meetings this week (e.g., "cross-team alignment", "design reviews", "sprint ceremonies")
- Any notable decisions, outcomes, or action items worth calling out specifically

Do NOT list every meeting individually. Think "meeting themes and patterns" not "meeting log."

## Step 3: Gather Notes / Journal

If no notes folder is configured, skip this section. Otherwise read the entries from the last 7 days in the configured folder. Files follow the configured filename pattern (default `YYYY-MM-DD.md`).

Use `Glob` to find files matching the last 7 days, then `Read` each one.

Synthesize the entries into a **few high-level bullets** covering:
- Key progress made during the week (decisions landed, docs completed, features shipped, etc.)
- Important things to pay attention to or follow up on next week

Do NOT replay each day individually. Aim for "what moved forward" and "what needs attention." If no entries exist for the period, note "No notes found."

## Step 4: Gather Git Activity

If no repos are configured, skip this section. Otherwise, for each configured repo path, run:

```bash
git fetch --all --quiet
git log --since="7 days ago" --oneline --all
```

Also check for new tags in the last 7 days:

```bash
git tag --sort=-creatordate | head -5
```

Group commits using the repo's configured `categories`. If none are configured, fall back to grouping by conventional-commit type (**Features**, **Fixes**, **Chores/Infra**, **Tests/DevX**, **Other**). Skip empty groups.

If a `ticket_prefix` is configured, extract matching ticket IDs (e.g. `ABC-1234`) and PR numbers from commit messages. Skip `chore(release)` commits. When multiple repos are configured, label each repo's section with its name.

## Step 5: Analyze & Recommend

Synthesize all gathered data (Steps 1-4) into 4 analyses:

1. **Time Allocation vs. LNO** — Classify the week's activities (from notes + meetings + Slack) into Leverage / Neutral / Overhead. Flag if overhead is creeping above ~10% or leverage work is below ~70%.

2. **Goal Alignment Check** — Map the week's work to the configured goals/objectives. Flag goals that got zero attention. Highlight where effort is concentrated vs. spread thin.

3. **3 Levels of Product Work** — Categorize activities into Optics (visibility, stakeholder management), Execution (shipping), and Impact (outcomes). Note if any level is being neglected.

4. **Things Claude Can Help With** — Based on patterns observed this week, suggest 3-5 specific things Claude can help with. These should feel like a thoughtful colleague noticing patterns and offering to take something off the user's plate — specific to what was actually observed, not generic advice. For **each** suggestion, produce two things: (a) a one-line **observation** of the pattern, and (b) a **ready-to-paste prompt** the user can drop straight into Claude to get that help. The prompt should be concrete and self-contained (name the actual doc / decision / goal), one or two sentences. Examples:
   - Observation: "You've iterated on the same PRD across 4 notes entries." → Prompt: "Consolidate my {PRD name} iterations from this week into one clean, sign-off-ready draft and list any open decisions at the top."
   - Observation: "You had 3 meetings about routing with no decision doc." → Prompt: "Synthesize this week's routing discussions into a one-page decision doc with options, a recommendation, and open questions."
   - Observation: "No progress on Goal 3 this week." → Prompt: "Suggest one 30-minute action tied to {Goal 3} this week plus a 3-link reading list."

## Step 6: Write the Output (HTML)

The output is a **single self-contained HTML file** (embedded CSS + one small copy script, no external assets) that accumulates dumps as stacked cards, newest on top. Each run adds one `<article class="dump doc">` card.

### 6a. New file vs. existing file

Check whether the configured output HTML file already exists.

- **If it does NOT exist:** `Write` the full scaffold from **6b**, with your new dump card as the only `<article>` between `<!-- DUMPS:START -->` and `<!-- DUMPS:END -->`.
- **If it DOES exist:** `Read` it, then use `Edit` to insert your new `<article class="dump doc"> … </article>` block **immediately after the `<!-- DUMPS:START -->` marker line** (so the newest dump renders on top). Change nothing else — leave the head, CSS, older dumps, and script untouched. **Never delete past dumps.**

### 6b. Full-file scaffold (only when creating the file)

Write exactly this shell, inserting your dump card where marked:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Weekly Context Dumps</title>
<style>
  :root {
    --bg:#fff; --bg-soft:#f5efe4; --ink:#3d2817; --ink-soft:#7a6a57; --ink-faint:#a89b8c;
    --line:#ece4d6; --line-strong:#e4dccb; --accent:#b55a30; --gold:#d4a01f; --gold-ink:#96700f;
    --card:#fff; --shadow-lg:0 2px 4px rgba(61,40,23,.06),0 18px 50px rgba(61,40,23,.11);
    --font:-apple-system,BlinkMacSystemFont,"Segoe UI",Inter,Roboto,Helvetica,Arial,sans-serif;
    --mono:ui-monospace,SFMono-Regular,"SF Mono",Menlo,Consolas,monospace;
  }
  *{box-sizing:border-box;} body{margin:0;background:var(--bg-soft);color:var(--ink);font-family:var(--font);line-height:1.6;-webkit-font-smoothing:antialiased;}
  .wrap{max-width:820px;margin:0 auto;padding:48px 24px 80px;}
  .page-title{font-size:30px;line-height:1.15;letter-spacing:-.02em;margin:0 0 6px;font-weight:700;}
  .page-sub{font-size:14px;color:var(--ink-faint);margin:0 0 32px;}
  .doc{border:1px solid var(--line-strong);border-radius:16px;background:var(--card);box-shadow:var(--shadow-lg);overflow:hidden;margin-bottom:30px;}
  .doc-bar{display:flex;align-items:center;gap:7px;padding:13px 16px;border-bottom:1px solid var(--line);background:var(--bg-soft);}
  .doc-bar i{width:11px;height:11px;border-radius:50%;background:#e0d8c8;display:inline-block;}
  .doc-bar .fname{margin-left:8px;font-family:var(--mono);font-size:12.5px;color:var(--ink-faint);}
  .doc-body{padding:26px 30px 30px;}
  .doc-body .title{font-size:21px;font-weight:700;margin:0 0 2px;letter-spacing:-.01em;}
  .doc-body .gen{font-size:13px;color:var(--ink-faint);margin:0 0 20px;}
  .doc h4{font-size:12px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;color:var(--ink-faint);margin:22px 0 10px;display:flex;align-items:center;gap:8px;}
  .doc h4:first-of-type{margin-top:0;} .doc h4 svg{width:15px;height:15px;}
  .doc ul,.doc ol{margin:0;padding-left:18px;} .doc li{font-size:14.5px;color:var(--ink);margin-bottom:7px;}
  .doc li .ch{color:var(--accent);font-weight:600;}
  .divider{height:1px;background:var(--line);margin:22px 0;border:0;}
  .doc-group{font-size:15px;font-weight:700;letter-spacing:-.01em;color:var(--accent);margin:26px 0 4px;padding-top:18px;border-top:1px solid var(--line-strong);display:flex;align-items:center;gap:8px;}
  .doc-group svg{width:16px;height:16px;}
  .stat{font-family:var(--mono);font-size:12.5px;color:var(--ink-faint);margin:0 0 12px;display:flex;flex-wrap:wrap;gap:6px 10px;}
  .stat b{color:var(--ink-soft);font-weight:600;}
  .theme{font-size:13px;font-weight:700;color:var(--ink);margin:12px 0 4px;} .theme:first-of-type{margin-top:0;}
  .verdict{font-size:13px;font-style:italic;color:var(--ink-soft);margin:10px 0 0;padding-left:12px;border-left:2px solid var(--line-strong);}
  .balance{margin:4px 0 0;} .balance-row{display:flex;align-items:baseline;gap:10px;font-size:14.5px;margin-bottom:8px;}
  .balance-row .lbl{font-weight:600;min-width:78px;} .balance-row .txt{color:var(--ink-soft);font-size:13.5px;}
  .lno{margin:4px 0 8px;} .lno-track{display:flex;height:26px;border-radius:7px;overflow:hidden;border:1px solid var(--line);}
  .lno-track div{display:flex;align-items:center;justify-content:center;font-size:11.5px;font-weight:700;color:#fff;}
  .lno-l{background:#5f7a4a;} .lno-n{background:#d4a01f;} .lno-o{background:#b55a30;}
  .lno-key{display:flex;gap:16px;margin-top:9px;font-size:12.5px;color:var(--ink-soft);}
  .lno-key span{display:inline-flex;align-items:center;gap:6px;} .lno-key i{width:9px;height:9px;border-radius:2px;display:inline-block;}
  table{width:100%;border-collapse:collapse;margin-top:4px;}
  th,td{text-align:left;font-size:13.5px;padding:8px 10px;border-bottom:1px solid var(--line);}
  th{color:var(--ink-faint);font-weight:600;font-size:11.5px;text-transform:uppercase;letter-spacing:.05em;} td:first-child{font-weight:600;}
  .tag{font-size:11.5px;font-weight:700;padding:2px 9px;border-radius:999px;}
  .tag-ok{background:#e9efe1;color:#4a5d3f;} .tag-gap{background:#f7e8de;color:#b55a30;} .tag-heavy{background:#faf0d5;color:#96700f;}
  .help-item{margin:0 0 14px;} .help-note{font-size:14.5px;color:var(--ink);margin:0 0 8px;}
  .prompt-block{display:flex;align-items:flex-start;gap:10px;background:var(--bg-soft);border:1px solid var(--line-strong);border-radius:10px;padding:12px 12px 12px 14px;}
  .prompt-block code{font-family:var(--mono);font-size:13px;line-height:1.5;color:var(--ink);flex:1;white-space:pre-wrap;}
  .copy-btn{position:relative;display:inline-flex;align-items:center;justify-content:center;width:32px;height:32px;padding:0;flex:none;background:rgba(255,255,255,.1);border:1px solid rgba(255,255,255,.18);border-radius:8px;color:#fff;cursor:pointer;transition:background .2s,border-color .2s,transform .1s;}
  .copy-btn:active{transform:scale(.92);}
  .copy-btn .ico{position:absolute;inset:0;display:inline-flex;align-items:center;justify-content:center;transition:opacity .18s ease,transform .22s cubic-bezier(.3,1.5,.5,1);}
  .copy-btn .ico svg{width:15px;height:15px;} .copy-btn .ico-check{opacity:0;transform:scale(.4) rotate(-15deg);color:#3fbf78;}
  .copy-btn.copied{background:rgba(63,191,120,.18);border-color:rgba(63,191,120,.5);}
  .copy-btn.copied .ico-copy{opacity:0;transform:scale(.4);} .copy-btn.copied .ico-check{opacity:1;transform:scale(1) rotate(0);}
  .copy-btn-light{background:transparent;border-color:var(--line-strong);color:var(--ink-soft);width:30px;height:30px;}
  .copy-btn-light:hover{background:#fff;color:var(--accent);} .copy-btn-light .ico svg{width:14px;height:14px;}
  .copy-btn-light.copied{background:rgba(63,191,120,.13);border-color:rgba(63,191,120,.5);}
  @media (max-width:680px){.doc-body{padding:20px;}}
</style>
</head>
<body>
<div class="wrap">
  <h1 class="page-title">Weekly Context Dumps</h1>
  <p class="page-sub">A running memory of the quarter · newest on top</p>
  <!-- DUMPS:START -->
  <!-- NEW DUMP CARD GOES HERE -->
  <!-- DUMPS:END -->
</div>
<script>
  (function(){
    function fallback(t){var a=document.createElement('textarea');a.value=t;a.style.position='fixed';a.style.opacity='0';document.body.appendChild(a);a.select();try{document.execCommand('copy');}catch(e){}document.body.removeChild(a);}
    function copy(t,d){if(navigator.clipboard&&navigator.clipboard.writeText){navigator.clipboard.writeText(t).then(d,function(){fallback(t);d();});}else{fallback(t);d();}}
    document.querySelectorAll('[data-copy]').forEach(function(b){var tm;b.addEventListener('click',function(){copy(b.getAttribute('data-copy'),function(){b.classList.add('copied');clearTimeout(tm);tm=setTimeout(function(){b.classList.remove('copied');},1600);});});});
  })();
</script>
</body>
</html>
```

### 6c. The dump card

Build one `<article class="dump doc">` following this structure. **Omit any `<h4>` section whose source wasn't configured or had no data.** Copy the inline `<svg>` icon markup verbatim from the matching heading in the reference `index.html` (Slack, Notes, GitHub, clock=LNO, flag=Goal, boxes=Balance, checklist=Actions, bulb=Claude) so icons render identically.

```html
<article class="dump doc">
  <div class="doc-bar"><i></i><i></i><i></i><span class="fname">context-dump.html</span></div>
  <div class="doc-body">
    <div class="title">Weekly Context Dump: {start} to {end}</div>
    <div class="gen">Generated on {today}</div>

    <h4>{slack svg} Slack highlights</h4>
    <ul><li>Key decision or blocker <span class="ch">(#channel)</span></li> …5-10 bullets…</ul>
    <hr class="divider" />

    <h4>{notes svg} Meetings</h4>
    <ul><li><span class="ch">Decision:</span> …</li><li><span class="ch">Watch:</span> …</li></ul>
    <hr class="divider" />

    <h4>{notes svg} Notes</h4>
    <ul><li><span class="ch">Progress:</span> …</li></ul>
    <hr class="divider" />

    <h4>{github svg} GitHub activity</h4>
    <div class="stat"><span><b>N</b> commits</span><span>·</span><span><b>N</b> PRs merged</span><span>·</span><span>latest release <b>vX</b></span></div>
    <div class="theme">Features</div><ul><li>… (ABC-1234, #PR)</li></ul>
    <div class="theme">Fixes</div><ul><li>…</li></ul>
    <div class="theme">New releases</div><ul><li>{tag} — {note}</li></ul>

    <div class="doc-group">{chart svg} Weekly Analysis &amp; Recommendations</div>

    <h4>{clock svg} Time allocation (<a href="https://coda.io/@shreyas/lno-framework">LNO</a>)</h4>
    <div class="lno">
      <div class="lno-track">
        <div class="lno-l" style="flex:72">Leverage 72%</div>
        <div class="lno-n" style="flex:18">Neutral 18%</div>
        <div class="lno-o" style="flex:10">10%</div>
      </div>
      <div class="lno-key"><span><i style="background:#5f7a4a"></i> Leverage</span><span><i style="background:#d4a01f"></i> Neutral</span><span><i style="background:#b55a30"></i> Overhead</span></div>
    </div>
    <p class="verdict">Verdict: …</p>
    <hr class="divider" />

    <h4>{flag svg} Goal alignment</h4>
    <table>
      <tr><th>Objective</th><th>This week</th><th>Status</th></tr>
      <tr><td>…</td><td>…</td><td><span class="tag tag-ok">On track</span></td></tr>
      <!-- tag-ok = On track, tag-heavy = Heavy, tag-gap = Gap -->
    </table>
    <hr class="divider" />

    <h4>{boxes svg} Product work balance</h4>
    <div class="balance">
      <div class="balance-row"><span class="lbl">Optics</span><span class="tag tag-ok">Solid</span><span class="txt">…</span></div>
      <div class="balance-row"><span class="lbl">Execution</span><span class="tag tag-ok">Solid</span><span class="txt">…</span></div>
      <div class="balance-row"><span class="lbl">Impact</span><span class="tag tag-heavy">Watch</span><span class="txt">…</span></div>
    </div>
    <hr class="divider" />

    <h4>{checklist svg} Recommended actions for next week</h4>
    <ol><li><strong>Action</strong> — why it matters.</li></ol>
    <hr class="divider" />

    <h4>{bulb svg} Things Claude can help with</h4>
    <div class="help-item">
      <p class="help-note">{one-line observation of the pattern}</p>
      <div class="prompt-block">
        <code>{ready-to-paste prompt}</code>
        <button class="copy-btn copy-btn-light" data-copy="{ready-to-paste prompt, HTML-escaped}" aria-label="Copy prompt"><span class="ico ico-copy"><svg viewBox="0 0 24 24" fill="none"><rect x="9" y="9" width="11" height="11" rx="2" stroke="currentColor" stroke-width="1.8"/><path d="M5 15V5a2 2 0 0 1 2-2h8" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/></svg></span><span class="ico ico-check"><svg viewBox="0 0 24 24" fill="none"><path d="M4 13l5 5L20 6" stroke="currentColor" stroke-width="2.8" stroke-linecap="round" stroke-linejoin="round"/></svg></span></button>
      </div>
    </div>
    <!-- repeat .help-item per suggestion (3-5) -->
  </div>
</article>
```

Set the three `.lno-*` `style="flex:N"` values to the actual percentages. In each `data-copy` attribute, HTML-escape the prompt (`&quot;` for `"`, `&amp;` for `&`); keep the visible `<code>` text human-readable. The `<code>` text and the `data-copy` value must match.

## Guidelines

- Date range is always "7 days ago" through "today" based on the current date.
- Skip any section whose source isn't configured. If a configured Slack channel has no meaningful activity, note "No significant activity" rather than dropping it.
- If a required MCP server (Slack, Outlook, Google Calendar) isn't connected, note that in the output and continue with the other sources.
- Keep the entire document scannable — a PM should read it in under 5 minutes.
- Always prepend the new dump card right after the `<!-- DUMPS:START -->` marker so newest renders on top; never delete past dumps or touch the head/CSS/script.
- Analysis should reference framework names when applicable (e.g., "per LNO framework", "per 3 Levels of Product Work") and stay concrete and actionable, not generic.
- Keep the analysis section to roughly one page.
