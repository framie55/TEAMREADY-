# TeamReady Daily Checklist

_Run this whenever the club manager asks for a "TeamReady daily check." Read-only unless a fix is explicitly flagged as safe and approved. Never change football data, finances, or player identities as part of a routine check._

## Checklist

1. **Build check** — confirm the app still builds cleanly (`npm run build` in the Base44 sandbox).
2. **Automated tests** — run them if/when they exist. (As of 2026-09-07 there are none — see Known Issues. Note this every time until it's fixed, don't silently skip it.)
3. **Recent logs / backend-function failures** — check the `LmsError` entity and `ChairmanDigestLog` entity for new unresolved entries; check the Base44 dashboard function logs for errors in the last 24-48 hours.
4. **Important integrations** — spot-check that Twilio, Stripe, OneSignal, and API-Football credentials/connections are still valid (a sudden burst of `LmsError` "Authenticate" failures is the usual symptom of an expired Twilio credential).
5. **Recent data inconsistencies** — sample-check that `Fixture.result_home/result_away` matches `MatchReport.home_score/away_score` for recently completed matches.
6. **Duplicate/unlinked player records** — scan `Player` for near-duplicate names or nicknames that resolve to more than one active record. Flag, don't merge.
7. **Upcoming fixtures with missing information** — any fixture in the next 7 days missing opponent, kickoff time, or venue.
8. **Incomplete availability/squad-selection** — for the next fixture, how many players have not yet responded; whether a `TeamSheet` has been published.
9. **Results vs statistics agreement** — for the most recently completed fixture, confirm "Finalise Match" was actually run (i.e. `PlayerMatchStat` rows exist for that fixture, not just bumped `Player.season_*` fields).
10. **Scheduled reminders/weekly jobs** — confirm the Thursday availability chase, matchday reminders, and any LMS broadcast jobs actually fired when expected (cross-check `ChairmanDigestLog`/`LmsError`/Base44 function run history).
11. **Recent code changes** — skim the last few days of commits for anything touching payments, player identity, or stats logic in particular; flag anything that looks risky or untested.
12. **Anything requiring the manager's decision** — surface it, don't resolve it unilaterally.

## Report format

Return a short report:

- **Overall status:** Green / Amber / Red
- **Checks completed:** (list)
- **Problems found:** (list, with evidence — file/entity names)
- **Safe fixes completed:** (only truly safe, reversible, non-data-altering fixes — e.g. re-running a recompute function, never merging players or editing financial records)
- **Items waiting for manager approval:** (list, with your recommendation and the alternative)
- **Recommended next action**

Do not make code changes just to show activity. A quiet, all-green day should produce a short report, not a padded one.
