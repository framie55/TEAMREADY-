# TeamReady Changelog

_Every change made is recorded here — what changed, why, and how it was tested. This is now the canonical changelog, carried forward in full from `docs/TEAMREADY_CHANGELOG.md` (superseded, kept for history)._

**Canonical, always-current copies of these documents live inside the TeamReady Base44 app itself (`docs/teamready/`, app ID `6a359ec6743bebc341bd156f`), tied to that app's own checkpoints/commits. This repo mirrors them.**

## 2026-09-07 — Phase 1 & Phase 2: Initial inspection and operations documentation

Completed a full read-only inspection of TeamReady (tech stack, ~95 data entities, ~90 backend functions, all major workflows, git history, tests, security). No application code, data, or configuration was changed. Created the original six standing operations documents (now superseded by this `docs/teamready/` structure, 2026-09-09).

**Outstanding items raised at the time:** the Twilio SMS failure, the LMS integrity alert, the missing data-access rules, the duplicate James Dickinson record and stray "Paul Griffith" record.

## 2026-09-09 — Player identity confirmations; disabled dead Twilio SMS in the LMS pick reminder

Confirmed with Paul: Mark Dickinson (`6a362b0234d01f9e9562092c`) is separate from Jiffy; the two Paul Griffiths (Griffo `...092b`, Griff `...5d26`) are correctly distinct. Investigated the two flagged duplicate/stray player records — both empty, already archived, downgraded from Critical to Low.

