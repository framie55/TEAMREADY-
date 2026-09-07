# TeamReady Data Rules

_These rules are non-negotiable. Read this before touching any player, statistic, fixture, availability, finance, or match-event record._

## Golden rule

**Never guess player identities, statistics, fixtures, scores, availability, finances, or match events.** If it's unclear which verified player something belongs to, or whether a record is a duplicate, flag it for the club manager. Do not resolve it yourself.

## Known verified identities (as told by the club manager — treat as ground truth, do not second-guess)

- **"Jiffy" = James Dickinson** (verified player id `6a61af1a2757ed8f54b81e60`). **Do not confuse with Mark Dickinson** (verified player id `6a362b0234d01f9e9562092c`) — a totally different person, known as just "Mark" or "Mark Dickinson," no nickname needed since the full name alone is unambiguous. Two separate Dickinsons at this club; both real, both active.
- **There are two totally different people both called Paul Griffiths — a goalkeeper and an outfield player.** The club calls the outfield player **"Griffo"** specifically to keep them apart. Confirmed mapping, verified 2026-09-07: `Paul Griffiths (Griffo)` = outfield/forward, id `6a362b0234d01f9e9562092b`, nickname `Griffo`. `Paul Griffiths (GK)` = goalkeeper, id `6a4de9c6ff18cb10446b5d26`, nickname `Griff`. These must remain two separate verified records, never merged.
- **Craig Smith must never be changed to or confused with Paul Smith.**

## What Phase 1 inspection found in the live data (reported, not fixed)

- `Player` has **no uniqueness constraint** on name or nickname, and **no verification/status flag** beyond `active`/`archived`. Disambiguation between the two Paul Griffiths is currently done by hand — appending "(GK)" / "(Griffo)" into the `full_name` text field — not by any structured field.
- A **duplicate record for James Dickinson** exists: one archived, one active (the active one correctly carries the "Jiffy" nickname).
- A **third, unexplained "Paul Griffith" record** (note: singular, likely a typo) exists alongside the two legitimate Paul Griffiths, marked archived.
- No record for "Paul Smith" currently exists, so no conflation with Craig Smith has happened — but nothing in the system would stop it happening in future without a manual check.

**None of the above has been changed.** Any fix requires the manager to confirm which record is correct before anything is merged, archived, or edited.

## Structural rules to follow when working with this data

1. **Always identify a player by their record ID, never by typed name or nickname alone**, when writing code or data that links to a player. Several entities (`PlayerMatchStat`, the arrays inside `MatchReport`, `HallOfFame`, `AwardVote`) store a free-text name *alongside* the ID — if you ever have to choose, trust the ID field, not the name.
2. **Before creating a new player record, search existing records for a similar name or nickname first.** The app itself does not currently enforce this — treat it as a manual step until a proper duplicate-check is added (see Known Issues).
3. **`HallOfFame` and `AwardVote` records can exist with only a typed name and no linked player ID** — if you see one of these without a player ID, don't assume it's linked to the "obvious" matching player; confirm first.
4. **Season stats shown on `Player` (`season_goals`, `season_assists`, etc.) are a cache, not the source of truth.** The real source of truth is `PlayerMatchStat`, rebuilt from `MatchTimelineEvent` by the `finaliseMatch` function. If the two disagree, trust `PlayerMatchStat` and re-run `recalculatePlayerStats`, don't hand-edit the `Player` fields.
5. **`Fixture.result_home`/`result_away` and `MatchReport.home_score`/`away_score` are two separate fields that can disagree.** If they do, flag it — don't silently pick one.
6. **Financial data lives in three separate places** (`Payment`, the rewards levy system, `FundraisingDonation`) — never assume a number from one matches or reconciles against another without checking.
7. **Stripe's webhook (`stripeWebhook` function, writing `Payment.status`) is the only trustworthy confirmation that a payment succeeded.** A "cash received" manual mark is a legitimate separate path for cash, but is not the same level of proof.
8. **AI-generated content (match analysis, AI draft reports) is a draft, not a fact**, even where it's technically saved on a record. It should always be visibly labelled as AI-generated and reviewed by a human before being relied on for stats or history.

## What "flag it" means in practice

State plainly what you found, why it's ambiguous, and what you'd recommend — then wait. Do not merge, archive, delete, or overwrite a player/stat/finance record based on your own best guess, no matter how confident it seems.
