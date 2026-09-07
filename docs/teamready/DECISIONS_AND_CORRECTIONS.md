# Decisions and Corrections

Every correction Paul has given overrides an earlier assumption, generated answer, or unverified record. Logged here permanently, distinct from the general `CHANGELOG.md`, so a future session never re-makes the same wrong assumption.

## 2026-09-09 — Twilio/SMS was a deliberate business decision, not a bug

**Earlier assumption (mine, wrong framing):** treated the `sendWeeklyPickSms` Twilio authentication failures as a critical technical bug requiring a fix (e.g. restoring credentials).
**Correction (Paul):** Twilio was deliberately discontinued — it was running out of credit too fast against the volume of messages needed. Any function still trying to send SMS is expected to fail; the fix is to stop it trying, not to reconnect it.
**Effect:** `SMS_ENABLED = false` guard added to `sendWeeklyPickSms`. Known Issues #1 status corrected from "Critical bug" to "Fixed — was a deliberate decision." Same correction applies to the ~14 other still-Twilio-dependent functions (Known Issues #21) — a channel-replacement decision is pending, not a "restore Twilio" fix.
**Database records affected:** none — this was a code-behaviour correction, not a data correction.

## 2026-09-09 — Matchday Studio was never used for live recording

**Earlier assumption (mine):** Matchday Studio might be a legitimate alternative recording surface, just riskier for being public.
**Correction (Paul):** confirmed "No" — Live Match Centre is the only screen ever used to run a match. Matchday Studio's recording capability had zero legitimate use.
**Effect:** the entire recording control surface (Kick Off/Goal/Card/Sub/Full-Time) was removed from `MatchdayStudio.jsx`, leaving it purely read-only. Known Issues #22.
**Database records affected:** none — this was a code change, no historical data touched.

## 2026-09-07 to 09 — Player identity confirmations

**Correction/confirmation (Paul), each overriding "flag it, don't guess":**
- "Jiffy is James Dickinson" — confirmed, matches the live data exactly.
- "There are two different Paul Griffiths — one goalkeeper, one outfield (Griffo)" — confirmed, matches live data exactly (Griffo `...092b`, Griff `...5d26`).
- "Mark Dickinson is known as just Mark or Mark Dickinson" and is not the same person as Jiffy — confirmed, recorded in `PLAYER_IDENTITY_REGISTER.md`.
**Effect:** all three locked in as verified ground truth. No merges or edits were needed — the underlying data already reflected these correctly; the confirmations closed out Known Issues #4/#5 investigation.
**Database records affected:** none needed correcting — investigation only.

## 2026-09-07 — Kevin Berry's LMS buyback: manager's theory confirmed by direct verification

**Paul's stated theory:** the £10 entry that caused the "20 paid / 21 picks" mismatch was "somebody who joined but never actually completed the payment and never actually picked a team... never materialised."
**Verification (me):** queried `Payment` (none exists for that entry), `LmsEntry` (`joined_at`/`terms_accepted_at` both null), `LmsPick` (`team_picked: "VOID — invalid entry"`, `pick_submitted_at: null`, set by Paul himself on 2026-09-03). Theory confirmed exactly.
**Effect:** Known Issues #2 marked Resolved. No correction needed to the theory — this is logged as an example of the golden rule working as intended (verify, don't just trust or dismiss a stated theory either way).

## 2026-09-09 — FA Team Sheet generator must include Man of the Match

