# TeamReady Weekly Workflow

_The club's real weekly routine, mapped onto the actual app screens/functions found during Phase 1 inspection. Automate the repetitive parts; always require a human decision before publishing, messaging people, or finalising uncertain football data._

| Step | Where it happens in the app | Automated today? |
|---|---|---|
| Confirm next fixture (venue, date, kick-off, opposition) | `Matches.jsx` → `Fixture` entity | Manual entry; no validation stops incomplete fixtures being saved — check by hand |
| Collect player availability | `Availability.jsx` (admin view), `PlayerAvailResponse.jsx` (player view) | Real automated SMS chase exists (`thursdayNoResponseChase`, `fridayAvailChase`) — confirm in Base44 dashboard that these are actually scheduled to fire |
| Identify missing replies | `Availability.jsx` counts (Available/Maybe/No response) | Automatic display; chasing is the function above |
| Prepare the available squad | `TeamSelection.jsx` | Manual, but reads live availability data — no re-typing needed |
| Prepare formation & substitutions | `TeamSelection.jsx` (draft), publish via `PublishModal` | Manual; draft/publish workflow exists so nothing goes out half-finished |
| Prepare matchday reminders & directions | Real automated SMS/push (`matchday24hrReminder`, `matchdayMorningPush`/`matchdayMorningSms`, `noonAwaySms`) | Automated — confirm schedule is live |
| Prepare team-talk information | `TeamTalk.jsx` | Manual |
| Record the score | `LiveMatchCentre.jsx` during/after the match | Manual entry, real-time |
| Record scorers, assists, appearances, awards | `LiveMatchCentre.jsx` live entry, **then press "Finalise Match"** | The Finalise step is what makes it official — see Data Rules doc for why this matters |
| Update verified statistics | Automatic once "Finalise Match" is pressed (`finaliseMatch` function rebuilds `PlayerMatchStat`) | **Do not skip pressing Finalise Match** |
| Prepare a draft match report | `MatchReport.jsx` (manual) or `AiMatchReport.jsx` (AI-drafted, for sharing only — not saved as the official report) | AI draft needs a human read-through before it's used anywhere as fact |
| Prepare the weekly programme content pack | `ProgrammeManager.jsx` | Pulls live fixture/player/sponsor/finance data automatically; still needs a human check before publishing |
| Update Last Man Standing | Admin panels under `src/components/lms/`, backend `lmsProcessResults` | Confirm each week whether this runs automatically or needs a manual click — not confirmed either way during inspection |
| Update club finances/subs | `Finances.jsx`, `MatchFeeTracker.jsx` | Stripe payments are automatically verified via webhook; cash payments are manually marked — both are legitimate, just don't treat a manual "cash received" mark as equivalent proof to a Stripe webhook confirmation |
| Produce WhatsApp-ready announcements | Various "share" buttons across the app | **Text is generated for you; sending is always a manual tap** — there is no automated WhatsApp sending today |

## Rule for all of the above

Automate the *preparation* and *chasing* steps. Never auto-publish programme content, never auto-send a WhatsApp/social announcement, and never let an uncertain football fact (a score, a scorer, an availability status) get treated as final without a human having looked at it once.
