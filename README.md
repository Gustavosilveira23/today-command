# today-command

A Claude Code slash command that generates a morning briefing to start your day with clarity instead of paralysis.

Inspired by [Teresa Torres'](https://www.producttalk.org/) workflow with AI, adapted for people who struggle with focus and tend to freeze in front of long lists.

## What it does

Every morning, you type `/today` in Claude Code and it builds a single markdown briefing that pulls together:

- Events from your **Google Calendar** (today's meetings, with times and links)
- Pending tasks from **Google Tasks** (picks the Top 3, doesn't dump everything)
- Loose to-dos from **markdown files** in your notes folder
- Current sprint in **Notion** (optional)
- 3 curated reads from your favorite sources

And delivers a `today.md` file in Obsidian-friendly format with:

- **One concrete action** to start in the next 15 minutes
- **The 3 priorities for today** (not the whole backlog)
- **Day log** with time blocks and concrete suggestions (no empty "free window" slots)
- **Stuck escape hatch** (because you will get stuck)
- **Wrap-up checklist** to mark what you got done (dopamine)
- **Reading of the Day** connected to today's context

See [example-output-en.md](example-output-en.md) for a real example of the result. The same command in Portuguese: [example-output-pt.md](example-output-pt.md). The briefing language matches whatever language you usually talk to Claude in.

## Why it exists

I (Gustavo, a UX Designer) tried to copy Teresa Torres' workflow directly and the result was overwhelming. Too much information, no prioritization, and on some days I'd skip the routine entirely because the briefing scared me more than it helped.

I rewrote the command with 3 main changes:

1. **Top 3, not backlog** — Claude has to choose, not list
2. **Blocks with intention** — every window of the day gets a concrete suggestion, inspired by Bullet Journal
3. **Escape hatch for paralysis** — a fixed section for when you freeze

The result became an ally, not a report.

## Install

### Option A — Ask Claude to install it (recommended)

Open Claude Code and paste this:

> Install the today command from https://github.com/Gustavosilveira23/today-command. Download the `today.md` file, save it to my `~/.claude/commands/` folder, then ask me about my task lists, notes folder, and reading sources so you can adapt it to my context.

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
| **Markdown files** (Obsidian, Notes) | Find `tasks.md` and loose checkboxes in your notes folder | Native (Glob + Grep), no MCP needed |
| **Google Calendar** | Today's events, meeting links | Native cloud MCP |
| **Google Tasks** | Pending tasks across lists | Community npm MCP |
| **Notion** | Sprint board, task database | Native cloud MCP |
| **WebFetch / WebSearch** | Curated reads from your sources | Native |

If you keep everything in markdown, you might not need any MCP. If you live in Notion, you can skip Google Tasks. Adapt. **Remove the corresponding sections from `today.md`** for any source you don't use (so Claude doesn't try to fetch from nothing).

### Google Calendar (native MCP)

1. Open `claude.ai` → Settings → Connected accounts
2. Connect Google Calendar with your Google account

Done. Exposes tools like `gcal_list_events`.

### Google Tasks (community MCP)

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

### Notion (native MCP)

1. Open `claude.ai` → Settings → Connected accounts
2. Connect Notion
3. Find the URL of your sprint board view (use `notion-search` and `notion-fetch` to locate it)
4. Paste that URL in section 3 of `today.md`

## Customize

Things you can change in `today.md`:

- **Tone**: the default is direct, light, no pressure. Adjust under "Briefing principles"
- **Number of priorities**: default is 3, bump to 5 if you prefer
- **Wikilinks**: if you don't use Obsidian, remove the `[[...]]` instructions
- **Fixed sections**: "If you get stuck" and "Wrap-up" are fixed, but you can swap them for whatever fits your routine (gratitude, intention of the day, etc.)
- **Output language**: the command instructs Claude to generate the briefing in **the same language you normally talk to Claude in**. No flag to flip — just write in your language and the briefing comes out in your language.

## Technical notes

- **Windows**: Local MCPs based on Python/uvx **don't work** in Claude Code running on VS Code Windows. Use npm packages with `claude mcp add` via terminal (that's why Google Tasks here uses the `mcp-googletasks-vrob` package)
- **OAuth in test mode**: On first Google Tasks auth, the app sits in "test mode" in Google Cloud Console. Remember to add your email as a test user, otherwise auth fails
- **Resilience**: `today.md` instructs Claude to keep going even if one source fails (e.g., if Google Tasks doesn't auth, it generates the briefing with just Calendar). No need to pray everything works
- **Idempotent**: You can run `/today` multiple times a day — it always overwrites the previous file

## Setup time

Total estimate: ~30-40min (most of it is Google Cloud Console)

| Step | Time |
|------|------|
| Create Google Cloud project + enable APIs + OAuth | ~15min |
| Connect Google Calendar (native MCP) | ~2min |
| Install and authenticate Google Tasks (npm) | ~5min |
| Connect Notion (optional) | ~3min |
| Copy and adapt `today.md` to your context | ~10min |

## Inspirations

- [Teresa Torres — Continuous Discovery](https://www.producttalk.org/) — origin of the morning briefing habit
- Carina — who introduced me to the workflow
- **Bullet Journal** (Ryder Carroll) — the time-blocks-with-intention format
- [Claude Design Skills](https://github.com/Gustavosilveira23/claude-design-skills) — another repo of mine, same skills/commands pattern

## License

MIT. Take it, adapt it, share it. If it becomes useful in your flow, drop me a line.
