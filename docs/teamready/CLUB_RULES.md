# Club Rules, Data Rules, and Terminology

## The golden rule

**Never guess player identities, statistics, fixtures, scores, availability, finances, or match events.** If it's unclear which verified player something belongs to, whether a record is a duplicate, or what a correction actually implies, flag it for Paul and wait. Do not resolve it yourself, no matter how confident it seems.

A correction from Paul overrides any earlier assumption, generated answer, or unverified record — see `DECISIONS_AND_CORRECTIONS.md`.

## Structural data rules

1. **Identify a player by record ID, never by typed name or nickname alone.** Several entities store a free-text name alongside the ID (`PlayerMatchStat`, `MatchReport` arrays, `HallOfFame`, `AwardVote`, `LmsPick`/`LmsEntry`) — if they disagree, trust the ID.
2. **Search for a similar existing player before creating a new one.** The app doesn't enforce this — it's a manual step until a real duplicate-check exists (Known Issues #10).
3. **`HallOfFame` and `AwardVote` can exist with only a typed name and no linked player ID.** Don't assume it's "obviously" a specific player — confirm first.
4. **`Player.season_*` fields are a cache, not the source of truth.** `PlayerMatchStat` (rebuilt from `MatchTimelineEvent` by `finaliseMatch`) is authoritative. If they disagree, re-run `recalculatePlayerStats` — never hand-edit the `Player` fields.
5. **`Fixture.result_home`/`result_away` and `MatchReport.home_score`/`away_score` can disagree** — they're two separate fields with no sync guarantee. Flag a mismatch, don't silently pick one.
6. **Club finances live in three separate, unreconciled ledgers**: `Payment`, the LMS rewards levy, and `FundraisingDonation`. Never assume a number from one reconciles against another without checking.
7. **Stripe's webhook is the only trustworthy confirmation a payment succeeded.** A `cash_received` manual mark (Monzo transfer, physical cash) is a legitimate separate path, but isn't the same level of proof — always note in the `notes` field exactly how a manually-confirmed payment was actually collected.
8. **AI-generated content (match analysis, drafted reports) is a draft, not a fact**, even when technically saved on a record. Label it as AI-generated and get human review before it's relied on for stats or history.
9. **No entity in the app declares row-level security.** Every one of the ~95 data types — including player phone numbers, DOB, login PINs, and every payment — currently has no data-layer access control. This is an open, unresolved decision (Known Issues #3), not yet acted on.
10. **No delete tool exists for database records via Claude's current tools** — only create/update/query. A genuine deletion needs the app's own UI or a deliberate, explained one-off function — never an improvised workaround for something low-stakes.

## Never (absolute)

- Never delete production data to solve a bug.
- Never silently merge, rename, or relink a player record.
- Never change confirmed statistics without evidence.
- Never send a message, publish content, or take a payment action without showing Paul what/who/when first and getting confirmation.
- Never deploy an untested change, or skip the build check before checkpointing.
- Never use a destructive git/database operation.
- Never expose secrets or credentials in code, docs, or commits. If access to a service is needed, name which service — never ask Paul to paste a password or API key into chat.

## Sponsors vs. charity partners — not the same thing

- **Commercial sponsors** (`Sponsor.sponsorship_type` = `kit`, `pitch`, `general`, `main_club`, etc.) give the club money in exchange for exposure. `priority: 'main'` on these marks the sponsor giving the most, and drives the site-wide "Main Club Sponsor" prominence.
- **Amber's Legacy** (`sponsorship_type: 'charity_partner'`) is different in kind, not just degree: it's a cervical cancer charity set up in memory of the daughter of Daz Cliff (Darren Cliff — "Cliffy" in the nickname table above), a player at the club. The club promotes it to support Daz and the cause, not for any commercial or promotional benefit. It carries `priority: 'main'` in the database for historical reasons, but must never be ranked, sized, or reported alongside paid sponsors as if it were one — it has its own dedicated placement (the About page) and stays out of any "who's paying the most" sponsor hierarchy.

## Terminology

- **O40s** = Over 40s (the age category this entire league/club is built around).
- **DSRM** = Darlington & Simpson Rolling Mills Social Club (opponent club name, easily confused with "Darlington Railway Athletic" — a genuine mix-up already found once in the matchday programme, see Known Issues).
- **LMS** = Last Man Standing (the side competition), not to be confused with any technical meaning of the acronym elsewhere.
- **MOTM** = Man of the Match (per-match award). **POTM** = Player of the Month.
- **"Finalise Match"** = the step that converts live timeline events into official per-player stats and the match report. Distinct from "Confirm Full Time" — as of 2026-09-09 the two happen together automatically (Known Issues #23).
- **"The programme"** = the weekly matchday PDF/graphic pack (~24 pages), not a piece of code.
- **Base44** = the low-code platform TeamReady is built on and hosted in. Not GitHub, not a traditional server.
- **Checkpoint** = Base44's save-point mechanism (`mcp__Base44__create_checkpoint`) — the rollback unit for code changes, roughly equivalent to a git commit + tag.

## Nicknames in day-to-day use (club slang → likely player)

This is informal shorthand seen in chat/notes — always resolve to a verified `Player` ID before writing anything, never treat the table below as sufficient on its own:

| Slang | Likely player | Confidence |
|---|---|---|
| Jiffy | James Dickinson | Confirmed |
| Griffo | Paul Griffiths (outfield) | Confirmed |
| Griff | Paul Griffiths (goalkeeper) | Confirmed |
| Corby | Paul Frame (the manager) | Confirmed, from programme content |
| Dimo | Paul Dimond/Diamond (spelling unresolved) | Needs Paul's confirmation on spelling |
| Ando | Stevie Anderson | Confirmed |
| Cliffy | Daz Cliff | Confirmed |
| Gash | Gareth Brazier | Confirmed |
| Sewell | David Sewell | Confirmed |
