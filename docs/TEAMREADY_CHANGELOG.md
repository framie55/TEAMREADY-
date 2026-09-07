# TeamReady Changelog

_Every change made by the technical manager/maintenance agent is recorded here — what changed, why, and how it was tested. Documentation-only additions are included too, since they're part of the app's own history._

**Canonical, always-current copies of these documents live inside the TeamReady Base44 app itself (`docs/`, app ID `6a359ec6743bebc341bd156f`), tied to that app's own checkpoints/commits. This repo mirrors them.**

## 2026-09-07 — Phase 1 & Phase 2: Initial inspection and operations documentation

**What happened:** Completed a full read-only inspection of TeamReady (tech stack, ~95 data entities, ~90 backend functions, all major workflows, git history, tests, security). No application code, data, or configuration was changed. Findings were presented to the club manager as a plain-English assessment.

Following manager approval, created the six standing operations documents under `docs/`:
- `TEAMREADY_SYSTEM_MAP.md`
- `TEAMREADY_DAILY_CHECKLIST.md`
- `TEAMREADY_WEEKLY_WORKFLOW.md`
- `TEAMREADY_DATA_RULES.md`
- `TEAMREADY_KNOWN_ISSUES.md`
- `TEAMREADY_CHANGELOG.md` (this file)

**Files affected:** new files only, under `docs/` — no existing file was modified.

**Testing:** documentation only; no build/test impact. App build was verified clean before and remains untouched.

**Outstanding items raised, awaiting manager decision (see `TEAMREADY_KNOWN_ISSUES.md` for full list):**
- A live, unresolved production failure in the weekly Last Man Standing SMS reminder (Twilio auth error).
- An unresolved LMS integrity alert (paid entries vs. picks mismatch).
- No data-access rules exist on any entity — needs a decision on who should access what.
- A confirmed duplicate player record (James Dickinson) and an unexplained extra "Paul Griffith" record — awaiting manager confirmation before any merge/edit.

Nothing above was fixed in this session. Next planned stage: a full prioritised health-check report (Phase 3), then safe fixes taken one at a time with manager approval per the Safe Change Process.

## 2026-09-09 — Player identity confirmations; disabled dead Twilio SMS in the LMS pick reminder

**Player identity, confirmed by club manager, recorded in `TEAMREADY_DATA_RULES.md`:**
- Mark Dickinson (id `6a362b0234d01f9e9562092c`) is a separate, real, unambiguous player — not to be confused with James "Jiffy" Dickinson.
- The two Paul Griffiths (Griffo, outfield, id `...092b`; Griff, goalkeeper, id `...5d26`) are confirmed correct and distinct — no change needed.
- Investigated the two flagged duplicate/stray records (Known Issues rows 4 and 5): both are empty (zero stats) and already archived — one self-corrected same-day, one already caught and annotated by an admin back in August. Downgraded from Critical to Low; no data was at risk. Left untouched pending the manager's call on whether to delete them outright (cosmetic only).

**Code change:** `base44/functions/sendWeeklyPickSms/entry.ts` — the weekly Last Man Standing pick-reminder function was still calling Twilio, which the club has deliberately stopped funding (cost of message volume). Every attempt was failing per-recipient with a Twilio auth error and logging a separate `LmsError` row each time (confirmed real failures on 2026-09-01 and 2026-09-04).

- **Fix:** added `const SMS_ENABLED = false` guard; the function now skips the Twilio call entirely and logs one clear, actionable `LmsError` summary ("N entrants need a manual reminder") instead of one failure per person. All existing logic (competition/gameweek lookup, message building, payment-link lookup) is untouched, so re-enabling later is a one-line flip back to `true`.
- **Why this function first:** it was the one with confirmed live failures and a real-money competition attached (paid entries vs. picks-submitted mismatch, Known Issues row 2).
- **Scope intentionally limited:** ~14 other functions have the same dead-Twilio dependency (availability chases, matchday reminders, chairman digest, MOTM SMS, other LMS broadcasts) — not touched yet, logged as Known Issues row 21, pending the manager's decision on a replacement channel.
- **Tested:** `npm run build` verified clean after the change (exit 0, no new errors). No test suite exists to run (see Known Issues row 6). Not live-invoked against production to avoid an out-of-schedule side effect; verified by code review and dry-run diff instead.
- **Checkpoints:** `6a9e8a34cd6c82d0ee2d54f4` = state immediately before this fix (restore to this to undo). `6a9eba00704d546ece83d190` (git commit `ca87ccc58d19d1268086119ad0665438e2b9abe6`) = state immediately after this fix, taken as the rollback point.

## 2026-09-09 (continued) — Explained the LMS integrity alert; confirmed a delete-tooling limitation

**Investigated Known Issues #2, #4, #5 fully, at the club manager's request:**

- The "20 paid entries, 21 picks submitted" alert is fully explained: the extra pick belongs to a phantom `LmsEntry` (`6a988c49fe9ec8f718e9c0c3`) created against the archived duplicate "Paul Griffith" record. Verified via direct query: no `Payment` record exists for it (the fee was never actually collected, only ever an expected-amount field), `joined_at`/`terms_accepted_at` were never set, and its `LmsPick` record was manually set to `team_picked: "VOID — invalid entry"` by the manager on 2026-09-03, the same day they spotted and eliminated it. No real entrant, no money ever at risk. Marked Resolved.
- The manager approved deleting the empty James Dickinson duplicate (`6a9d2f18586e6e627cf46f91`, confirmed zero references anywhere). **Not yet deleted** — the maintenance agent's toolset only supports create/update/query on entity records, not delete (the platform itself supports deletion via `entities.X.delete(id)`, confirmed by grepping the app's own code, but only from inside the running app or a real function, not from a generic tool available here). Manual deletion via Squad in the app was recommended instead of building a one-off workaround for a single empty, harmless record.
- The "Paul Griffith" player record (`...fe3c`) was deliberately left alone: it's still referenced by the (already-voided, harmless) `LmsEntry`/`LmsPick` pair above, so deleting the player now would only create orphaned references. Recommendation stands: leave archived unless the manager wants all three records removed together as one deliberate operation.

