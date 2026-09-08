# Decisions and Corrections

Every correction Paul has given overrides an earlier assumption, generated answer, or unverified record. Logged here permanently, distinct from the general `CHANGELOG.md`, so a future session never re-makes the same wrong assumption.

## 2026-09-09 — Twilio/SMS was a deliberate business decision, not a bug

**Earlier assumption (mine, wrong framing):** treated the `sendWeeklyPickSms` Twilio authentication failures as a critical technical bug requiring a fix (e.g. restoring credentials).
**Correction (Paul):** Twilio was deliberately discontinued — it was running out of credit too fast against the volume of messages needed. Any function still trying to send SMS is expected to fail; the fix is to stop it trying, not to reconnect it.
**Effect:** `SMS_ENABLED = false` guard added to `sendWeeklyPickSms`. Known Issues #1 status corrected from "Critical bug" to "Fixed — was a deliberate decision." Same correction applies to the ~14 other still-Twilio-dependent functions (Known Issues #21) — a channel-replacement decision is pending, not a "restore Twilio" fix.
**Database records affected:** none — this was a code-behaviour correction, not a data correction.

## 2026-09-09 — Matchday Studio was never used for live recording

**Earlier assumption (mine):** Matchday Studio might be a legitimate alternative recording surface, just riskier for being public.
**Correction (Paul):** confirmed "No" — Live Match Centre is the only screen ever used to run a match. Matchday Studio's recording capability had zero legitimate use.
**Effect:** the entire recording control surface (Kick Off/Goal/Card/Sub/Full-Time) was removed from `MatchdayStudio.jsx`, leaving it purely read-only. Known Issues #22.
**Database records affected:** none — this was a code change, no historical data touched.

## 2026-09-07 to 09 — Player identity confirmations

**Correction/confirmation (Paul), each overriding "flag it, don't guess":**
- "Jiffy is James Dickinson" — confirmed, matches the live data exactly.
- "There are two different Paul Griffiths — one goalkeeper, one outfield (Griffo)" — confirmed, matches live data exactly (Griffo `...092b`, Griff `...5d26`).
- "Mark Dickinson is known as just Mark or Mark Dickinson" and is not the same person as Jiffy — confirmed, recorded in `PLAYER_IDENTITY_REGISTER.md`.
**Effect:** all three locked in as verified ground truth. No merges or edits were needed — the underlying data already reflected these correctly; the confirmations closed out Known Issues #4/#5 investigation.
**Database records affected:** none needed correcting — investigation only.

## 2026-09-07 — Kevin Berry's LMS buyback: manager's theory confirmed by direct verification

**Paul's stated theory:** the £10 entry that caused the "20 paid / 21 picks" mismatch was "somebody who joined but never actually completed the payment and never actually picked a team... never materialised."
**Verification (me):** queried `Payment` (none exists for that entry), `LmsEntry` (`joined_at`/`terms_accepted_at` both null), `LmsPick` (`team_picked: "VOID — invalid entry"`, `pick_submitted_at: null`, set by Paul himself on 2026-09-03). Theory confirmed exactly.
**Effect:** Known Issues #2 marked Resolved. No correction needed to the theory — this is logged as an example of the golden rule working as intended (verify, don't just trust or dismiss a stated theory either way).

## 2026-09-09 — FA Team Sheet generator must include Man of the Match

