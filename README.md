# Enterprise Vibe Coding · EightKB 2026

Slides for Chrissy LeMaire's EightKB 2026 session on AI-assisted development for DBAs: what changed when coding agents moved from the browser onto your machine, what they can build, and how sandboxes, permission modes, hooks, skills, and MCP keep them inside the lines.

The deck is a single static HTML page. There is no build step and no framework.

## Files

| Path | What it is |
|------|------------|
| `index.html` | The deck. One `<section class="slide">` per slide. |
| `index.md` | The same content as plain Markdown, for reading, searching, or reuse. |
| `deck.css` | Slide layouts, typography, reveal animation. |
| `deck.js` | Navigation, keyboard handling, scale-to-fit, reveal choreography. |
| `images/` | Screenshots and book covers referenced by the slides. |
| `session-proposal.md` | The abstract as submitted, with sources for every statistic. |
| `scripts/check-deck.ps1` | Structural and copy checks for `index.html`. |
| `scripts/test-*.ps1` | Content tests that pin the copy of specific slides. |
| `.claude/` | The hooks, skills, and MCP servers the talk demonstrates, running in this repo. |

## Presenting

Open `index.html` in a browser. Fonts load from Google Fonts, so the first load wants a network connection; everything else is local.

| Key | Action |
|-----|--------|
| `→` `Space` `PageDown` `L` | Next slide |
| `←` `PageUp` `H` | Previous slide |
| `Home` / `End` | First / last slide |
| `F` | Toggle fullscreen |
| Click | Left 18% of the window goes back, anywhere else goes forward |
| Click a screenshot | Opens it in a lightbox; `Esc` or any navigation key closes it |

The current slide is kept in the URL hash (`#slide-12`), so a reload or a shared link lands on the same slide.

## Editing

Each slide is a `<section class="slide">` with a `data-section` label and one or more `data-reveal` elements. Reveals stagger in document order; add `data-reveal="with-prev"` to land an element together with the one before it.

After editing, run the checks:

```powershell
pwsh -NoProfile scripts/check-deck.ps1
Get-ChildItem scripts/test-*.ps1 | ForEach-Object { pwsh -NoProfile -File $_ }
```

`check-deck.ps1` confirms every local `src` and `href` resolves to a file, slide copy contains no em dashes, en dashes, or double hyphens, every slide has a `data-section` and a `data-reveal`, and element ids are unique. It ends with `OK  <n> slides, no problems` or one `FAIL` line per finding.

If you edit the copy of a slide that a `test-*.ps1` file pins, update the test in the same change.

## The agent setup in this repo

The talk argues that instruction files are requests and hooks are enforcement. This repo runs that setup on itself through `.claude/settings.json`:

- **PreToolUse** hooks in `.claude/hooks/sql/` block `TrustServerCertificate`, string-interpolated SQL, `NVARCHAR(MAX)`, non-sequential GUID keys, missing audit columns, and `DROP`/`TRUNCATE`. The ones in `.claude/hooks/general/` require a read before an edit, gate code changes on a failing test, and refuse to read anything that looks like a credential.
- **PostToolUse** hooks warn when a file grows past a line-count threshold and run the humanizer skill on any Markdown the agent writes.
- **Stop** hooks run the test scripts and send the session's diff to a second vendor's model for read-only review before the turn is allowed to end.
- **Skills** in `.claude/skills/` (`build`, `doublecheck`, `fix-ci`, `humanizer`, `impeccable`) are the ones the deck lists.
- **MCP servers** in `.mcp.json` are all read-only documentation sources: Microsoft Learn, dbatools, GitHub, and Cloudflare Docs.

Hook scripts run under Git Bash and are pinned to LF line endings by `.gitattributes`.

## Style rules

`CLAUDE.md` carries the copy rules for slide text. The short version: no dashes used as punctuation, no two-beat slogans, and a list of buzzwords that stay out of the deck.