**No data was changed in this entry** — investigation and documentation only.

## 2026-09-09 (continued) — Matchday UX: removed public recording risk in Matchday Studio; auto-finalise on Full Time

At the club manager's request, audited the player portal and the live matchday recording screen for UX friction (two read-only research passes). Two concrete, approved fixes came out of the matchday audit:

**1. Removed the entire live-recording control surface from Matchday Studio** (`src/pages/MatchdayStudio.jsx`). This page turned out not to be a simple broadcast view with a couple of extra buttons — it was a full second live-match engine (Kick Off, Half Time, Second Half, Goal, Card, Sub, Full Time), each writing directly to `MatchTimelineEvent`, `Fixture`, and `Payment`, on a route (`/matchday-studio`) with **no login or role check at all**. Confirmed with the club manager that this recording capability has never actually been used — Live Match Centre is the only screen used to run a match — and confirmed separately that the page's read-only display (scoreboard, timeline, on-pitch/bench, score) already loads correctly from real database records on open, independent of these controls. Removed: the Kick Off button, the Goal/Card/Sub/Half-Time/Full-Time action grid, the `ActionBtn` helper, the four Record*/FullTime modals and their imports, the goal-celebration overlay, and all now-orphaned state/handlers (`handleKickOff`, `handleHalfTime`, `handleSecondHalf`, `handleGoalRecorded`, `handleCardRecorded`, `handleSubRecorded`, `handleFullTime`, `addEvent`, plus the now-unused `Play`/`useCallback` imports). Nothing else on the page (broadcast scoreboard/timeline, stats, MOTM voting, photos) was touched.

**2. Folded "Finalise Match" into "Confirm Full Time"** (`src/pages/LiveMatchCentre.jsx`). Per Known Issues #7/#23, a match could look fully completed while official per-player stats silently stayed unfinalised, because finalising was a separate, easy-to-miss second button. `handleFullTime` now calls `handleFinaliseMatch()` itself at the end — one action, one result. The manual "Re-finalise Match" button is deliberately left in place (finalise is documented as idempotent, `$set` not `$inc`), so a manager can still safely re-run it after correcting a mistake via Undo Last.

**Tested:** `npm run build` verified clean after each change and again after final cleanup (exit 0, no new errors). Ran `npx eslint` on both changed files specifically to check for anything newly broken by the edits (not a full-repo lint pass) — found and removed one newly-unused import (`useCallback`) caused by the edit; all other warnings shown were pre-existing and unrelated to this change. No test suite exists to run (Known Issues #6). Not manually clicked through in a live match (no fixture in progress); verified by full-file reads, dry-run diffs, and build/lint checks instead.

**Checkpoints:** `6a9ebe223850f24a7163c776` = state immediately before these changes (restore to undo). `6a9f08fe076753356d30198d` (git commit `580b4000ac5bf1f72b408b2a18831121ef7e3a91`) = state immediately after, taken as the rollback point.

**Still open from the same UX review, not yet actioned:** duplicate-tap protection on goal/sub/card entry, an on-screen toast when a card fine is raised, per-event edit/delete (today's "Undo" only reverses the single most recent event), and the player-portal structural issues (two home screens, two nav menus, two backend aggregators — login was ruled out as a pain point since the club has already standardised on Name+PIN).

## 2026-09-07 (LMS) — Recorded Kevin Berry's buyback payment and credited the pot

**What happened:** Club manager confirmed (via WhatsApp payment screenshot) that Kevin Berry paid his £10 LMS buyback fee directly via a Monzo link, outside the app's Stripe flow. His `LmsEntry` (`6a3abb05d53687455d99ee26`) and GW4 pick (Aston Villa, `LmsPick` `6a9f0388f17e42aac3d03f73`) had already been entered correctly by the manager in the app; the only gap was the payment record and pot credit, since the Monzo payment bypassed the automatic Stripe webhook that normally does this.

- Verified there is exactly one "Kevin Berry" player record (id `6a362b0234d01f9e95620926`) — no identity ambiguity.
- Created a `Payment` record (`6a9f0a4cc4ac78dc5c4bb7d7`, type `lms_buyback`, £10.00, status `cash_received` with `cash_confirmed_by`/`cash_confirmed_at` set) so this payment is now auditable in the ledger like any other manually-collected payment, notes explicitly stating it was via Monzo, not Stripe.
- Linked it via `LmsEntry.entry_payment_id`.
- Credited the competition pots using the **exact same split the app's own Stripe flow uses** (`base44/shared/lmsFeeSplit.ts`: 80% prize pot / 5% players' pool / 15% club pot when there's no valid referrer — confirmed Kevin's entry has no referrer code) rather than putting the full £10 into the prize pot, so his contribution stays consistent with every other entrant's: **prize pot +£8.00 (£376→£384), club pot +£1.50 (£68→£69.50), players' pool +£0.50 (£23.50→£24).**

No code was changed; this is a data entry, done manually because the payment happened outside the app's own payment flow.
