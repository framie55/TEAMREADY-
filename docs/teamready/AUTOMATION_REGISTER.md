# Automation Register

Every backend function is one folder under `base44/functions/*/entry.ts` (Deno.serve handlers). Scheduling for any of them is configured in Base44's own dashboard, not visible in the code — comments describing an intended schedule (e.g. "runs every Thursday at 2pm") are the developer's note, not proof it's actually configured to fire. Confirm real schedules in the Base44 dashboard, not from code comments.

## Fully documented automations

### sendWeeklyPickSms (LMS pick reminder)
- **Purpose:** Tuesday 18:00 reminder to active LMS entrants to make their weekly pick.
- **Reads:** `LmsCompetition`, `LmsEntry`, `LmsGuestEntrant`, `Payment` (pending entry links).
- **Changes:** nothing directly (message-only); on failure, writes `LmsError`.
- **Status as of 2026-09-09: SMS sending disabled (`SMS_ENABLED = false`)** — Twilio was deliberately discontinued by the club (cost). The function still computes who needs reminding and logs **one** clear `LmsError` summary instead of one failure per recipient. To restore: flip `SMS_ENABLED` back to `true` and confirm Twilio is funded.
- **Approval required:** no (was fully automatic before disabling).
- **Failure behaviour:** logs to `LmsError`, doesn't block anything else.

### finaliseMatch
- **Purpose:** converts `MatchTimelineEvent` records into official `PlayerMatchStat` rows and populates `MatchReport.goalscorers/assists/cards`.
- **Reads:** `MatchTimelineEvent`, `TeamSheetPosition`.
- **Changes:** deletes+recreates `PlayerMatchStat` (idempotent, `$set` not `$inc` — genuinely safe to re-run), updates `MatchReport`.
- **Trigger:** now automatic, called by `LiveMatchCentre.jsx`'s `handleFullTime` as of the 2026-09-09 fix. Also has its own manual "Re-finalise Match" button for corrections.
- **Approval required:** no — but the result should always be visually reviewed.
- **Failure behaviour:** shows an error banner in the UI (`finaliseResult?.error`); doesn't corrupt existing data if it fails partway (idempotent design).

### recalculatePlayerStats
- **Purpose:** global recompute of `Player.season_*` cached fields from all `PlayerMatchStat` rows — the documented "source of truth: PlayerMatchStat" recovery tool.
- **Changes:** `Player.season_*` fields, via `$set`.
- **When to use:** if season totals look wrong anywhere, run this rather than hand-editing `Player`.

### chairmanDigest
- **Purpose:** weekly summary (availability, outstanding fees, latest news) via SMS.
- **Status:** written to look automatic but currently gates on `role === 'admin'` — effectively manual-trigger only. *Needs Paul's confirmation whether this should actually run on a schedule.*

### autoPlayerOfMonth
- **Purpose:** runs on the 1st of each month (per code comment — schedule not independently verified), auto-selects/announces Player of the Month.

### stripeWebhook
- **Purpose:** the only trustworthy confirmation a Stripe payment succeeded — flips `Payment.status` to `paid`. Never treat a client-side "success" screen as proof; this webhook is the source of truth.

### postToFacebook
- **Purpose:** real, connected Graph API post to the "Grindon Broadway 040s" Facebook page.
- **Approval required:** should always be treated as a real, visible publish action — confirm content before triggering.

## Still dependent on the discontinued Twilio account (not yet fixed, Known Issues #21)

`fridayAvailChase`, `thursdayNoResponseChase`, `sendAvailabilityReminder`/`sendAvailabilitySms`, `matchday24hrReminder`, `matchdayMorningSms`, `noonAwaySms`, `motmSubsSms`, `sendChairmanMatchPacket`, other `Lms*Sms` broadcast functions. **Decision pending:** replace with push notifications (OneSignal, already wired and free) / WhatsApp-ready text generation, or apply the same blanket-disable pattern used for `sendWeeklyPickSms`. Until decided, assume these are silently not reaching anyone.

## Real, working integrations (not Twilio-dependent)

- **OneSignal push** (`sendPushNotification`) — real, free, but only reaches players who've enabled push.
- **Stripe** — real checkout + webhook confirmation for LMS entries/buybacks and match fees.
- **API-Football** (`lmsGetFixtures`) — real fixture data for LMS picks.
- **Open-Meteo** (`fetchFixtureWeather`) — real, free, no key needed.
- **Facebook Graph API** (`postToFacebook`) — real, connected.

## Explicitly NOT real automation (common misconception worth documenting)

**"WhatsApp" features across the app are not automated.** Every one generates message text and opens a `wa.me` link or the WhatsApp share sheet — a human must tap send every time. There is no WhatsApp Business API integration. If true WhatsApp automation is ever wanted, that's a separate, bigger (and likely paid) integration decision — not a quick fix.

## Standing scheduled Routine (Claude-side, not a Base44 function)

**"TeamReady Weekly Admin Check-in"** (trigger id `trig_01FBZHf6in1d7rkgX1yA3NKm`) — Sundays 08:00 UTC, fires into this persistent Claude Code session, prompts Paul for match admin info and acts on the reply (FA team sheet, league table update). Created 2026-09-09. This is a Claude Code Remote Routine, not a Base44 backend function — disable/edit via `update_trigger`/`delete_trigger` tools, not via the Base44 app.

## Safety principle for all automations

Every automation that could plausibly run twice must be safe to run twice — no duplicate messages, statistics, payments, or records. `finaliseMatch`/`recalculatePlayerStats` already meet this (idempotent `$set` design). `sendWeeklyPickSms`'s guest/player SMS loop deduplicates by phone number within a single run but has not been checked for safety if the whole function is invoked twice in a row — *needs verification* before relying on that. A failed automation should be visible (via `LmsError` or similar), never silently disappear.
