# Player Identity Register

**Golden rule: identify a player by their record `id`, never by typed name or nickname alone.** Before creating, editing, or linking any player record, search for a similar existing name/nickname first — the app itself does not enforce this (Known Issues #10).

## Confirmed identities (ground truth — do not second-guess)

| Nickname / common reference | Full name | Player ID | Position | Notes |
|---|---|---|---|---|
| Jiffy | James Dickinson | `6a61af1a2757ed8f54b81e60` | FWD | Real stats attached (7 apps, 2 goals, 3 assists as of 2026-09-07). **Never link to Paul Griffiths.** |
| — | Mark Dickinson | `6a362b0234d01f9e9562092c` | MID | A completely different person from James Dickinson. Known as just "Mark" or "Mark Dickinson" — no nickname needed. |
| Griffo | Paul Griffiths (Griffo) | `6a362b0234d01f9e9562092b` | FWD, shirt #10 | Outfield player. One of two people named Paul Griffiths at this club — **never merge with Griff.** |
| Griff | Paul Griffiths (GK) | `6a4de9c6ff18cb10446b5d26` | GK | Goalkeeper. The other Paul Griffiths. Added from a WhatsApp contact "Griff Keeper." |
| Craig Smith | Craig Smith | `6a362b0234d01f9e9562091e` | DEF | **Must never be changed to or confused with Paul Smith.** No "Paul Smith" record currently exists — if one is ever created, check this isn't a conflation before assuming it's the same person. Confirmed correct spelling against the FA's official registered squad list, 2026-09-09. |
| — | Kevin Berry | `6a362b0234d01f9e95620926` | CB | **Note the near-identical ID to Griffo's** (`...20926` vs `...2092b`) — easy to mis-copy, always double-check the last character. Free-kick specialist. |
| Framie | Paul Frame | `6a362b0234d01f9e9562091a` | — | **Co-manager of the club, and also plays.** Not to be confused with Corby. |
| Corby | Anthony Richardson | `6a362b0234d01f9e9562091c` | — | **The other co-manager — does not play.** The club's matchday programme's "The Manager" feature page can refer to either co-manager depending on the week; don't assume "the manager" always means Paul Frame. |
| Buddy | Stephen Halliday | `6a38ca0a21cb430084f8f8b6` | MID | **Corrected spelling 2026-09-09** (was "Stephen Haliday " with a trailing space — missing the second "l") after confirming against the FA's registered squad list and the real match history attached to this record (2 real appearances, incl. 2nd half vs Darlington Railway Athletic, 29 Aug 2026). **Currently injured** (confirmed by Paul, 2026-09-09). |

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
| Paul Diamond | `6a362b0234d01f9e95620923` | ST | Dimo | **Spelling corrected 2026-09-09** — confirmed "Diamond" against the FA's official registered squad list (was "Dimond" in the database, matching neither the FA nor Paul's own sent email, which already correctly said "Diamond"). |
| Worz | James Shickle | `6a38ca0a21cb430084f8f8b7` | — | **Spelling corrected 2026-09-09** — was "James Shikle" (missing the "c"), confirmed against the FA's official registered squad list. |
| Stevie Anderson | `6a38ca0a21cb430084f8f8b8` | SUB | Ando |
| Daz Cliff | `6a362b0234d01f9e95620920` | SUB | Cliffy |
| Mick Greenwell | `6a38ca0a21cb430084f8f8b9` | SUB | — (also the FA-upload recipient/contact and a committee member — Player Registrations, per the club programme) |
| Rob Kelly | `6a38ca0a21cb430084f8f8bb` | SUB | Rob |

## Known duplicate/stray records (investigated, resolved, left as-is)

- **James Dickinson duplicate** (`6a9d2f18586e6e627cf46f91`) — completely empty (zero stats/contact), archived same day it was created (2026-09-06), harmless same-day mis-click. Paul approved deletion; not yet deleted — no delete tool available to Claude, needs manual removal via Squad in the app.
- **"Paul Griffith" (singular)** (`6a6e2dee326d3a4cb774fe3c`) — archived August 2026 by an admin with a note calling it "a duplicate of Griffo." **This note is now in question, not confirmed**: the FA's own official registered squad list (2026-09-09) shows "Paul Griffith" and "Paul Griffiths" as two *separately* registered names, which raises a real possibility this is a genuine third person, not a duplicate at all. *Needs Paul's confirmation* before this archived record is treated as settled either way. Still referenced by a harmless, already-voided `LmsEntry`+`LmsPick` pair regardless of the outcome (explained in `KNOWN_ISSUES.md`).
- **"Stephen Halliday" (empty duplicate)** (`6a9d2f18586e6e627cf46f93`) — **resolved 2026-09-09**: confirmed by Paul as an empty duplicate of Stephen "Buddy" Halliday (`...f8f8b6`, the real record, which had the spelling "Haliday" and now carries the corrected spelling plus his confirmed injury status). Archived (not deleted — no delete tool available).

## What "flag it" means in practice for player identity

State plainly what's ambiguous and what you'd recommend, then wait. Never merge, archive, delete, rename, or relink a player record based on a best guess — including when two names look similar, when a nickname could plausibly match more than one record, or when a name is misspelled somewhere.

## Outstanding work

A full systematic audit of all ~90+ `Player` records for near-duplicates (beyond the two cases already found) has **not** been done — only the specific cases surfaced by real workflows so far. Recommend a dedicated pass (e.g. fuzzy-match all `full_name`/`nickname` pairs) before relying on this register as exhaustive. *Needs Paul's confirmation this is wanted, given it touches every player record.*
