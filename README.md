# today-command

A Claude Code slash command that generates a morning briefing to start your day with clarity instead of paralysis.

Inspired by [Teresa Torres'](https://www.producttalk.org/) workflow with AI, adapted for people who struggle with focus and tend to freeze in front of long lists.

## What it does

Every morning, you type `/today` in Claude Code and it builds a single markdown briefing that pulls together:

- Events from your **Google Calendar** (today's meetings, with times and links)
- Open tasks from **markdown files** in your notes folder (Obsidian, plain notes, whatever you use)
- Current sprint in **Notion** (optional)
- 3 curated reads from your favorite sources

And delivers a `today.md` file in Obsidian-friendly format with:

- **Previous day review** — before building today's plan, asks you what happened yesterday (done / partial / didn't do) so carryovers are intentional, not assumed
- **One concrete action** to start in the next 15 minutes
- **The 3 priorities for today** (not the whole backlog)
- **Today's agenda** — meetings only, each with a 1-sentence prep line (no fake "focus blocks" or scheduled breaks)
- **Where each project stands** — 1 line per active project, so the briefing also works as a quick status check
- **Stuck escape hatch** (because you will get stuck)
- **Wrap-up checklist** to mark what you got done (dopamine)
- **Reading of the Day** connected to today's context
- **Daily history** — each briefing is archived to `daily/YYYY-MM-DD.md` so you build a log over time, with any manual notes you added during the day preserved
- **Flow improvement suggestions** — after generating, suggests 1-2 concrete tweaks to your routine

See [example-output-en.md](example-output-en.md) for a real example of the result. The same command in Portuguese: [example-output-pt.md](example-output-pt.md). The briefing language matches whatever language you usually talk to Claude in.

## Why it exists

I (Gustavo, a UX Designer) tried to copy Teresa Torres' workflow directly and the result was overwhelming. Too much information, no prioritization, and on some days I'd skip the routine entirely because the briefing scared me more than it helped. You can see what that first version looked like in [example-overwhelming.md](example-overwhelming.md) — the briefing that made me freeze.

Then I rewrote the command, kept iterating, and ended up with 4 main changes:

1. **Top 3, not backlog** — Claude has to choose, not list
2. **No fake schedule** — the briefing names *what* matters today, not *when* to do every minute (I control time externally with a pomodoro app; you might use a calendar block, gut feeling, whatever — the briefing doesn't duplicate it)
3. **Phase 0 review of yesterday** — before building today's plan, Claude asks how yesterday went (done / partial / didn't do) and carries over what you choose
4. **Escape hatch for paralysis** — a fixed section for when you freeze

The result became an ally, not a report.

## Install

### Option A — Ask Claude to install it (recommended)

Open Claude Code and paste this:

> Install the today command from https://github.com/Gustavosilveira23/today-command. Download the `today.md` file, save it to my `~/.claude/commands/` folder, then ask me about my notes folder, the projects I want tracked, and my reading sources so you can adapt it to my context.

Claude will download the file, save it, and walk you through a mini-onboarding to customize it. Most aligned with the spirit of the project: let the AI do the boring parts.

### Option B — One-liner (curl)

```bash
# Linux / Mac
curl -o ~/.claude/commands/today.md https://raw.githubusercontent.com/Gustavosilveira23/today-command/main/today.md

# Windows (PowerShell)
curl https://raw.githubusercontent.com/Gustavosilveira23/today-command/main/today.md -o $env:USERPROFILE\.claude\commands\today.md
```

Then open the file and adapt the `> **Adapt here:**` blocks manually.

### Option C — Clone the repo

```bash
git clone https://github.com/Gustavosilveira23/today-command.git
cp today-command/today.md ~/.claude/commands/today.md
```

## Data sources (all optional)

The briefing gets richer the more sources you connect, but **nothing is required**. Use only what fits your organization style.

| Source | What for | How to connect |
|--------|----------|----------------|
| **Markdown files** (Obsidian, plain notes) | Primary task source: project files, `tasks.md`, loose checkboxes | Native (Glob + Grep), no MCP needed |
| **Google Calendar** | Today's events, meeting links | Native cloud MCP |
| **Notion** | Sprint board, task database | Native cloud MCP |
| **Google Tasks** | Pending tasks (optional, only if you already live there) | Community npm MCP |
| **WebFetch / WebSearch** | Curated reads from your sources | Native |

If you keep everything in markdown, you might not need any MCP beyond Google Calendar. **Remove the corresponding sections from `today.md`** for any source you don't use (so Claude doesn't try to fetch from nothing).

> Tip: pick **one** source for tasks (markdown OR Google Tasks), not both. Duplication adds friction. The default in this repo treats markdown as the primary source.

### Google Calendar (native MCP)

1. Open `claude.ai` → Settings → Connected accounts
2. Connect Google Calendar with your Google account

Done. Exposes tools like `gcal_list_events`.

### Notion (native MCP, optional)

1. Open `claude.ai` → Settings → Connected accounts
2. Connect Notion
3. Find the URL of your sprint board view (use `notion-search` and `notion-fetch` to locate it)
4. Paste that URL in section 3 of `today.md`