**Instruction (Paul):** "he need the mom also" — the FA email recipient (Mick Greenwell) needs MOTM included, not just lineup/goals/cards.
**Effect:** the generator reads `form.motm_player_id` (the manager's own pick from the existing Match Report page selector) first, falling back to the `MotmVote` player-vote tally with vote counts if no manager pick has been made yet — never inventing a MOTM.

## 2026-09-09 — FA registered squad list cross-check: two spelling fixes, one duplicate resolved, one Claude documentation error corrected

**Source:** Paul sent a screenshot of the FA's official registered squad list — the authoritative source for player name spelling.

- **"Paul Diamond" confirmed correct** (database had "Paul Dimond") — fixed. Known Issues #24.
- **"James Shickle" confirmed correct** (database had "James Shikle", missing a letter) — fixed. Known Issues #27.
- **Stephen Halliday duplicate resolved**: Paul confirmed "Stephen Halliday is Buddy and only played 2nd half vs Darlington RA a few weeks ago but injured atm" — matched exactly to the real record's actual match history. The empty duplicate (created 6 Sept, zero stats) was archived; the real record's spelling was corrected from "Haliday" to "Halliday" and his injury status logged. Known Issues #28.
- **Correction to Claude's own earlier documentation, not a database error:** `APP_OVERVIEW.md` had wrongly stated "Manager: Paul Frame, known as 'Corby.'" **Paul's correction:** "Framie is me Paul Frame, Corby is Anthony Richardson, we are both managers. I play also, Corby doesn't." The database was correct throughout (Paul Frame = Framie, Anthony Richardson = Corby, both are `Player` records with the right nicknames) — the error was entirely in Claude's inference from the matchday programme, now fixed. Known Issues #29.
- **"Paul Griffith" question — closed by Paul, 2026-09-09:** raised as possibly a third person given the FA list shows both spellings separately registered. **Paul's instruction: "No just leave them as they are... one goalkeeper, one outfield player, Griff Keeper, Griffo the outfield player."** Confirms the original two-Griffiths setup is correct and complete; the archived "Paul Griffith" record stays archived, untouched, no further investigation. Known Issues #5 closed.

## 2026-09-07 — Public website Phase 1: security blocker found, sponsor assets supplied, badge name and founding year confirmed

**Phase 1 recon finding (mine, not yet acted on pending Paul's go-ahead):** the existing `/public/*` and `/supporter/*` pages (already live, zero login required) fetch full `Player` records client-side with no field whitelisting; since no entity has row-level security, private fields (`login_pin`, `otp_code`, `invite_token`, `phone`, `email`, `date_of_birth`, emergency contacts) are exposed in the browser network response today, independent of any new build work. Reported to Paul in full; awaiting his go-ahead to patch and to confirm the "extend existing pages, don't duplicate" approach.

**Sponsor logos and club badge supplied by Paul:** Amber's Legacy, GKB Financial Planning, SafeSwitch Solutions, UCS Renewables (three logo variants), David Graham Roofing, DKJ Joinery, and the club badge. No image-upload tool is available to Claude — Base44's own Sponsor/Club admin screens are the only path for the actual files; Claude can create/update the surrounding data records and verify once Paul uploads.

**Badge name and founding year — confirmed by Paul, correcting a real ambiguity found across three different badge assets:** a kit mockup photo showed "Grindon Broadway Over 40s Football Club, EST 2021" — different name AND different year from the confirmed badge image ("Grindon Board Inn Over 40s, EST. 1995"). Paul's instruction: **"Defo 1995 est"** and confirmed "Grindon Board Inn" (not "Broadway") is the name for the official badge. The "EST 2021" kit mockup is wrong and must not be used publicly. `APP_OVERVIEW.md` already had 1995 correct; no change needed there. Known Issues #30.

**Sponsor categorisation — Claude's own error caught and corrected before writing bad data:** initially asked Paul to pick a sponsorship category for UCS Renewables using options that mixed up two different database fields (`sponsorship_type` enum vs. the separate `priority` enum, which has "main"/"standard" — "main" isn't a valid `sponsorship_type` value). Caught before any write; re-asked correctly. Paul confirmed: UCS Renewables = type `kit` (logo confirmed on training tops), priority `main`. DKJ Joinery = `pitch`. David Graham Roofing = `pitch`. SafeSwitch Solutions = `general`.

**David Graham Roofing — confirmed same person as the goalkeeper:** Paul confirmed ("yes to all") this sponsor is the same David Graham who plays in goal (`Davie Graham`, player id `6a5bd695e42bb914bb55bb2b`) — not linked as a database relationship, just noted here since it's a real fact about a real person, not to be assumed again.

**Database records created:** four new `Sponsor` records — UCS Renewables (`6a9f3299075e24e0b122223b`), DKJ Joinery (`6a9f3299075e24e0b122223c`), David Graham Roofing (`6a9f3299075e24e0b122223d`), SafeSwitch Solutions (`6a9f3299075e24e0b122223e`) — all with `logo_url: null`, `website_url: null` pending Paul supplying/uploading them.

## 2026-09-07 (continued) — Patched the live public/supporter Player data exposure

**Instruction (Paul):** "Ok" — approved proceeding on the security patch (and separately, to hold off inferring approval for the full new-pages build from the same one-word reply — that was Claude's own judgement call, flagged back to Paul rather than assumed).
**Fix:** created `getPublicPlayerData` backend function (same safe-aggregator pattern as the existing `getGuestDashboardData`), explicitly whitelisting only public-safe `Player` fields (name, nickname, position, shirt number, photo, season stats). Replaced the direct `base44.entities.Player.filter(...)` call in all 5 affected pages (`PublicSquad.jsx`, `SupporterSquad.jsx`, `SupporterStats.jsx`, `SupporterMatchCentre.jsx`, `SupporterHome.jsx`) with a call to this function.
**Verified:** `npm run build` clean; targeted `eslint` on all 6 changed files clean except pre-existing, unrelated unused-import errors already documented in Known Issues #18.
**Effect:** Known Issues #31 (new row, Fixed). Known Issues #3 (no row-level security anywhere in the app) remains open — this fix closes the specific live exposure, not the underlying structural gap.
**Database records affected:** none — code-only change.

## 2026-09-08 — Club badge confirmed correct as-is; only the stale database field needed fixing

**Instruction (Paul):** "Yes the badge is correct" — confirming the badge already live across the app (the hardcoded `grindonboardinnbadge.png` file, "Grindon Board Inn," EST. 1995) matches the real badge image he supplied, so no visual/code change was needed.
**Finding (mine, verified before acting):** the ~30 hardcoded code references were already correct; the only actual inconsistency was the separate `Club.badge_url` field pointing at a different, unused file, and `Club.name` being blank.
**Effect:** updated `Club.badge_url` to the confirmed-correct file, and set `Club.name` to "Grindon Board Inn Over 40s FC." Known Issues #30 closed.
**Database records affected:** `Club` `6a38094e4d090aec62268573` (`badge_url`, `name`).
**Also confirmed (verbally, "Use any player and yes you can take them from the app no problem"):** the player-card graphics Paul sent are a style reference, not finished assets to use as-is — Claude builds the card component itself, pulling real photos/stats from whichever `Player` records make sense, rather than needing one hand-supplied per player.

## 2026-09-08 — Domain registered: grindonboardinn40s.co.uk

Paul registered `grindonboardinn40s.co.uk` via IONOS (plus `.com` and `.info` in the same bundle), confirmed by screenshot — status "Domain not in use," expiring 08/09/2027. **Not connected to anything live** — per Paul's own standing instruction, no domain gets connected to the app until he explicitly approves launch. Purchasing/registrar management is Paul's own action; Claude has no registrar access and didn't do anything here beyond logging it. One housekeeping item on Paul's side, not Claude's: the `.info` domain shows "Confirmation of contact details required" in IONOS — needs him to resend/complete the verification email, unrelated to the website build.

## 2026-09-08 (continued) — Correct club contact email confirmed

**Background:** three different, conflicting email addresses had turned up across the app during the website build: `grindonbroadway@email.com` (hardcoded in `PublicContact.jsx`), `club@gridonboardinn.com` (hardcoded in `SupporterSponsors.jsx`, also misspelled "gridon"), and `grindonbroadwayover40s@gmail.com` (the actual Base44 account owner's email on the `Club` record's `created_by` field). Flagged as unresolved in earlier changelog entries rather than guessed at.
**Instruction (Paul):** "grindonbroadwayover40s@gmail.com that's the correct email."
**Effect:** `PublicContact.jsx` updated to use the confirmed address. The other two wrong addresses were already removed from the sponsor pages earlier today (replaced with links to the new "Become a Sponsor" page) — this closes out the very last stray reference.
**Database records affected:** none — code-only correction.

## 2026-09-08 (continued) — Investigated a genuine way to upload images directly; found the real mechanism, but couldn't safely use it, and removed the resulting risk

**Paul's question:** why haven't the sponsor logos he's sent multiple times been connected — can Claude just use the ones already sent or already on the app?

**What was found, for real:** the app does have a genuine file-upload API (`base44.integrations.Core.UploadFile`, used throughout the app's own admin screens — e.g. the Sponsor edit modal). A backend function could call this with service-role access. Built one (`uploadAssetAndLink`) to prove it.

**Why it couldn't be completed:** actually moving the bytes of the images Paul sent in chat into that function required either (a) direct network access from Claude's own environment to Base44's servers — blocked by Claude's own organisation's outbound proxy policy, confirmed by direct test; or (b) relaying the image data through Claude's own conversation context as base64 text — technically possible but prohibitively expensive (a single small logo consumed ~70,000+ tokens of context; five images would have been unworkable). A third route (Artifact asset storage as a relay) turned out to require a capability not enabled for this account. Checked whether the logos might already be sitting in Base44 under some other name (`ProgrammeAsset` sponsor_logo entries) — confirmed empty, they aren't.

**Security correction, caught before it caused harm:** the `uploadAssetAndLink` function, as written, was unauthenticated and could overwrite any field on any entity via `asServiceRole` — a real vulnerability if left live and undiscovered, regardless of whether it was ever wired up to anything. Disabled it outright (returns 410) rather than leave working-but-unused attack surface sitting in the codebase, undoing the point of the security work done earlier this session.

**Practical answer given to Paul:** the fastest real path is the one the app already provides — open the Sponsor record in the app's own admin screen (Sponsor edit modal) and use its existing logo upload field directly; it uses the exact same underlying API. Claude verifies the result afterward.

## 2026-09-08 (continued) — Sponsor logos linked via git-hosted files; UCS Renewables and Amber's Legacy confirmed as the "main" sponsors

Paul re-sent the 5 logo files, this time as local file attachments rather than inline chat images — which let Claude commit them straight into this repo (`assets/sponsor-logos/`) and link each Sponsor's `logo_url` to a `raw.githubusercontent.com` URL, without the network/cost/capability problems hit in the previous attempt. All 5 confirmed live: SafeSwitch Solutions, DKJ Joinery, David Graham Roofing, UCS Renewables, GKB Financial Planning.

Paul then sent UCS Renewables' official logo separately and said "these are our main sponsors... we've just received our sponsorship money from them so we need to have these plastered all over the website... predominant on every page." Checked the Sponsor entity's own `priority` field rather than guessing which records this applied to: only **UCS Renewables** and **Amber's Legacy** are flagged `priority: 'main'` — everyone else (SafeSwitch, DKJ, David Graham Roofing, GKB, Board Inn East Herrington) is `standard`. Built the sitewide placement (`MainSponsorStrip.jsx`) to follow that existing flag rather than hardcoding sponsor names, so it stays correct automatically if Paul changes a sponsor's priority in future.

**Caveat given to Paul:** these logos are hosted from git, not Base44's own media storage like every other image in the app — functional and stable, but not the permanent "native" home. Recommended he re-upload the same files via the app's own Sponsor edit screen when convenient.

## 2026-09-08 (continued) — Amber's Legacy confirmed as a charity tribute, not a commercial sponsor

Paul confirmed the reasoning behind keeping Amber's Legacy out of the new commercial "main sponsor" hierarchy was correct, and gave the real context: Amber's Legacy is a cervical cancer charity set up in memory of the daughter of Daz Cliff (Darren Cliff), a player at the club. The club promotes it to support Daz and the cause — not for any financial or promotional benefit. Recorded permanently in `CLUB_RULES.md` under a new "Sponsors vs. charity partners" section so this is never mistaken for a commercial arrangement, never reported alongside paid sponsor revenue, and never moved out of its own dedicated spot on the About page.

## 2026-09-08 (continued) — Most sponsors are in-kind and player-owned, not cash

Paul explained the real nature of each sponsor, correcting an unstated assumption that they were arms-length businesses paying the club cash:

- **SafeSwitch Solutions** is Paul's own business — he personally funds the matchday programme's production, printing and team sweatshirts rather than paying the club a fee.
- **GKB Financial Planning** is owned by player Gareth Brazier ("Gash") — he buys the club's kit (polo shirts), no cash sponsorship. Corrected `sponsorship_type` from `general` to `kit` to match.
- **David Graham Roofing** is owned by player David Graham — paying for a pitchside banner.
- **DKJ Joinery** is owned by Chris Johnson, who is **not yet a player** — he's signing for the club in December once he turns 40 (O40s league eligibility). He's paying for a pitchside banner. **No `Player` record created** — he isn't a player yet, and this is not a request to add him. **His future signing was not published anywhere public** — a sponsor thank-you isn't the venue for a transfer announcement.

Verified each named player against real records before writing anything (golden rule): Gareth Brazier and David Graham both confirmed as existing active `Player` records; Chris Johnson confirmed absent, as expected. Linked `contributor_id` on GKB and David Graham Roofing to the correct player IDs. Rewrote all four `thank_you_message` fields to describe what was actually given, replacing generic placeholder text. Full detail in `CLUB_RULES.md`.

Immediately afterward, Paul added: **Paul Mooney** (an active player, confirmed) also owns **The FNF Method**, and will be paying for a pitchside banner like DKJ/David Graham Roofing. The FNF Method already existed as a hardcoded showcase card on `/public/sponsors` (with real Instagram/phone contact details) but had no actual `Sponsor` database record, so it wasn't flowing into the programme, ticker, or dashboard like a real sponsor. Created one (`contributor_id` linked to Paul Mooney), matching the `pitch` sponsorship type used for the other banner sponsors. No logo file has been supplied for it yet, so it won't appear in the site-wide sponsor strip until one is.

## 2026-09-08 (continued) — Three domains in play; public website and staff app to be kept on separate addresses

Working through getting the public website live surfaced three different domains, confirmed one at a time rather than assumed:

1. **grindonbroadwayover40s.org** — bought "this morning," per Paul. Confirmed via DNS: still sitting untouched on IONOS's default parking page, not connected to anything.
2. **grindonbroadwayover40s.com** — already connected as a Base44 custom domain from before this session. After Paul published the app in the Base44 dashboard, this domain went live and now correctly serves the app (confirmed via HTTP checks from the Base44 sandbox: both `/` and `/public` return 200 with the real app shell, not an error or parking page).
3. **grindonboardinnover40s.co.uk** — registered today (confirmed via Paul's IONOS registration confirmation email, customer number 316388145). Paul explicitly confirmed: **this is the domain to use for the public website.**

Paul's stated requirement: **the public website and the staff/player app must be on separate addresses** — the website is for the general public, the app is for players and staff. Plan agreed: connect `grindonboardinnover40s.co.uk` in Base44 as an additional custom domain for the public site; the existing address stays as the staff/player app's home. Domain connection itself (Base44 dashboard + IONOS DNS records) is Paul's own action — Claude has no tool access to either dashboard.

**Resolved:** Paul confirmed keep the existing `/` → `/public` redirect everywhere (option A) — "I don't want the public to have a login, it's a public site." Acted on the stated reasoning, not just the letter of the choice: removed the "Team Login" button from the public site's header entirely (`PublicWebLayout.jsx`) so no login prompt is visible anywhere on the public-facing pages. `/login` itself is untouched and still reachable directly by URL — this only removes the visible link from public pages. Build verified clean.
**Checkpoint:** `6a9fda9b410ca379978c5b14`.

## 2026-09-08 (continued) — The earlier `/` → `/public` redirect fix was never actually verified, and had a real bug

Once `grindonboardinnover40s.co.uk` finished DNS/SSL propagation, Paul opened it in a real browser and got the Manager/Player **login screen** at the bare root — not the public homepage. This exposed that the original launch-blocker fix (logged earlier as "Fixed" in `KNOWN_ISSUES.md` #34) had only ever been checked via `curl` from the Base44 sandbox, which can only see HTTP status codes — it cannot observe client-side React routing behaviour. The fix looked plausible and was reported as working without ever being confirmed in an actual browser. That was a real gap in how it was tested, not just an edge case.

**Root cause found in `src/lib/AuthContext.jsx`:** `authError.type` is only ever set to `'auth_required'` when the app's public-settings check returns an HTTP 403 — which only happens when the app's Base44 "App Visibility" setting is **Private**. This app's visibility is set to **Public** (confirmed from the Base44 dashboard screenshot Paul sent), so for an anonymous visitor that check succeeds with a 200, `authError` stays `null` the whole time, and the old redirect logic (nested inside `if (authError) { ... }`) never ran at all. Execution fell through to the normal `<Routes>`, where `/` sits behind `ProtectedRoute`, which client-side-redirected an unauthenticated visitor straight to `/login`.

**Fix:** moved the `/` → `/public` redirect in `src/App.jsx` out from under the `authError` check entirely, keying it directly on `isAuthenticated` (already provided by `AuthContext`) instead. This covers both cases correctly — an anonymous visitor on this Public-visibility app (no authError ever set) and, unchanged, an anonymous visitor on a hypothetically Private-visibility app (`authError.type === 'auth_required'`) — since in both cases `isAuthenticated` is `false`. A genuinely logged-in staff member still reaches their Dashboard at `/` exactly as before, since `isAuthenticated` is `true` for them.

**Tested:** `npm run build` clean. Root cause traced by reading the actual auth-check source rather than guessing.
**Checkpoint:** `6a9fe654ccf6a094673c8cd2`.
**Lesson recorded:** a curl/HTTP-status check on a React SPA proves the server responded — it proves nothing about which screen a real visitor sees. Client-side routing behaviour needs either a real browser check (via Paul) or an explicit test of the actual condition the code branches on, not just "did the page return 200."

## 2026-09-08 (continued) — Homepage hero uses an AI-generated squad image with real sponsor branding, by Paul's explicit and informed decision

Paul sent an image to use as the homepage hero — a polished, professional-looking squad walkout shot with the club badge, real GKB Financial Planning and Amber's Legacy branding on the shirts, taglines and CTA buttons baked in. Before using it, checked the file's embedded metadata rather than trusting how it looked: it carries a cryptographically signed C2PA content-provenance manifest from **OpenAI's image service**, confirming it's AI-generated — not a real photograph of real players.

Flagged this to Paul directly, twice, with the specific concern spelled out: the image places two real sponsors' branding (one of them, Amber's Legacy, a memorial charity — see the earlier entry above) on fabricated, non-existent players, without those sponsors having agreed to that use. Offered an alternative (same composition, no real branding on the shirts) as a lower-risk option.

**Paul's response, twice, explicit:** he wants the image used exactly as sent, sponsor logos and all — his reasoning being that the sponsorship itself is completely real (GKB and Amber's Legacy genuinely do sponsor the club), so the image is not making a false claim about who sponsors the club, only using a synthetic rendering technique instead of a camera. This is his call to make about his own club's marketing, and it was made with full knowledge that the image is AI-generated — that was the material fact Claude was responsible for surfacing, and it was surfaced clearly before this was actioned.

**Implemented:** `PublicHome.jsx` hero now displays this image directly (natural aspect ratio, not cropped) with two invisible clickable link overlays positioned over the image's own "View Fixtures" and "Join Our Journey" buttons, since the image already contains that text — no duplicate heading/CTA text was added on top.

**For any future session:** this hero image is confirmed AI-generated, not a real photo — do not describe it to Paul or anyone else as a genuine photograph, and do not re-raise this as an unresolved concern; it was raised, heard, and explicitly decided. If Paul supplies a real matchday or squad photo later, swapping it into this same hero slot is a one-line change (see the image `src` in `PublicHome.jsx`) — worth offering, not worth insisting on.

## Pending — not yet confirmed by Paul

- **Programme "Results This Season" mix-up root cause** (Known Issues #25) — whether this is a recurring generation bug or a one-off manual slip.
- **Whether the full player-identity register needs a systematic duplicate audit** across all ~90+ records — the FA registered-squad cross-check (2026-09-09) covered the 37 active players and found 2 real spelling errors plus 1 more duplicate, so a similar pass may be worth doing for the remaining ~55 inactive/archived records too.

## How this file is maintained

Every time Paul corrects a name, identity, statistic, fixture, score, process, or app rule: (1) check whether it affects existing database records, (2) record it here, (3) update the relevant doc (`PLAYER_IDENTITY_REGISTER.md`, `KNOWN_ISSUES.md`, etc.), (4) add validation to prevent recurrence where practical, (5) never change historical data until it's confirmed which records are actually affected, (6) tell Paul explicitly whether any existing records need repairing.
