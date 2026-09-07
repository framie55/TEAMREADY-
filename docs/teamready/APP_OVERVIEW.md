# App Overview

## The club

**Grindon Board Inn Over 40s FC** (also called Grindon Broadway Over 40s FC). Established 1995, over 25 years running. Based at the Board Inn pub, Durham Road, East Herrington, Sunderland SR3 3NS — players and supporters gather there after every match. Plays in the **Sunderland & District Mill View Social Club Over 40s League, Division One**. **Co-managers: Paul Frame ("Framie" — also plays) and Anthony Richardson ("Corby" — does not play).** Chairman: Brian Burnikell. Club values: respect, teamwork, enjoyment, commitment, pride. Charity partner: Amber's Legacy. Main shirt sponsor: GKB Financial Planning. Other sponsors include UCS Renewables/Technologies, SafeSwitch Solutions, GM Sports Therapy.

Source: club's own matchday programme (Week 6, 2026/27 season), cross-checked against live app data. — *Confirmed.*

## What TeamReady does

Runs the club's entire weekly and matchday operation in one phone-first app:

- Player records, availability, squad selection, formations
- Live matchday scoring (goals, cards, subs, full-time) and finalised per-player stats
- Match reports, an FA-upload team sheet generator, the 24-page weekly matchday programme
- Awards: Player of the Match (manager pick + player vote + public fan vote), Player of the Month
- Training, club finances/subs, sponsors
- "Last Man Standing" — a separate side competition with real Stripe entry fees
- Reminders (real: SMS via Twilio [currently being phased out due to cost] and push via OneSignal; "WhatsApp" features generate text for manual sending only, never automated)

## Who uses it

- **Staff** (admin/manager/assistant manager/chairman/treasurer): Base44-managed email+password or Google login.
- **Players**: no password — invite link, SMS one-time code, name+PIN (the club's actual preferred method — players struggled with links), or emailed magic link.
- **Guests/supporters/public**: read-only public pages, plus a token-gated Last Man Standing entry flow for non-players.

## Tech stack

- Frontend: React 18 + Vite 6, React Router v6, Tailwind + shadcn/ui, TanStack React Query, React Hook Form + Zod.
- Backend: ~90 serverless functions, TypeScript on Deno, one per folder under `base44/functions/`.
- Data: ~95 entities as `.jsonc` schemas under `base44/entities/`, stored in Base44's managed database — no traditional SQL/NoSQL connection string, accessed only via the Base44 SDK/MCP tools.
- Hosting: entirely inside Base44 (App ID `6a359ec6743bebc341bd156f`) — not GitHub-deployed. Base44's own S3-backed git remote holds the app's code history (788+ commits as of 2026-09-07).

## What TeamReady is explicitly not

- Not a general club-website CMS — the public marketing pages exist but are secondary to the operational core.
- Not connected to the FA's Full-Time website — that site is behind Cloudflare bot protection and cannot be scraped (confirmed by direct test, 2026-09-07); league table and opposition scouting data have to be manually screenshotted and transcribed in.
- Not currently enforcing any data-access rules at the entity level — see `CLUB_RULES.md` and `KNOWN_ISSUES.md` #3.

## Where this information came from

Phase 1 full-codebase inspection (2026-09-07, five parallel research passes covering tech stack/security, data model, and all major workflows), direct verification against live data throughout the sessions since, and the club's own Week 5/6 matchday programme PDF (2026-09-07). Anything below marked *Needs Paul's confirmation* has not been independently verified.