### Google Tasks (community MCP, optional)

Only worth setting up if Google Tasks is already where you keep tasks. Otherwise, skip — markdown is simpler.

There's no native MCP, so we use a community npm package.

**Google Cloud Console setup** (~15min):
1. Create a project at [console.cloud.google.com](https://console.cloud.google.com)
2. Enable the **Google Tasks API**
3. Create **OAuth 2.0** credentials (type: "Desktop app")
4. On the OAuth consent screen, add your email as a test user
5. Add the scope `https://www.googleapis.com/auth/tasks`

**Install the MCP in Claude Code** (via terminal):

```bash
claude mcp add \
  -e "GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com" \
  -e "GOOGLE_CLIENT_SECRET=your-client-secret" \
  -s user \
  google-tasks -- npx -y mcp-googletasks-vrob
```

**First-time auth:**
- Claude will call the `authenticate` tool automatically the first time you run `/today`
- Browser opens, you authorize with your Google account
- It's saved, no need to do it again

## Why Obsidian works well here (but isn't required)

I built `/today` with Obsidian because that's where my notes live, but the command is just markdown — it works with any editor that reads `.md` files. You could rebuild the same flow against Notion, Apple Notes, plain folders, whatever. I picked Obsidian for these reasons:

- **Wikilinks (`[[Project]]`, `[[Meeting]]`)** — every mention of a project, meeting, or task becomes a graph node, so jumping from the briefing into the project file is one click. The briefing pulls double duty as a navigation hub. In plain editors the syntax renders as text (harmless); in Notion you'd swap for page mentions or database relations.
- **Daily Notes** — Obsidian's Daily Notes plugin uses the same `YYYY-MM-DD.md` filename pattern as the `/today` archive, so each daily briefing shows up in your calendar automatically. Zero extra setup.
- **Frontmatter + Dataview** — the `type: daily`, `tags`, `date` fields make briefings queryable. You can build a dashboard like "all the days I had a Project Atlas meeting" without leaving Obsidian.
- **Local files** — the entire history is a folder of markdown on your disk. No vendor lock-in, no API limits, no "what if the company shuts down".

**If you use Notion instead**: keep your tasks in a Notion database, point section 2 of `today.md` at that DB via the Notion MCP, and either remove the `[[wikilinks]]` or swap them for Notion's `@` mentions in a post-processing step. The briefing logic stays the same — you just lose the local-graph navigation.

**If you use plain notes** (Apple Notes, Google Docs, anything): keep everything as-is but ignore the `[[...]]` syntax — it shows up as plain text and doesn't break anything.

## Customize

Things you can change in `today.md`:

- **Tone**: the default is direct, light, no pressure. Adjust under "Briefing principles"
- **Number of priorities**: default is 3, bump to 5 if you prefer
- **Wikilinks**: harmless plain text if you don't use Obsidian. Remove the `[[...]]` instructions only if you want a cleaner-looking briefing in your editor
- **Fixed sections**: "If you get stuck" and "Wrap-up" are fixed, but you can swap them for whatever fits your routine (gratitude, intention of the day, etc.)
- **Project list**: edit the "Where each project stands" pointer to match your active projects
- **Output language**: the command instructs Claude to generate the briefing in **the same language you normally talk to Claude in**. No flag to flip — just write in your language and the briefing comes out in your language.

## Technical notes

- **Windows**: Local MCPs based on Python/uvx **don't work** in Claude Code running on VS Code Windows. Use npm packages with `claude mcp add` via terminal (that's why Google Tasks here uses the `mcp-googletasks-vrob` package)
- **OAuth in test mode**: On first Google Tasks auth, the app sits in "test mode" in Google Cloud Console. Remember to add your email as a test user, otherwise auth fails
- **Resilience**: `today.md` instructs Claude to keep going even if one source fails (e.g., if Notion doesn't respond, it generates the briefing without sprint data). No need to pray everything works
- **History**: Each briefing is automatically archived to `daily/YYYY-MM-DD.md` during Phase 0 (the review of yesterday), so you never lose a previous day's plan. Any manual notes you added during the day are preserved in the archive.
- **Idempotent**: You can run `/today` multiple times a day — it always overwrites `today.md` (but the first run of the day archives yesterday's)

## Setup time

Total estimate: ~15min if you skip Google Tasks (most people should), ~30-40min if you set everything up.

| Step | Time |
|------|------|
| Connect Google Calendar (native MCP) | ~2min |
| Connect Notion (optional) | ~3min |
| Create Google Cloud project + Tasks OAuth (optional) | ~15min |
| Install and authenticate Google Tasks via npm (optional) | ~5min |
| Copy and adapt `today.md` to your context | ~10min |

## Inspirations

- [Teresa Torres — Continuous Discovery](https://www.producttalk.org/) — origin of the morning briefing habit
- Carina — who introduced me to the workflow
- **Bullet Journal** (Ryder Carroll) — the discipline of choosing what matters today, not what's possible
- [Claude Design Skills](https://github.com/Gustavosilveira23/claude-design-skills) — another repo of mine, same skills/commands pattern

## License

MIT. Take it, adapt it, share it. If it becomes useful in your flow, drop me a line.
