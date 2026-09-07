# Weekly Operations

The club's real weekly routine, mapped onto actual app screens/functions and everything established with Claude as of 2026-09-09.

## Before the match

| Step | Where | Automated? |
|---|---|---|
| Confirm opponent, competition, date, kick-off, home/away, venue | `Matches.jsx` → `Fixture` | Manual entry, no required-field validation |
| Collect availability | `Availability.jsx` (admin), `PlayerAvailResponse.jsx` (player) | Real SMS chase exists (`thursdayNoResponseChase`, `fridayAvailChase`) but depends on the Twilio decision — see `AUTOMATION_REGISTER.md` |
| Show who hasn't replied | `Availability.jsx` counts | Automatic display |
| Prepare squad, starting XI, subs, formation (+ alternative, e.g. 3-5-2) | `TeamSelection.jsx` | Manual, reads live availability — no re-typing |
| Directions and matchday reminders | Real SMS/push (`matchday24hrReminder`, `matchdayMorningPush`/`Sms`, `noonAwaySms`) | Depends on Twilio decision |
| Opposition scouting | Manual — screenshot the opponent's FA results/players page (site can't be scraped, confirmed Cloudflare-blocked 2026-09-07), then run through the "AI scout" feature | Screenshot step confirmed unavoidable |

## Matchday

1. Live scoring in **Live Match Centre** (`LiveMatchCentre.jsx`) — the only screen that should be used to record goals/cards/subs. **Matchday Studio is read-only** (fixed 2026-09-09) and must never be used for recording.
2. **Confirm Full Time** now automatically finalises the match (as of 2026-09-09) — one action produces the official score and the official per-player stats. The "Re-finalise Match" button still exists for safe re-runs after a later correction.
3. Undo only reverses the single most recent event — for anything further back, undo forward to that point and re-enter (Known Issues, still open).

## After the match

1. Confirm scorers, assists, appearances/subs, GK/clean sheet, cards, MOTM (manager's pick — currently only said in the FA email, not saved anywhere structured; the player vote is saved via `MotmVote`).
2. Player subs paid — `Finances.jsx`/`MatchFeeTracker.jsx`, or record any off-platform payment (Monzo, cash) as a `Payment` with `status: cash_received` and a clear note of how it was actually collected — never guess a split of an amount that wasn't shown.
3. **FA Upload Team Sheet** — one click on the Match Report page (`MatchReport.jsx`, "Generate FA Team Sheet" button, added 2026-09-09) builds the full sheet — lineup, subs with times, goals, cards, MOTM — from real recorded match data. Review before emailing to Micky Greenwell (Player Registrations).
4. **Match report writing** — Paul dictates, Claude drafts a professional version and writes it into Base44 directly (agreed 2026-09-09, replacing the previous voice→ChatGPT→copy→paste-into-Claude→Base44 chain). Confirm destination each time: news feed summary, the longer programme write-up, or both.
5. **League table update** — Paul screenshots the FA table, sends it, Claude transcribes it into `LeagueTable.rows[]` exactly as shown — keep existing clean team names, never invent a GF/GA split the screenshot doesn't show.
6. **Programme content** — see `PROGRAMME_WORKFLOW.md`. Sanity-check the "Results This Season" panel against real `Fixture` data before it goes out — a real mix-up (two similarly-named Darlington opponents swapped) was found and needs the source traced (Known Issues).
7. Update Last Man Standing (results processing, buyback/pick reconciliation) — verify against `Payment`/`LmsPick`, never trust `entry_fee_paid_pence` alone as proof of payment.

## Standing weekly reminder

**Routine: "TeamReady Weekly Admin Check-in"** — fires every Sunday 08:00 UTC (09:00 UK) into this persistent session (trigger id `trig_01FBZHf6in1d7rkgX1yA3NKm`, created 2026-09-09). Asks Paul for: final score/opponent, Finalise Match confirmation, league table screenshot, opposition scouting screenshots if needed. On reply: generates the FA team sheet, updates the league table, flags inconsistencies rather than accepting them silently. Editable/cancellable on request — day, time, and content can all change.

## The actual communications routine (as described by Paul, not yet all built/automated)

- Tuesday 18:00 — initial availability request
- Wednesday — availability update/heads-up
- Friday 12:00 — reminder
- Friday 18:00 — final reminder where needed
- Saturday morning — directions and match information
- After the match — result and match information
- Saturday ~12:05 — post-match pub information (where applicable)

**Status: documented as the target routine, not yet implemented as editable admin settings in the app.** Currently the actual SMS-based automation depends on Twilio, which is being phased out — see `AUTOMATION_REGISTER.md` for what's real vs. aspirational here. Before connecting or sending any WhatsApp-based automation, Paul must see what will be sent, who receives it, when, and whether it's automatic or needs approval — nothing sends until that's shown and approved.
