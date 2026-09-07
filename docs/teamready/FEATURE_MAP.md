# Feature Map

For each feature: what it does, the main page(s), the real source-of-truth data, and what it depends on. See `DATABASE_MAP.md` for entity field detail.

| Feature | Main page(s) | Source-of-truth data | Depends on | Status |
|---|---|---|---|---|
| Players | `Squad.jsx`, `AppUsers.jsx` | `Player` | — | Working. No duplicate-name check on create (Known Issues #10) |
| Availability | `Availability.jsx` (admin), `PlayerAvailResponse.jsx` (player, token link) | `AvailabilityResponse` | `Player`, `Fixture` | Working, two visually different UIs for the same action (in-app vs SMS-link) |
| Fixtures | `Matches.jsx` | `Fixture` (`result_home`/`result_away`) | — | Working, no required-field validation |
| Squad & formation | `TeamSelection.jsx` | `TeamSheet` + `TeamSheetPosition` | `Player`, `Fixture`, `AvailabilityResponse` | Working, real draft→publish workflow |
| Live matchday scoring | `LiveMatchCentre.jsx` | `MatchTimelineEvent` | `TeamSheet`, `Fixture` | Working. Fixed 2026-09-09: Full Time now auto-finalises (see Known Issues #23) |
| Official per-match stats | (backend only) | `PlayerMatchStat` | Rebuilt from `MatchTimelineEvent` by `finaliseMatch` | Working since the auto-finalise fix |
| Season totals (displayed everywhere) | — | Cached fields on `Player` (`season_goals` etc.) | Written by 3 places — see Known Issues #7 | Working but has a residual drift risk if `handleFullTime`'s own increment and `finaliseMatch`'s recompute ever disagree |
| Match reports (manual) | `MatchReport.jsx` | `MatchReport` entity is written by `finaliseMatch`, NOT by this page — this page instead publishes to `ClubNews` | `Fixture` | Confusing naming: the page and the entity are not the same thing |
| **FA Upload Team Sheet generator** | `MatchReport.jsx` (button added 2026-09-09) | Pulls live from `Fixture`, `TeamSheet`/`TeamSheetPosition`, `MatchTimelineEvent`, `MotmVote` | — | Built and verified against a real sent email, matched exactly |
| AI match analysis | `AiMatchReport.jsx` (share-only draft), `generateMatchReportAi` function (auto-writes `MatchReport.ai_analysis`, no approval gate) | `MatchReport.ai_analysis` | `MatchTimelineEvent` | Working but unreviewed AI content feeds `MatchIntelligence.jsx`/`ClubBrain.jsx` averages — Known Issues #8 |
| Weekly matchday programme | Generated externally as a 24-page PDF/graphic pack, pulling from the app (exact generation pipeline not fully traced — see `PROGRAMME_WORKFLOW.md`) | Multiple entities (see that doc) | Nearly everything else | Mostly automated, one confirmed data mix-up found 2026-09-07 (two similarly-named opponents swapped in a results panel) |
| Awards | `PlayerOfMonth.jsx` (`PotmVote`/`PotmPoll`, monthly), `ManagerMotm.jsx` (`MotmVote`, manager's per-match pick), `MotmVotePage.jsx` (public fan vote, same entity) | See entity names | — | Three deliberately separate systems, not duplicates, despite similar names. Manager's own "pick" (as used in the FA email) has no dedicated storage field currently used — only the player vote is saved to `MotmVote` |
| Last Man Standing | `LmsPickPage.jsx`, `LmsPublicPage.jsx`, `src/components/lms/*` | `LmsCompetition`, `LmsGameweek`, `LmsPick`, `LmsEntry`, `LmsGuestEntrant`, `LmsResultsCache`, `LmsError` | Real Stripe checkout, real (formerly Twilio, now largely disabled) reminders | Real money — `lmsFeeSplit.ts` splits every entry/buyback fee 80% prize pot / 5% players' pool / 15% club pot (10/5 if there's a valid referrer) |
| Finances | `Finances.jsx`, `MatchFeeTracker.jsx` | `Payment` (subs/fines, Stripe or manually marked `cash_received`) | — | Three separate, unreconciled money ledgers exist — `Payment`, the LMS rewards levy (`RewardsFund`/`RewardsLevyTransaction`), and `FundraisingDonation` — Known Issues #12 |
| League table | Displayed across several pages (`LeagueHub.jsx` etc.) | `LeagueTable` (single record, `rows[]` array) | Manually transcribed from an FA screenshot each week — **cannot be scraped**, confirmed 2026-09-07 (Cloudflare-blocked) | Working, manual update process now established with Claude |
| Opposition scouting | `OppositionHub.jsx` and related, `OppositionRecord`/`OppositionFullTimeData` | Manually transcribed from FA opponent pages (screenshot workflow), then run through an "AI scout" feature | — | Manual data entry step confirmed unavoidable (same Cloudflare block) |
| Reminders/comms | Real: Twilio SMS, OneSignal push. Not real: every "WhatsApp" feature is a `wa.me` link a human must tap send on | `notificationEngine`, `sendSms`, `sendPushNotification` | — | Twilio being phased out (cost) — see `AUTOMATION_REGISTER.md` |
| Matchday Studio (public broadcast view) | `MatchdayStudio.jsx` | Reads `Fixture`, `MatchTimelineEvent`, `TeamSheet`/`TeamSheetPosition` (read-only) | — | **Fixed 2026-09-09**: previously had a full duplicate live-recording engine (Kick Off/Goal/Card/Sub/Full-Time) on a public, unauthenticated route — removed entirely. Now purely a read-only scoreboard/timeline/stats/MOTM-vote/photos view |
| Facebook posting | `postToFacebook` function | Real, connected Graph API (page: "Grindon Broadway 040s") | — | Working |
| Weather | `fetchFixtureWeather` | Real, free Open-Meteo API | — | Working |

## Match report dictation workflow (in progress, 2026-09-09)

Paul currently dictates a match report by voice into ChatGPT (for professional tone), copies the result into a separate Claude session, then manually pastes it into Base44. Agreed direction: Paul can instead dictate/type directly into a Claude Code session (this one or a future one), Claude drafts the professional version, and writes it straight into Base44 — cutting the ChatGPT hop and the manual paste. **Not yet built as a page feature** — currently done conversationally. Destination field (news feed summary vs. the longer programme write-up vs. both) — *Needs Paul's confirmation each time until a default is agreed.*
