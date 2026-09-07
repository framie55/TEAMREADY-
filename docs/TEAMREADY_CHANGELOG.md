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
