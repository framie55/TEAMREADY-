# TeamReady System Map

_Plain-English + technical reference for how TeamReady actually works. Written for both the club manager and any developer/AI picking this up. Last updated: 2026-09-07 (Phase 1 inspection). Keep this updated whenever the app changes materially._

**Canonical copy of this document lives in the TeamReady Base44 app itself (`docs/` folder, app ID `6a359ec6743bebc341bd156f`), since that's where the live app runs. This copy is mirrored here for the project's git history.**

## 1. What TeamReady is

TeamReady runs the weekly and matchday operations of **Grindon Board Inn Over 40s FC**: players, availability, fixtures, squad selection, formations, live match scoring, stats, match reports, Player of the Month/Match, training, the "Last Man Standing" side competition, club finances/subs, sponsors, the weekly programme, and matchday reminders.

It is **not a normal GitHub-deployed app** — it is built and hosted entirely inside **Base44** (a low-code AI app builder). The live app, its code, and its own git history all live in Base44's own infrastructure (App ID `6a359ec6743bebc341bd156f`).

## 2. Who uses it

- **Admins / manager / assistant manager / chairman / treasurer** — staff roles, log in with email+password (or Google) via Base44's built-in auth.
- **Players** — no password. Four different passwordless login routes exist: a long-lived invite link, SMS one-time code, name + 4-digit PIN, or an emailed magic link.
- **Guests / supporters / public** — read-only public pages (fixtures, news) and a token-gated Last Man Standing entry flow for non-players.

## 3. Tech stack

- **Frontend:** React 18 + Vite 6, React Router v6, Tailwind CSS + shadcn/ui components, TanStack React Query for data fetching, React Hook Form + Zod for forms.
- **Backend:** ~90 serverless functions written in TypeScript (Deno), one per file under `base44/functions/`.
- **Data:** ~95 data models ("entities") defined as `.jsonc` schema files under `base44/entities/`, stored in Base44's managed database.
- **Build tool:** Vite. Type-checking via `tsc` against `jsconfig.json` (checks the plain `.jsx` files, not a TypeScript app).

## 4. Feature map — what depends on what, and the source of truth

| Feature | Main pages | Source-of-truth data | Depends on |
|---|---|---|---|
| Players | `Squad.jsx`, `AppUsers.jsx` | `Player` entity | — (root record everything else links to) |
| Availability | `Availability.jsx`, `PlayerAvailResponse.jsx` | `AvailabilityResponse` | `Player`, `Fixture` |
| Fixtures | `Matches.jsx` | `Fixture` (`result_home`/`result_away` live here) | — |
| Squad & formation | `TeamSelection.jsx` | `TeamSheet` + `TeamSheetPosition` | `Player`, `Fixture`, `AvailabilityResponse` |
| Live match scoring | `LiveMatchCentre.jsx` | `MatchTimelineEvent` (event log) | `TeamSheet`, `Fixture` |
| Official per-match stats | (produced by backend, not a page) | `PlayerMatchStat` | Rebuilt from `MatchTimelineEvent` by the `finaliseMatch` function when you press **Finalise Match** |
| Season totals shown around the app | — | Cached fields on `Player` (`season_goals`, `season_assists`, etc.) | Written from **three separate places** — see Known Issues |
| Match reports | `MatchReport.jsx` (manual), `AiMatchReport.jsx` (AI, share-only) | `MatchReport` entity | `Fixture`, `MatchTimelineEvent`; also auto-gets an unreviewed AI analysis via `generateMatchReportAi` |
| Weekly programme | `ProgrammeManager.jsx` | `Programme`, `ProgrammeAsset` | Live-pulls `Fixture`, `Player`, `Sponsor`, `LeagueTable`, `MatchReport`, `AvailabilityResponse`, `Payment`, `Expense` |
| Last Man Standing | `LmsPickPage.jsx`, `LmsPublicPage.jsx`, admin panels under `src/components/lms/` | `LmsCompetition`, `LmsGameweek`, `LmsPick`, `LmsEntry`, `LmsResultsCache` | Real Stripe entry fees, real Twilio SMS reminders, real fixture data from API-Football |
| Finances | `Finances.jsx`, `MatchFeeTracker.jsx` | `Payment` (subs/fines, Stripe or cash) | **Two other, separate ledgers also exist** — `RewardsFund`/`RewardsLevyTransaction` (a rewards-pot levy skimmed off match fees) and `FundraisingDonation` (fundraising, unrelated to player subs). These are not unified — see Known Issues. |
| Sponsors | `Sponsors.jsx`, `SponsorHub.jsx`, `CommercialHub.jsx` | `Sponsor`, `SponsorLead` | — |
| Awards/MOTM | `PlayerOfMonth.jsx` (monthly, `PotmVote`/`PotmPoll`), `ManagerMotm.jsx` (per-match manager pick, `MotmVote`), `MotmVotePage.jsx` (public fan vote on the same `MotmVote` record) | See entity names above | These are three **deliberately separate** features, not duplicates, despite similar names. |
| Reminders/comms | Real: Twilio SMS (`base44/functions/sendSms`, `sendPostMatchSocial`, several scheduled-looking chase functions), OneSignal push (`sendPushNotification`). **Not real:** every "WhatsApp" feature in the app just opens a pre-filled `wa.me` link — a human still has to tap send. |
| Facebook | `postToFacebook` function | Real, connected Graph API integration (page: "Grindon Broadway 040s") |
| Weather | `fetchFixtureWeather` | Real, free Open-Meteo API, no key needed |