**Instruction (Paul):** "he need the mom also" — the FA email recipient (Mick Greenwell) needs MOTM included, not just lineup/goals/cards.
**Effect:** the generator reads `form.motm_player_id` (the manager's own pick from the existing Match Report page selector) first, falling back to the `MotmVote` player-vote tally with vote counts if no manager pick has been made yet — never inventing a MOTM.

## 2026-09-09 — FA registered squad list cross-check: two spelling fixes, one duplicate resolved, one Claude documentation error corrected

**Source:** Paul sent a screenshot of the FA's official registered squad list — the authoritative source for player name spelling.

- **"Paul Diamond" confirmed correct** (database had "Paul Dimond") — fixed. Known Issues #24.
- **"James Shickle" confirmed correct** (database had "James Shikle", missing a letter) — fixed. Known Issues #27.
- **Stephen Halliday duplicate resolved**: Paul confirmed "Stephen Halliday is Buddy and only played 2nd half vs Darlington RA a few weeks ago but injured atm" — matched exactly to the real record's actual match history. The empty duplicate (created 6 Sept, zero stats) was archived; the real record's spelling was corrected from "Haliday" to "Halliday" and his injury status logged. Known Issues #28.
- **Correction to Claude's own earlier documentation, not a database error:** `APP_OVERVIEW.md` had wrongly stated "Manager: Paul Frame, known as 'Corby.'" **Paul's correction:** "Framie is me Paul Frame, Corby is Anthony Richardson, we are both managers. I play also, Corby doesn't." The database was correct throughout (Paul Frame = Framie, Anthony Richardson = Corby, both are `Player` records with the right nicknames) — the error was entirely in Claude's inference from the matchday programme, now fixed. Known Issues #29.
- **"Paul Griffith" question — closed by Paul, 2026-09-09:** raised as possibly a third person given the FA list shows both spellings separately registered. **Paul's instruction: "No just leave them as they are... one goalkeeper, one outfield player, Griff Keeper, Griffo the outfield player."** Confirms the original two-Griffiths setup is correct and complete; the archived "Paul Griffith" record stays archived, untouched, no further investigation. Known Issues #5 closed.

## 2026-09-07 — Public website Phase 1: security blocker found, sponsor assets supplied, badge name and founding year confirmed

**Phase 1 recon finding (mine, not yet acted on pending Paul's go-ahead):** the existing `/public/*` and `/supporter/*` pages (already live, zero login required) fetch full `Player` records client-side with no field whitelisting; since no entity has row-level security, private fields (`login_pin`, `otp_code`, `invite_token`, `phone`, `email`, `date_of_birth`, emergency contacts) are exposed in the browser network response today, independent of any new build work. Reported to Paul in full; awaiting his go-ahead to patch and to confirm the "extend existing pages, don't duplicate" approach.

**Sponsor logos and club badge supplied by Paul:** Amber's Legacy, GKB Financial Planning, SafeSwitch Solutions, UCS Renewables (three logo variants), David Graham Roofing, DKJ Joinery, and the club badge. No image-upload tool is available to Claude — Base44's own Sponsor/Club admin screens are the only path for the actual files; Claude can create/update the surrounding data records and verify once Paul uploads.

**Badge name and founding year — confirmed by Paul, correcting a real ambiguity found across three different badge assets:** a kit mockup photo showed "Grindon Broadway Over 40s Football Club, EST 2021" — different name AND different year from the confirmed badge image ("Grindon Board Inn Over 40s, EST. 1995"). Paul's instruction: **"Defo 1995 est"** and confirmed "Grindon Board Inn" (not "Broadway") is the name for the official badge. The "EST 2021" kit mockup is wrong and must not be used publicly. `APP_OVERVIEW.md` already had 1995 correct; no change needed there. Known Issues #30.

**Sponsor categorisation — Claude's own error caught and corrected before writing bad data:** initially asked Paul to pick a sponsorship category for UCS Renewables using options that mixed up two different database fields (`sponsorship_type` enum vs. the separate `priority` enum, which has "main"/"standard" — "main" isn't a valid `sponsorship_type` value). Caught before any write; re-asked correctly. Paul confirmed: UCS Renewables = type `kit` (logo confirmed on training tops), priority `main`. DKJ Joinery = `pitch`. David Graham Roofing = `pitch`. SafeSwitch Solutions = `general`.

**David Graham Roofing — confirmed same person as the goalkeeper:** Paul confirmed ("yes to all") this sponsor is the same David Graham who plays in goal (`Davie Graham`, player id `6a5bd695e42bb914bb55bb2b`) — not linked as a database relationship, just noted here since it's a real fact about a real person, not to be assumed again.

**Database records created:** four new `Sponsor` records — UCS Renewables (`6a9f3299075e24e0b122223b`), DKJ Joinery (`6a9f3299075e24e0b122223c`), David Graham Roofing (`6a9f3299075e24e0b122223d`), SafeSwitch Solutions (`6a9f3299075e24e0b122223e`) — all with `logo_url: null`, `website_url: null` pending Paul supplying/uploading them.

## Pending — not yet confirmed by Paul

- **Programme "Results This Season" mix-up root cause** (Known Issues #25) — whether this is a recurring generation bug or a one-off manual slip.
- **Whether the full player-identity register needs a systematic duplicate audit** across all ~90+ records — the FA registered-squad cross-check (2026-09-09) covered the 37 active players and found 2 real spelling errors plus 1 more duplicate, so a similar pass may be worth doing for the remaining ~55 inactive/archived records too.

## How this file is maintained

Every time Paul corrects a name, identity, statistic, fixture, score, process, or app rule: (1) check whether it affects existing database records, (2) record it here, (3) update the relevant doc (`PLAYER_IDENTITY_REGISTER.md`, `KNOWN_ISSUES.md`, etc.), (4) add validation to prevent recurrence where practical, (5) never change historical data until it's confirmed which records are actually affected, (6) tell Paul explicitly whether any existing records need repairing.
