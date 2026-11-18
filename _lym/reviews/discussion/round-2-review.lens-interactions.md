```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions in the record that act on the same state, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D3 pick × D7 switch, both writing the counter: held. D7 writes it as D3's step 4 does.
- D3 pick × D2 prune and replace: held.
- D2 retired entry reads as none × D7 `set` refusing a retired account: held.
- D1 "fewer than two positive accounts" × D7 switch × D4 beams-pin skip: held. No candidate, so the pin stands.
- D4 beams-pin skip × D7 beams switch × D5 `blocked_until`: **broken** (finding 2).
- D4 hub pin never skipped × D7 hub pin changes nothing × D5 `blocked_until` on the hub account: held.
- D7 "already resolves elsewhere, no new switch" × the concurrent-sessions criterion: held.
- D6 copy in the resume check × D4 launched-account record in the run directory: **broken** (finding 1).
- D6 check resolution × D4/D7 launch resolution, which resolve separately: **broken** (finding 3).
- D6 refused-resume fallback × D7 relaunch and API-error watch clearing: held.
- D6 refused-resume fallback × D7 orch restart and fresh-start capabilities: held, apart from finding 1.
- D8 live readers on the launched directory × D7 relaunch rewriting the record × D6 copying with `<session>/`: held.
- D8 known directories × D1 load failure (readers degrade, launch fails): held.
- D4 scrub × D8 outside-hub reading of the process's own `CLAUDE_CONFIG_DIR`: held.
- D5 OpenLaunch settings with no creel flag directory × D1/D8 machine layer needing `Board` and `Anchor`: **broken** (finding 4).
- D9 prime never picked × D2 prune × D7 switch giving the prime an entry: held.
- D1 pin change × D3 "pinned worktrees are never picked": **broken** (finding 5).
- Acceptance criteria D1–D9 against their decisions: held, except the D5 OpenLaunch criterion (finding 4) and the D6 copy-record criterion (finding 1).
- Constraints (seam, cwd-resolution, lym-files, tolerant-records, cli-cobra) against D2–D8: held.

**Findings**

1. **MEDIUM, design.** `_lym/discussion/decision-record.md:170`
   - Summary: D6 says the resume check records a copy it made beside the run directory's launched-account record (D4). On a normal resume, though, the check runs before the run directory exists. The check therefore has nowhere to record the copy. D6's refused-resume fallback depends on that record, so it never fires there: neither the start-path resume nor the orch's `resumeOrch` falls back.
   - Evidence: `internal/shuttleengine/run.go:589` runs `checkResume` inside `start`, before `launch` creates the run directory at `run.go:645` and calls `Prepare` at `run.go:650`. `CheckResume(sessionID, workdir)` takes no run directory (`internal/shuttleengine/engine.go:357`). `resumeOrch` reaches the check through `StartSession` → `start` (`internal/orchcli/start.go:808`, `run.go:545`).
   - Only D7's relaunch, where the check runs after `Prepare`, has the directory.
   - Fix: D6 should say how the copy fact reaches the run. For example, the check returns it, shuttle carries it into the run record, and `Prepare` or `launch` writes it.

2. **LOW, consistency.** `_lym/discussion/decision-record.md:194-199`
   - Summary: For a beams pin, D7 depends on D5 having written the pinned account's `blocked_until` before the switch reads D4's resolution. The record does not order the two writes.
   - If the switch runs first, D4 still resolves to the pin. Bullet 3 writes an entry, but "the worktree then resolves to another account" is false, so no relaunch follows. The run falls to retryAPIError's "as today" handling, and its entry is already switched.
   - The same happens when the `blocked_until` write fails.
   - Evidence: D5 at `decision-record.md:145-146` and D7 at `decision-record.md:186-199`. Both act on the same limited turn end in `Wait` (`internal/shuttleengine/wait.go:347`, `wait.go:418`).
   - Fix: D7 should state that the limit's `blocked_until` is recorded under the account file's lock before the switch reads the resolution.

3. **LOW, design.** `_lym/discussion/decision-record.md:164-166, 204-206`
   - Summary: The resume check resolves the account by D4 separately from the `Prepare` that builds the launch. A `set` or a sibling's switch between the two can copy the transcript into account A and launch the session under account B, and the resume then finds no conversation.
   - This applies to the start path (check, then `Prepare`, at `run.go:589` and `run.go:650`) and to D7's relaunch (step 2 `Prepare`, then step 3 check).
   - Fix: in D7's relaunch, the check reads the launched-account record `Prepare` just wrote. On the start path, the account the check resolved is passed on to `Prepare`.

4. **LOW, consistency.** `_lym/discussion/decision-record.md:135, 328`
   - Summary: D5 has `OpenLaunch` write settings with the status line "also with no creel flag directory", and its criterion tests such a session. That configuration can't occur.
   - The creel flag directory is empty exactly when the engine's `dirs.Board` is empty (`internal/shuttleengine/claudeengine/claudeengine.go:319-321`). The machine layer stacks only when both `Board` and `Anchor` are told (`pattern/PATTERN-config-layers.md:47`), so such an engine loads no accounts and none is ever listed.
   - Fix: drop the clause and that part of the criterion.

5. **LOW, design.** `_lym/discussion/decision-record.md:82`, read with D1 at `:48-50`
   - Summary: D3 never picks for a worktree a pin covers, and picks only when a worktree is made. If the operator later removes or changes a hub pin or beams pin, every worktree made under it has no entry. Those worktrees keep running as today, on the default or `env` directory, and are never weighed until an adopt or a `set`.
   - The record says nothing about that transition. The board asks for a pick "every time a worktree gets work on a machine where it has no account yet".
   - Fix: state it as intended behaviour, or pick at the first engine-built launch of an unpinned worktree with no entry.

Tool calls: none.
