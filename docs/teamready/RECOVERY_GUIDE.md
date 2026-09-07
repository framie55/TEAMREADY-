# Recovery Guide

**What's verified below is limited to what's been directly observed using Claude's actual tools this session. Anything wider (Base44's own infrastructure-level backup policy, disaster recovery beyond what a checkpoint restores) has not been confirmed with Base44 support and is marked accordingly — do not claim backups exist beyond what's stated here.**

## Code rollback: Base44 checkpoints (verified, used repeatedly this session)

`mcp__Base44__create_checkpoint` saves the app's current code state and returns a `checkpoint_id` plus the underlying `git_commit_hash` in Base44's own S3-backed git remote (`s3://base44-app-repositories/6a359ec6743bebc341bd156f`). This is the real rollback unit for code changes.

- **Every code change made this session was checkpointed immediately after a verified build**, both before and after, so either state can be restored. Recent examples: `6a9e8a34cd6c82d0ee2d54f4`→`6a9eba00704d546ece83d190` (Twilio SMS fix), `6a9ebe223850f24a7163c776`→`6a9f08fe076753356d30198d` (Matchday Studio + auto-finalise fix), `6a9f171a25bd400824ae03d3` (FA report generator).
- **How to restore:** *needs confirmation of the exact UI/tool step* — checkpoints are created via the MCP tool; restoring one has not been exercised this session. Confirm the mechanism (likely the Base44 dashboard's checkpoint/version history view) before relying on it under pressure.
- **Who can perform a restore:** *Needs Paul's confirmation* — likely anyone with admin access to the Base44 dashboard, not verified.
- 788+ commits exist in the underlying git history as of 2026-09-07, ~85% generic ("File changes"/"External agent changes") — the commit history itself is not a reliable audit trail (Known Issues #14), even though it is a technically complete rollback chain.

## Database rollback

**No equivalent checkpoint/rollback mechanism has been found for data** (player records, fixtures, payments, etc.) — only code is checkpointed. Data changes made via `create_entities`/`update_entities` are immediate and not automatically versioned anywhere Claude has found. *Needs investigation*: whether Base44's platform has any database-level backup/point-in-time-restore, separate from code checkpoints. Until confirmed, treat every data write as effectively permanent — this is why the golden rule (verify before writing) matters more than "we can just roll it back."

## What would be lost after a failure

- **If the Base44 sandbox/app itself failed:** code is safe back to the last checkpoint; any data written after the last... there is no data checkpoint, so this framing doesn't quite apply — data loss risk depends entirely on Base44's own platform-level database durability, which hasn't been verified.
- **If a Claude Code session were lost mid-task:** any code edit not yet checkpointed could be in an inconsistent state — this is why the standing practice is checkpoint-before and checkpoint-after every change, never leaving a half-applied edit unchecked.
- **This git repo (`TEAMREADY-`)** only ever holds documentation, never the application code — losing it would not affect the running app, only the local mirror of `docs/teamready/`, `CLAUDE.md`, and the older `docs/TEAMREADY_*.md` files. The canonical copies live in the Base44 app itself.

## No delete tool for data (relevant to recovery too)

Claude's current toolset has no delete capability for database records — only create/update/query. This is actually a mild safety feature (nothing gets permanently removed by an automated slip), but it also means the "known empty duplicate" player records (Known Issues #4) sit there until manually removed via the app's own UI. See `PLAYER_IDENTITY_REGISTER.md`.

## Practical rollback checklist (what to actually do)

1. Note the checkpoint ID immediately before any change (already standing practice).
2. Make the smallest possible change.
3. Run `npm run build` (and, for anything touching a specific file, a targeted `eslint` pass) before checkpointing the "after" state.
4. If something goes wrong: the two checkpoint IDs (before/after) are logged in `CHANGELOG.md` for every change made — use them to identify exactly what to revert to.
5. For anything involving real match/financial data: there is no undo beyond careful verification before writing. Treat every `create_entities`/`update_entities` call on real data with the same care as a checkpoint-worthy code change, because there isn't a database checkpoint to fall back on.

## Outstanding questions for Paul / Base44 support

- Does Base44 retain any database backups or point-in-time restore, separate from code checkpoints?
- What's the actual UI process to restore a named checkpoint, and who has permission to do it?
- Is there a faster way to hard-delete a genuinely empty, unreferenced record (like the James Dickinson duplicate) than the app's own Squad page, for future one-off cleanups?