## 5. Authentication & permissions

- Staff: Base44-managed email/password or Google login (`Login.jsx`, `Register.jsx` — note `Register.jsx` currently has no route pointing to it, see Known Issues).
- Players: `loginWithToken` (365-day invite link), `sendPlayerOtp`/`verifyPlayerOtp` (SMS code), `loginWithNamePin` (name + PIN), `sendPlayerEmailLink`/`verifyPlayerEmailLink` (magic link).
- Guests (Last Man Standing only): `loginWithGuestNamePin`, `validateGuestToken`.
- **Important:** none of the ~95 data entities declare any row-level security (read/write) rules. Access control today relies entirely on individual pages and functions checking `user.role` themselves (e.g. the chairman digest function checks `role === 'admin'` before running). There is no data-layer safety net. See Known Issues — this needs a decision from the club manager, not a unilateral fix.

## 6. Backend functions, grouped

Roughly 90 functions live under `base44/functions/`, one folder each. Groups:

- **Matchday/fixtures:** `finaliseMatch` (converts live match events into official stats — the single most important function in the app), `recalculatePlayerStats`, `applyUnavailabilityToFixture`, `fetchFixtureWeather`, `aiTeamSelector`, `generateMatchReportAi`.
- **Availability/reminders (SMS/push):** `fridayAvailChase`, `thursdayNoResponseChase`, `sendAvailabilityReminder`/`sendAvailabilitySms`, `matchday24hrReminder`, `matchdayMorningPush`/`matchdayMorningSms`, `noonAwaySms`.
- **Last Man Standing:** the largest single group (~28 functions) — fixture syncing (`lmsGetFixtures`), public pick pages, results processing/elimination (`lmsProcessResults`), integrity checks (`lmsIntegrityCheck`, `lmsFridayIntegrityAlert`), a long list of scheduled-looking SMS broadcasts, and Stripe checkout functions for entry/buy-back fees.
- **Chairman/reporting digests:** `chairmanDigest` (weekly summary — currently only fires when an admin triggers it manually, despite being written to look automatic), `weeklyReliabilityDigest`.
- **Awards:** `openMotmVote`, `announceMotmWinner`, `autoPlayerOfMonth` (runs on the 1st of the month), `checkPlayerMilestone`.
- **Payments:** `stripeWebhook` (the actual source of truth for "has this been paid" — never trust a client-side flag), `matchFeeStripeCheckout`, `lmsStripeCheckout`, `bankCashDeposit`.
- **Auth:** the login/invite/OTP functions listed above.
- **Notifications infrastructure:** `notificationEngine` (a dispatcher that explicitly lists WhatsApp only as a *future*, unimplemented provider), `sendSms`, `sendPushNotification`.
- **Social/marketing:** `postToFacebook`, `sendPostMatchSocial`, `generateReferralCode`.

**Scheduling caveat:** which of these run automatically, and when, is configured in Base44's own dashboard/scheduler — it is not visible in the code itself. Comments inside several functions describe an intended schedule (e.g. "Runs automatically every Thursday at 2pm"), but that is the developer's note, not proof the schedule is actually configured and firing. **Confirm actual schedules in the Base44 dashboard, don't assume from code comments.**

## 7. Build, test, deploy

- Build: `npm run build` (Vite) — currently passes cleanly.
- Tests: **none exist.** No test framework, no test files, no `test` script.
- Lint: `npm run lint` — 314 issues, all the same harmless "unused import" rule, auto-fixable with `npm run lint:fix`.
- Type-check: `npm run typecheck` — 648 issues, concentrated in the newer public/supporter/player-portal pages. Doesn't block the build, but is a real early-warning signal worth acting on.
- Deploy: Base44 builds (`npm run build`) and serves (`npm run dev`) the app itself per `base44/config.jsonc`; there is no separate GitHub Actions/CI pipeline — version control is Base44's own S3-backed git remote.

## 8. How to recover from common failures

- **App won't build / preview is broken:** run `npm run build` in the Base44 sandbox first to see the real error before assuming anything is badly wrong — this has been quick and clean historically.
- **A scheduled reminder/digest didn't go out:** check the `LmsError` entity and `ChairmanDigestLog` entity first — real failures are actively logged there (confirmed live, unresolved entries exist as of this writing — see `TEAMREADY_KNOWN_ISSUES.md`). Then check the function's own logs in the Base44 dashboard, and confirm Twilio/OneSignal credentials haven't expired (these are Base44 platform "Secrets," not visible from the code).
- **A player's stats look wrong:** don't hand-edit `Player.season_*` fields. Re-run `recalculatePlayerStats` (documented in its own file as "SOURCE OF TRUTH: PlayerMatchStat") — it recomputes everything from the real match-event records.
- **Uncertain whether a payment went through:** trust `stripeWebhook`/the `Payment.status` field over anything shown client-side — the webhook is the documented source of truth for Stripe payments.

## 9. Areas that always require manual/human confirmation before acting

- Merging or editing any player identity record (nicknames, duplicate names).
- Treating AI-generated match analysis as fact rather than a draft.
- Any financial reconciliation across the three separate money ledgers (`Payment`, rewards levy, fundraising).
- Publishing programme content, match reports, or sending any broadcast message externally.
- Changing who can access what data, given there are currently no data-layer access rules at all.