**Code change:** `base44/functions/sendWeeklyPickSms/entry.ts` — added `SMS_ENABLED = false` guard (Twilio deliberately discontinued by the club, not a bug). Function now logs one clear summary instead of one failure per recipient. Scope intentionally limited to this one function; ~14 others still Twilio-dependent (Known Issues #21).
**Tested:** `npm run build` clean. No test suite exists.
**Checkpoints:** `6a9e8a34cd6c82d0ee2d54f4` (before) → `6a9eba00704d546ece83d190` / commit `ca87ccc58d19d1268086119ad0665438e2b9abe6` (after).

## 2026-09-09 (continued) — Explained the LMS integrity alert; confirmed a delete-tooling limitation

The "20 paid entries, 21 picks submitted" alert fully explained: a phantom `LmsEntry` (`6a988c49fe9ec8f718e9c0c3`) against the archived duplicate "Paul Griffith" record — no real payment, no real pick, already voided by Paul on 2026-09-03. Marked Resolved. Confirmed Claude's toolset has no delete capability for database records (create/update/query only) — manual deletion via the app's own UI recommended instead of an improvised workaround. No data was changed in this entry.

## 2026-09-09 (continued) — Matchday UX: removed public recording risk in Matchday Studio; auto-finalise on Full Time

**1. Removed the entire live-recording control surface from Matchday Studio** (`src/pages/MatchdayStudio.jsx`) — it was a full duplicate live-match engine on a public, unauthenticated route. Confirmed unused for recording; removed all recording controls/handlers/modals, left the read-only broadcast view untouched.
**2. Folded "Finalise Match" into "Confirm Full Time"** (`src/pages/LiveMatchCentre.jsx`) — `handleFullTime` now calls `handleFinaliseMatch()` automatically; manual re-finalise button kept for later corrections.
**Tested:** `npm run build` clean, targeted `eslint` clean (one newly-unused import fixed).
**Checkpoints:** `6a9ebe223850f24a7163c776` (before) → `6a9f08fe076753356d30198d` / commit `580b4000ac5bf1f72b408b2a18831121ef7e3a91` (after).
**Still open:** duplicate-tap protection, card-fine toast, per-event edit/delete, player-portal structural issues (two home screens, two nav menus, two backend aggregators — login ruled out as the pain point since the club uses Name+PIN).

## 2026-09-07 (LMS) — Recorded Kevin Berry's buyback payment and credited the pot

Kevin Berry paid his £10 LMS buyback via Monzo, outside Stripe. His `LmsEntry`/`LmsPick` (Aston Villa, GW4) were already correctly entered by Paul. Created `Payment` `6a9f0a4cc4ac78dc5c4bb7d7` (type `lms_buyback`, `cash_received`, notes explicit about Monzo not Stripe), linked via `LmsEntry.entry_payment_id`, credited pots using the same 80/15/5 split Stripe would apply: prize pot +£8.00 (£376→£384), club pot +£1.50 (£68→£69.50), players' pool +£0.50 (£23.50→£24). No code changed — data entry only, because the payment happened outside the app's own flow.

## 2026-09-09 — Built the FA Upload Team Sheet generator

Added a "Generate FA Team Sheet" button to `MatchReport.jsx` (`generateFaReport`/`copyFaReport` functions, new UI panel). Pulls live from `Fixture`, `TeamSheet`/`TeamSheetPosition`, `MatchTimelineEvent`, `MotmVote`, `Player` — builds the exact format Paul already emails to Micky Greenwell for FA upload (competition, date, venue, result, starting XI, subs, goalscorers, cards, substitutions with times, MOTM). MOTM logic: uses the manager's own pick (`form.motm_player_id`, from the existing selector on the same page) first, falls back to the player-vote tally with counts if no pick made yet — never invents one.

**Verification:** built the same report by hand first, from the real 2026-09-05 Darlington DSRM fixture, and cross-checked every line (starting XI, all 5 subs with times/reasons, the 47th-minute free-kick goal, no cards) against Paul's own already-sent email — matched exactly except one name-spelling conflict found and logged (Known Issues #24).
**Tested:** `npm run build` clean before and after; targeted `eslint` on the changed file only (one pre-existing unrelated unused-import error left alone).
**Checkpoint:** `6a9f171a25bd400824ae03d3`.

## 2026-09-07 — League table updated from screenshot

Paul confirmed the FA's Full-Time site is Cloudflare-blocked (verified directly — `fulltime.thefa.com` returns a 403 "Attention Required" challenge page even from the Base44 sandbox's own network, not just Claude's session proxy). Transcribed Paul's league-table screenshot into `LeagueTable.rows[]` (13 teams, repositioned per that week's results) — kept existing clean team names over the screenshot's truncated/miscapitalised versions, computed GF/GA precisely only for Grindon's own row (verified against the real Darlington match: 15 for, 8 against), left every other team's GF/GA `null` since the FA site only shows goal difference, never a split.

## 2026-09-07 — Reviewed the matchday programme; found a data mix-up

Reviewed the club's real 24-page Week 6 matchday programme PDF. Cross-checked the "Player by Player" section against real match data — matched exactly. Found a genuine error in the "Results This Season" panel: the 29 August result was mislabelled as a loss to "Darlington DSRM" (1-2) when the real 29 August result was a 3-1 win over "Darlington Railway Athletic" — the 1-2 loss to Darlington DSRM actually happened 5 September, a different fixture, which didn't appear in that list at all. Logged as Known Issues #25 — root cause not yet traced. No code or data changed; documentation and flagging only.

## 2026-09-09 — Set up the weekly admin check-in Routine

Created a scheduled Routine ("TeamReady Weekly Admin Check-in," trigger id `trig_01FBZHf6in1d7rkgX1yA3NKm`), Sundays 08:00 UTC (09:00 UK), firing into this persistent Claude Code session. Asks Paul for the previous match's result, Finalise Match confirmation, league-table screenshot, and opposition-scouting screenshots if needed; on reply, generates the FA team sheet and updates the league table. Flagged one caveat to Paul: a fresh-fired session might not automatically carry the same TeamReady app connection — to be confirmed on the first real firing.

## 2026-09-09 — Established permanent project memory structure

At Paul's explicit request, built a permanent documentation structure so future Claude Code sessions can pick up TeamReady work without re-deriving context:
- `CLAUDE.md` at the project root (read automatically at the start of every session) — concise orientation, links out to detail.
- `docs/teamready/` — thirteen files: `APP_OVERVIEW.md`, `FEATURE_MAP.md`, `DATABASE_MAP.md`, `PLAYER_IDENTITY_REGISTER.md`, `CLUB_RULES.md`, `DAILY_OPERATIONS.md`, `WEEKLY_OPERATIONS.md`, `PROGRAMME_WORKFLOW.md`, `AUTOMATION_REGISTER.md`, `KNOWN_ISSUES.md`, `DECISIONS_AND_CORRECTIONS.md`, `CHANGELOG.md` (this file), `RECOVERY_GUIDE.md`.

Populated entirely from information already verified this session (the five original research passes, direct database queries, and every fix/decision made since) — nothing invented, nothing guessed. Superseded the original six `docs/TEAMREADY_*.md` files (kept for history, not deleted, marked as superseded in `CLAUDE.md`). Several sections of Paul's original spec (a draft/confirmed/published status system on records, a single match-completion screen, an editable admin-settings comms schedule, a programme content-pack export, a mobile admin dashboard, an automated test suite, a full 90+-player duplicate audit) are **documented as design goals / open gaps, not built** — this was a documentation-only stage, no production code or data was changed, per Paul's explicit instruction.

## 2026-09-09 — FA registered squad list cross-check: real identity fixes and one Claude documentation correction

Paul sent a screenshot of the FA's official registered squad list. Cross-checked against all 37 active `Player` records:

- **Fixed:** "Paul Dimond" → "Paul Diamond" (confirmed against FA list — Known Issues #24).
- **Fixed:** "James Shikle" → "James Shickle" (confirmed against FA list — Known Issues #27).
- **Fixed:** found a second real duplicate-player case — an empty "Stephen Halliday" (created 6 Sept, zero stats, same pattern as the earlier James Dickinson duplicate) alongside the real record misspelled "Stephen Haliday " (2 real match appearances). Paul confirmed the real one is "Buddy," played 2nd half vs Darlington Railway Athletic a few weeks ago, currently injured. Archived the empty duplicate (with a note), corrected the real record's spelling, logged his injury status. Known Issues #28.
- **Corrected (Claude's own documentation, not a database error):** `APP_OVERVIEW.md` had wrongly stated the club has one manager (Paul Frame, "Corby"). Paul corrected: the club has **two co-managers** — Paul Frame ("Framie," also plays) and Anthony Richardson ("Corby," does not play). The database was right all along; only Claude's inference from the matchday programme was wrong. Fixed. Known Issues #29.
- **Reopened, needs Paul's confirmation:** the archived "Paul Griffith" (singular) record was previously assumed a duplicate of Griffo, but the FA's own list shows "Paul Griffith" and "Paul Griffiths" as two *separately* registered names — this may be a real third person, not a duplicate. Known Issues #5 reopened, nothing changed on this record pending confirmation.

**Database records affected:** `Player` `6a362b0234d01f9e95620923` (name), `Player` `6a38ca0a21cb430084f8f8b7` (name), `Player` `6a9d2f18586e6e627cf46f93` (archived), `Player` `6a38ca0a21cb430084f8f8b6` (name + injured flag). All changes backed by an authoritative source (the FA's own registered list, or Paul's direct confirmation) — nothing guessed.

## 2026-09-07 — Public website Phase 1 report delivered; four new sponsor records created; badge/founding-year confirmed

Delivered the Phase 1 assessment Paul required before any public-website build work. Headline finding: the existing `/public/*` and `/supporter/*` pages are already live with zero login, and several fetch the full `Player` record client-side with no field whitelisting — since no entity has row-level security, private fields (login PIN, OTP, invite token, phone, email, DOB, emergency contacts) are exposed in the browser today. Flagged as a blocker per Paul's own stated condition; awaiting his go-ahead to patch and to confirm the "extend existing pages, don't duplicate" approach. No code changed yet on this.

Paul supplied real brand assets: Amber's Legacy, GKB Financial Planning, SafeSwitch Solutions, UCS Renewables, David Graham Roofing, DKJ Joinery logos, and the club badge. Confirmed via Paul: club badge name is "Grindon Board Inn" (not "Broadway") and founding year is definitively 1995 — a kit mockup showing "Grindon Broadway ... EST 2021" was wrong and is not to be used. Known Issues #30.

**Database change:** created four new `Sponsor` records — UCS Renewables (type `kit`, priority `main`), DKJ Joinery (`pitch`), David Graham Roofing (`pitch`), SafeSwitch Solutions (`general`) — `logo_url`/`website_url` left null pending Paul supplying websites and uploading logos via the app's own Sponsor admin screen (no image-upload tool available to Claude). Caught and corrected my own mistake before writing: an early sponsorship-category question conflated the `sponsorship_type` field with the separate `priority` field — re-asked correctly before any write.

**Tested:** data-entry only, no code changed — verified via `query_entities` read-back that all four records were created correctly.

## 2026-09-07 (continued) — Fixed the live public/supporter Player data exposure

Paul approved the fix flagged in the Phase 1 report. Created `base44/functions/getPublicPlayerData/entry.ts` — a new backend function following the same safe-aggregator pattern as the existing `getGuestDashboardData`, explicitly whitelisting only public-safe `Player` fields (`full_name`, `nickname`, `position`, `shirt_number`, `photo_url`, `previous_club`, season stats) and never touching `login_pin`, `otp_code`, `invite_token`, `phone`, `email`, `date_of_birth`, or emergency contacts.

**Code changes:** replaced the direct `base44.entities.Player.filter({active:true, archived:false})` call with `base44.functions.invoke('getPublicPlayerData', {})` in all 5 affected pages: `src/pages/public/PublicSquad.jsx`, `src/pages/supporter/SupporterSquad.jsx`, `src/pages/supporter/SupporterStats.jsx`, `src/pages/supporter/SupporterMatchCentre.jsx`, `src/pages/public/SupporterHome.jsx`.
**Tested:** `npm run build` clean; targeted `eslint` on all 6 changed files — only pre-existing, unrelated unused-import errors present (already documented in Known Issues #18), nothing new introduced.
**Checkpoints:** `6a9f346b7548fad476affefe` (before) → `6a9f35ec4dd8fe09cdbc6ac1` / commit `93585d57d7055c0ebe798d7a25224d27399ecd04` (after).
**Still open:** Known Issues #3 — no row-level security anywhere in the app. This fix closes the specific live exposure on these 5 pages; the same direct-entity-read pattern should be checked for other sensitive entities (e.g. `Sponsor`, `ClubNews`) before any further public-website build work.

## 2026-09-08 — Club badge database field fixed; player-card style approach confirmed; two players added to the identity register

Paul confirmed the badge already showing live in the app is correct, so no code/asset change was needed — only `Club.badge_url` (previously pointing at a different, unused file) and `Club.name` (previously blank) needed updating to match reality. Known Issues #30 closed.

Paul also sent a full homepage mockup (desktop + mobile) matching the existing gold/black/dark design already live on the public pages — confirms extending the existing pages, not rebuilding, remains the right approach. Confirmed the player-card graphics he'd sent are a style reference: Claude builds the card component itself using real `Player` data/photos rather than needing pre-made cards per player.

**Database records affected:** `Club` `6a38094e4d090aec62268573` (`badge_url`, `name`) — data-only change, no code touched.

Added two players confirmed via direct database query against real player-card graphics Paul sent — Dave Taylor ("Disco," DEF #4) and Anth Holmes ("Holmsey," FWD #17) — to `PLAYER_IDENTITY_REGISTER.md`. Both were already correct, real, active records; the register was just incomplete.

## 2026-09-08 — Public website: rebuilt the homepage to match Paul's supplied mockup

Paul supplied a full homepage mockup (desktop + mobile). Rebuilt `src/pages/public/PublicHome.jsx` to match it, extending the existing page rather than creating a new one:

- **New sections:** Latest Result banner (real most recent completed fixture, win/loss/draw colour-coded), "Meet the Squad" spotlight (dynamically features the current top scorer — no player hardcoded, updates automatically as stats change), "Programmes" panel (static "pick up at the match" messaging — no real programme PDF exists in the app yet, so nothing was invented or linked).
- **Security fix applied proactively:** found that `Sponsor` records carry undeclared fields beyond the formal schema (address, contact_name, phone, email, contract_notes) — the same category of issue just fixed for `Player`. Built `getPublicSponsorData` (same whitelist pattern as `getPublicPlayerData`) and routed the sponsor strip through it instead of a direct `base44.entities.Sponsor.filter()` call, rather than building new code on top of a known-bad pattern.
- Domain name (`grindonboardinn40s.co.uk`, registered via IONOS) logged — not connected to anything; Paul was advised to leave the domain's DNS/connection settings untouched until the site is ready and he explicitly approves going live.

**Tested:** `npm run build` clean; targeted `eslint` on both changed/new files clean (one pre-existing unused `loading` state variable, present before this change, left alone).
**Checkpoint:** `6a9faa1bca15293b492be5b2` / commit `49d7b81678135dfde8b01ca8e61017b1f032e2cb`.
**Not yet built:** dedicated News and Programmes list pages (the mockup's nav shows these; current nav still only has Home/Fixtures/Squad/Sponsors/LMS/Contact) — flagged to Paul as a separate decision rather than built speculatively.
