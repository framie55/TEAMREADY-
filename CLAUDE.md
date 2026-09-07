# TeamReady — Working Notes for Claude

Read this first, every session. It orients you fast; the detail lives in `docs/teamready/`.

## What TeamReady is

A phone-first web app that runs the weekly and matchday operations of **Grindon Board Inn Over 40s FC** (also known as Grindon Broadway Over 40s FC), a grassroots veterans football club in Sunderland, playing in the Sunderland & District Mill View Social Club Over 40s League, Division One. Founded 1995. Manager: Paul Frame ("Corby").

It covers: players, availability, fixtures, squad selection, formations, live matchday scoring, stats, match reports, awards (MOTM/POTM), training, a "Last Man Standing" side competition, club finances/subs, sponsors, the weekly matchday programme, and reminders.

**The app is not hosted on GitHub.** It's built and run entirely inside **Base44** (a low-code AI app builder). App ID `6a359ec6743bebc341bd156f`. This git repo (`TEAMREADY-`) mirrors the app's documentation for continuity between sessions and tools, but the actual application code, data, and deploys live in the Base44 sandbox — reached via the `mcp__Base44__*` tools (list_directory, read_file, write_file, edit_file, run_command, query_entities, create_entities, update_entities, list_entity_schemas, create_checkpoint), not via local git operations.

## How it's structured

- Frontend: React 18 + Vite 6, Tailwind + shadcn/ui, TanStack Query. ~200 pages under `src/pages/`.
- Backend: ~90 serverless functions under `base44/functions/`, one per folder (`entry.ts`).
- Data: ~95 entities under `base44/entities/*.jsonc`.
- Full detail: `docs/teamready/APP_OVERVIEW.md`, `FEATURE_MAP.md`, `DATABASE_MAP.md`.

## Run, test, build (inside the Base44 sandbox, via `mcp__Base44__run_command`, cwd `/app`)

```
npm install       # install deps
npm run dev       # local dev server (Base44 sandbox also runs this automatically for live preview)
npm run build     # production build — ALWAYS run this after any code change, before checkpointing
npm run lint      # ESLint — expect ~314 pre-existing "unused import" errors, harmless
npm run typecheck # tsc — expect ~648 pre-existing errors, mostly in public/supporter/player-portal pages
```
**There is no test suite.** `npm test` does not exist. See `docs/teamready/KNOWN_ISSUES.md` #6.

## Deployment

Base44 builds and serves the app itself (`base44/config.jsonc`). No GitHub Actions/CI. Version control is Base44's own S3-backed git remote — 788+ commits, effectively all from Paul (`paulframe1979@gmail.com` / `paul@safeswitch.co.uk`), mostly generic "File changes"/"External agent changes" messages. **Always run `mcp__Base44__create_checkpoint` immediately after a verified code change** — this is the rollback point. See `docs/teamready/RECOVERY_GUIDE.md`.

## Database and API

Query/write live data with `mcp__Base44__query_entities` / `create_entities` / `update_entities` (no delete tool exists — see Known Issues #4/#5 for the workaround). Full entity map: `docs/teamready/DATABASE_MAP.md`.

## The golden rule (never break this)

**Never guess player identities, statistics, fixtures, scores, availability, finances, or match events.** Verify against real records before writing anything. If ambiguous, flag it and wait — do not resolve it yourself. Full rules: `docs/teamready/CLUB_RULES.md`. Confirmed player identities (read before touching any player record): `docs/teamready/PLAYER_IDENTITY_REGISTER.md`.

Quick reference, the ones that bite:
- **Jiffy = James Dickinson** (`6a61af1a2757ed8f54b81e60`). Not Mark Dickinson (`...092c`), a different person.
- **Two Paul Griffiths, never merge them**: Griffo = outfield (`...092b`), Griff = goalkeeper (`6a4de9c6ff18cb10446b5d26`).
- Craig Smith ≠ Paul Smith.
- A nickname must never be used to auto-create or auto-match a player record.

## Safety rules

- Never delete production data to fix a bug. Never merge/rename a player without Paul's explicit confirmation of which record is correct.
- Never let a match go "official" (stats, reports, payments) without a human having looked at it — AI-generated content is a draft until confirmed.
- Never send a message, publish programme content, or take a payment action without showing Paul what/who/when first.
- Before any code change: checkpoint first, smallest possible diff, verify build, then checkpoint again and log it in `docs/teamready/CHANGELOG.md`.
- No delete tool exists for database records — see Known Issues before attempting a workaround.

## Daily / weekly operations

`docs/teamready/DAILY_OPERATIONS.md` and `WEEKLY_OPERATIONS.md`. Run the daily check only when Paul asks for it ("run the TeamReady daily check").

## Known problems and current priorities

`docs/teamready/KNOWN_ISSUES.md` (severity-ranked, living list) and `docs/teamready/AUTOMATION_REGISTER.md`.

## Documentation index

| File | Contents |
|---|---|
| `docs/teamready/APP_OVERVIEW.md` | What the app is, who uses it, tech stack, club background |
| `docs/teamready/FEATURE_MAP.md` | Every major feature, its pages, its data, its dependencies |
| `docs/teamready/DATABASE_MAP.md` | Entity-by-entity data dictionary and relationships |
| `docs/teamready/PLAYER_IDENTITY_REGISTER.md` | Every confirmed player identity, nickname, and id |
| `docs/teamready/CLUB_RULES.md` | Data rules, terminology, golden rule detail |
| `docs/teamready/DAILY_OPERATIONS.md` | The daily check routine |
| `docs/teamready/WEEKLY_OPERATIONS.md` | The full weekly matchday cycle |
| `docs/teamready/PROGRAMME_WORKFLOW.md` | How the matchday programme is built, section by section |
| `docs/teamready/AUTOMATION_REGISTER.md` | Every backend function/automation: trigger, reads, writes, failure mode |
| `docs/teamready/KNOWN_ISSUES.md` | Severity-ranked list of everything found wrong, fixed, or open |
| `docs/teamready/DECISIONS_AND_CORRECTIONS.md` | Every correction Paul has given, and what it overrode |
| `docs/teamready/CHANGELOG.md` | Every change made to the app, chronological |
| `docs/teamready/RECOVERY_GUIDE.md` | Backups, checkpoints, rollback, what would be lost |

**Superseded:** the six `docs/TEAMREADY_*.md` files (system map, daily checklist, weekly workflow, data rules, known issues, changelog) predate this structure and are kept for history — their content has been carried forward into the files above. Don't maintain both going forward.
