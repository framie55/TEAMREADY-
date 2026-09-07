# Daily Operations

Run this whenever Paul asks for "the TeamReady daily check." Read-only unless a fix is explicitly flagged as safe and approved. Never change football data, finances, or player identities as part of a routine check.

## Checklist

1. Build check — `npm run build` in the Base44 sandbox, confirm exit 0.
2. Automated tests — none exist yet (Known Issues #6). Note this every time, don't silently skip it.
3. Recent logs/failures — check `LmsError` and `ChairmanDigestLog` for new unresolved entries; check Base44 dashboard function logs for the last 24–48 hours.
4. Integrations — spot-check Twilio (being phased out, see `AUTOMATION_REGISTER.md`), Stripe, OneSignal, API-Football credentials are still valid. A burst of `LmsError` "Authenticate" failures is the usual symptom of an expired credential.
5. Data consistency — sample-check `Fixture.result_home/result_away` against `MatchReport.home_score/away_score` for recently completed matches.
6. Duplicate/unlinked players — scan `Player` for near-duplicate names/nicknames resolving to more than one active record. Flag, don't merge. See `PLAYER_IDENTITY_REGISTER.md`.
7. Upcoming fixtures — any fixture in the next 7 days missing opponent, kickoff time, or venue.
8. Availability/selection — for the next fixture, how many players haven't responded; whether a `TeamSheet` has been published.
9. Results vs. stats — for the most recently completed fixture, confirm `PlayerMatchStat` rows actually exist (i.e. Finalise Match ran, not just that the fixture shows a score).
10. Scheduled jobs — confirm the Thursday availability chase, matchday reminders, and LMS broadcasts actually fired when expected (cross-check `LmsError`/`ChairmanDigestLog`/Base44 function run history).
11. Recent code changes — skim the last few days of Base44 checkpoints/commits for anything touching payments, player identity, or stats logic specifically.
12. Anything requiring Paul's decision — surface it, don't resolve it unilaterally.

## Report format

- **Overall status:** Green / Amber / Red
- **Checks completed:** (list)
- **Problems found:** (list, with evidence — entity/file names)
- **Safe fixes completed:** (only truly safe, reversible, non-data-altering — e.g. re-running a recompute function; never merging players or editing financial records)
- **Items waiting for Paul's approval:** (list, with a recommendation and the alternative)
- **Recommended next action**

A quiet, all-green day should produce a short report, not a padded one. Never make code changes just to show activity.

## Standing automation

A weekly (not daily) Routine exists — **"TeamReady Weekly Admin Check-in,"** Sundays 08:00 UTC (09:00 UK) — that messages Paul asking for the previous Saturday's match result, Finalise Match confirmation, a league-table screenshot, and opposition scouting screenshots if relevant, then acts on whatever he sends (FA team sheet generation, league table update). See `WEEKLY_OPERATIONS.md`. This fires into this same persistent Claude Code session — if a future firing can't reach the TeamReady app data properly, that's a sign the session binding needs checking, not a data problem.
