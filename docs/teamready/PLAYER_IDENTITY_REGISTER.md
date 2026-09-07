# Player Identity Register

**Golden rule: identify a player by their record `id`, never by typed name or nickname alone.** Before creating, editing, or linking any player record, search for a similar existing name/nickname first — the app itself does not enforce this (Known Issues #10).

## Confirmed identities (ground truth — do not second-guess)

| Nickname / common reference | Full name | Player ID | Position | Notes |
|---|---|---|---|---|
| Jiffy | James Dickinson | `6a61af1a2757ed8f54b81e60` | FWD | Real stats attached (7 apps, 2 goals, 3 assists as of 2026-09-07). **Never link to Paul Griffiths.** |
| — | Mark Dickinson | `6a362b0234d01f9e9562092c` | MID | A completely different person from James Dickinson. Known as just "Mark" or "Mark Dickinson" — no nickname needed. |
| Griffo | Paul Griffiths (Griffo) | `6a362b0234d01f9e9562092b` | FWD, shirt #10 | Outfield player. One of two people named Paul Griffiths at this club — **never merge with Griff.** |
| Griff | Paul Griffiths (GK) | `6a4de9c6ff18cb10446b5d26` | GK | Goalkeeper. The other Paul Griffiths. Added from a WhatsApp contact "Griff Keeper." |
| Craig Smith | Craig Smith | *(id not yet recorded here)* | DEF | **Must never be changed to or confused with Paul Smith.** No "Paul Smith" record currently exists — if one is ever created, check this isn't a conflation before assuming it's the same person. |
| — | Kevin Berry | `6a362b0234d01f9e95620926` | CB | **Note the near-identical ID to Griffo's** (`...20926` vs `...2092b`) — easy to mis-copy, always double-check the last character. Free-kick specialist. |

## Other players resolved during real match/report work (2026-09-05 fixture, 2026-09-07 verification)

| Name (as used in-app) | Player ID | Position | Nickname |
|---|---|---|---|
| David Graham | `6a5bd695e42bb914bb55bb2b` | GK | Davie Graham |
| David Sewell | `6a6ffe24fe2123781ae7f85c` | RB | Sewell |
| Garry Wake | `6a362b0234d01f9e95620927` | CB | — |
| Gareth Brazier | `6a3d9d2ba00bf08738369af3` | LB | Gash |
| Paul Mooney | `6a3d9abb01fca3df025bf8ed` | CM | — |
| Billy Harrison | `6a38ca0a21cb430084f8f8b5` | CM | — (captain, confirmed via `is_captain` on team sheet) |
| Gary Barnfather | `6a3c40a378d16ff23000c4f0` | ST | — |
| Paul Dimond / "Paul Diamond" | `6a362b0234d01f9e95620923` | ST | Dimo | **Spelling conflict, unresolved**: the database has "Paul Dimond," the manager's own sent FA email says "Paul Diamond." *Needs Paul's confirmation* which spelling is correct — this could cause an FA registration mismatch. |
| Stevie Anderson | `6a38ca0a21cb430084f8f8b8` | SUB | Ando |
| Daz Cliff | `6a362b0234d01f9e95620920` | SUB | Cliffy |
| Mick Greenwell | `6a38ca0a21cb430084f8f8b9` | SUB | — (also the FA-upload recipient/contact and a committee member — Player Registrations, per the club programme) |
| Rob Kelly | `6a38ca0a21cb430084f8f8bb` | SUB | Rob |

## Known duplicate/stray records (investigated, resolved, left as-is)

- **James Dickinson duplicate** (`6a9d2f18586e6e627cf46f91`) — completely empty (zero stats/contact), archived same day it was created (2026-09-06), harmless same-day mis-click. Paul approved deletion; not yet deleted — no delete tool available to Claude, needs manual removal via Squad in the app.
- **"Paul Griffith" (singular)** (`6a6e2dee326d3a4cb774fe3c`) — archived August 2026 by an admin who correctly identified it as a duplicate of Griffo, with a note saying so. Still referenced by a harmless, already-voided `LmsEntry`+`LmsPick` pair (a phantom LMS signup that was never paid or picked — explained fully in `docs/teamready/KNOWN_ISSUES.md` and the old `TEAMREADY_CHANGELOG.md`). Left alone deliberately — deleting the player now would orphan those two records for no benefit.

## What "flag it" means in practice for player identity

State plainly what's ambiguous and what you'd recommend, then wait. Never merge, archive, delete, rename, or relink a player record based on a best guess — including when two names look similar, when a nickname could plausibly match more than one record, or when a name is misspelled somewhere.

## Outstanding work

A full systematic audit of all ~90+ `Player` records for near-duplicates (beyond the two cases already found) has **not** been done — only the specific cases surfaced by real workflows so far. Recommend a dedicated pass (e.g. fuzzy-match all `full_name`/`nickname` pairs) before relying on this register as exhaustive. *Needs Paul's confirmation this is wanted, given it touches every player record.*
