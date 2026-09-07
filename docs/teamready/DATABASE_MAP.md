# Database Map

~95 entities exist in total (`base44/entities/*.jsonc`). This covers the entities actually verified through direct use this session — the ones that matter for daily operation. Treat any entity not listed here as *Needs investigation* before relying on it. No entity in the app declares row-level security (RLS) — see `CLUB_RULES.md`.

## Core identity

**`Player`** — the root record everything else links to. Fields include `full_name` (only required field), `nickname` (single string), `position`/`primary_position`/`secondary_position`, `date_of_birth`, `phone`, `email`, `login_pin`, `invite_token`, `active`/`archived`/`archived_at`, cached season stats (`season_goals`, `season_assists`, `season_appearances`, `season_starts`, `season_sub_apps`, `season_yellow_cards`, `season_red_cards`, `season_minutes_played`, `season_player_motm`, `season_manager_motm`), `shirt_number`, `reliability_score`, `selection_score`, `referral_code`/`referred_by_code`, `notes` (free text — sometimes contains real operational info, e.g. payment preferences or "duplicate of X" annotations). **No uniqueness constraint on name or nickname, no verification flag.** See `PLAYER_IDENTITY_REGISTER.md`.

**`User`** — built-in Base44 auth entity. `role` enum: `admin, user, chairman, manager, asst_manager, treasurer, player`. Separate from `Player` (a person can exist as both).

## Fixtures and matchday

**`Fixture`** — `opponent`, `opponent_id`, `home_away`, `kickoff_datetime`, `venue`, `competition`, `status` (`scheduled`/`completed`/`cancelled`), `result_home`/`result_away` (**literal home/away team scores, not "us vs them"** — always check `home_away` before reading these), `meet_time`, weather fields, various `*_sms_sent_at` timestamps.

**`TeamSheet`** — one per fixture, `formation`, `published`/`published_at`, `draft_mode`, `reveal_mode`.

**`TeamSheetPosition`** — per-player row on a `TeamSheet`: `player_id`, `role` (`starting`/`sub`), `position_label`, `pitch_x`/`pitch_y` (useful for sorting into a sensible starting-XI order: sort by `pitch_y` descending then `pitch_x` — GK first, forwards last), `is_captain`, `is_vice_captain`.

**`MatchTimelineEvent`** — the real, authoritative log of what happened in a match. `fixture_id`, `event_type` (`kick_off`/`half_time`/`full_time`/`goal`/`yellow_card`/`red_card`/`substitution`), `minute`, `player_id`, `related_player_id` (assist, for goals), `sub_off_player_id`/`sub_on_player_id` (for substitutions), `goal_type` (e.g. `free_kick`, `own_goal`), `notes`.

**`PlayerMatchStat`** — official per-match, per-player stats. Rebuilt (delete+recreate, idempotent) from `MatchTimelineEvent` by the `finaliseMatch` function. **This is the source of truth for "did this player actually get a stat row for this match" — not `Player.season_*`.**

**`MatchReport`** — written by `finaliseMatch` (not by the `MatchReport.jsx` page, confusingly). `fixture_id`, `home_score`/`away_score` (often left null — the real score lives on `Fixture`), `goalscorers[]`/`assists[]`/`yellow_cards[]`/`red_cards[]` (each `{player_id, player_name, minute}` — trust `player_id`), `motm_player_id`/`motm_player_name` (exists in schema but not currently populated by any page — the "manager's pick" used in the FA email isn't saved anywhere structured yet), `ai_analysis` (auto-written, unreviewed).

**`AvailabilityResponse`** — `fixture_id`, `player_id`, `response` (`available`/`maybe`/`unavailable`), `responded_at`, `previous_response`, various reminder-sent timestamps.

## Awards

**`MotmVote`** — per-fixture, players vote for a teammate. `voter_player_id`, `voted_for_player_id`+`voted_for_player_name`, `vote_token`.
**`PotmVote`** / **`PotmPoll`** — monthly/season award.
**`FanMotmVote`** — public fan vote, browser-token based.
**`AwardVote`** — generic category vote; `voter_name`/`nominated_player_name` are the *required* fields, the `_id` equivalents are optional — can exist unlinked to a real player.

## Finance

**`Payment`** — the main ledger. `type` enum: `match_fee, training_fee, fine, event, lms_entry, lms_buyback`. `status` enum: `pending, paid, overdue, waived, cash_promised, cash_received`. `cash_confirmed_by`/`cash_confirmed_at` for manually-collected payments (Monzo/cash) — use `status: cash_received` for these, never invent a payment intent. `amount_pence` throughout (always pence, not pounds).
**`RewardsFund`/`RewardsLevyTransaction`/`RewardsConfig`/`PlayerRewardPoints`/`PlayerRewardSummary`** — a second, parallel points/payout ledger fed by a levy on match fees. Not directly linked to `Payment` by ID.
**`FundraisingCampaign`/`FundraisingDonation`** — a third, independent ledger.

## Last Man Standing

**`LmsCompetition`** — one record per season. `entry_fee_pence`, `buyback_fee_pence`, `prize_pot_pence`/`club_pot_pence`/`players_pool_pence`/`referral_pool_pence` (all in pence, all incremented via `base44/shared/lmsFeeSplit.ts`'s 80/15/5 split — see `AUTOMATION_REGISTER.md`), `current_gameweek`, `max_buybacks`, `allow_buyback`, `buyback_window_expires_gameweek` on each entry.
**`LmsEntry`** — one per registered-player entrant. `player_id`, `status` (`active`/`eliminated`/`winner`/`pending_payment`), `entry_fee_paid_pence` (**this is the expected/recorded amount, not proof a payment actually happened** — always cross-check against a real `Payment` record before trusting it), `entry_payment_id` (often null even when it shouldn't be), `buybacks_used`, `buyback_paid`, `joined_at`/`terms_accepted_at` (null = signup never actually completed).
**`LmsGuestEntrant`** — same shape, for non-players.
**`LmsPick`** — `competition_id`, `player_id` or `entry_id`, `gameweek`, `team_picked`, `pick_submitted_at` (null = no real pick was ever submitted, even if a row exists), `pick_method`.
**`LmsError`** — real production error log, actively used. Check this first when a scheduled LMS job seems to have failed.

## League and opposition

**`LeagueTable`** — single record, `rows[]` array of `{pos, team, p, w, d, l, gf, ga, pts, is_us}`. Manually updated from an FA-site screenshot (site cannot be scraped — Cloudflare-blocked, confirmed 2026-09-07). `gf`/`ga` are frequently `null` for other teams since the FA site only shows goal difference, not the split — never invent a split.
**`Opponent`** — one per opposition club.
**`OppositionRecord`/`OppositionFullTimeData`** — scouting data, manually transcribed from FA screenshots.

## Notes on reliability

- Free-text name fields exist *alongside* ID fields on many entities (`PlayerMatchStat`, `MatchReport` arrays, `HallOfFame`, `AwardVote`, `LmsPick`/`LmsEntry`). **Always trust the ID field over the name field** if they ever disagree.
- No delete tool is available to Claude for any entity — only create/update/query. Deleting a record requires either the app's own UI or a purpose-built one-off function, and should be a deliberate, explained action, not a workaround. See `RECOVERY_GUIDE.md`.
- *Needs Paul's confirmation / further investigation*: the remaining ~60 entities not listed above (training, sponsors, club directory, programme content, social/photos, referrals, etc.) — not yet mapped in this level of detail.
