```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state a decision introduces or changes, I traced the code paths that already run and say whether the decision survives each one.

**Paths traced, per decision**

- **D1 (PostToolUse block):**
  - Claim and acknowledge through `ClaimAtBoundary`: breaks. The claim always hands out the first message whatever its size, and a body may be up to 64 KiB, which the 10,000-character field cap does not hold (finding 1).
  - Hook output read by Claude Code: survives.
- **D2 (merged Stop command, block records nothing):**
  - `ParseEvents` → `EventStop`/`EventWaiting`, used by `shuttleengine/wait.go`, `run.go:897`, `activity.go:218`, `orchengine/watcher.go:199` and `websterengine/strand.go:75`: survives. None of them sees a Stop record for a blocked stop.
  - `ParseSessionSignals` stamp pairing (`signals.go:75`): survives only if the stamp line is also skipped on a block. The record is silent on the stamp (finding 3).
- **D2/D4 continuation count and the "woken" mark per claimable set:**
  - The creel daemon exits on every binary replacement (`server.go:47`, `doc.go:36`), and the next verb starts a fresh one.
  - The count breaks: the record does not say it outlives a restart, so the D2 bound breaks (finding 2).
  - The once-per-set ring breaks: a reconnected waiter can be rung again for a set it already woke. The record's restart bound names only D5's typed bell (part of finding 2).
- **D5 (shuttle-launched strands only):**
  - The `Delivery` seam's `Strand` (`seam.go`) carries no launch origin.
  - Reed state has `cmd`, `resumeCmd` and `sessionId` (`reedengine/state.go:26-28`), which can tell the two apart.
  - Survives, with a mapping the plan can find; no finding.
- **D5 and sessions launched before the deploy:**
  - Breaks partly. Such a session carries the old `PostToolUse` hook but no new `Stop` delivery.
  - The bell tells it to end the turn and not to run `lym creel read`, so the mail stays put until a tool call or a restart (finding 4).
- **D6 (no pane path):**
  - Old `reminded` records and notice reservations are folded tolerantly (`fold.go:231`, `doc.go:61`): survives.
  - Orch and other strands launched by shuttle get the hooks through `claudeengine.go:197-208` whenever a board directory exists: survives.
- **D9 (sentence on launch and resume lines):**
  - The resume line is stored in reed's `resumeCmd` when the strand is launched (`state.go:27`). Strands launched before the deploy keep the old one, which Open risks already names: survives.
- **D4 wake turn with no `UserPromptSubmit`:**
  - `wait.go` held-turn-end handling and the orch watcher read the woken turn's end as a turn end, as the record's bound says: survives.

**Findings**

1. **MEDIUM, design. A message over about 10,000 characters is acknowledged as read but reaches the session as a preview (D1, D6, Constraints "JSON output").**
   - The constraint says "D1's 8 KiB claim budget keeps each field under the cap". It doesn't: `ClaimAtBoundary` "always hands out the first" message whatever its size (`internal/creelengine/verbs.go:162`, loop at 188-198).
   - A body may be up to `maxBodyBytes = 64 << 10` (`internal/creelengine/pool.go:17`).
   - A post of 10–64 KiB therefore goes whole into `reason` and `systemMessage`, Claude Code replaces both with a preview, and creel acknowledges the message as read.
   - D6 removes the old route for long mail, the 4 KiB notice cap that sent a long message through `lym creel read` (`reminder.go:23`).
   - Result: "delivered in full" fails, with no state left to show what was lost.
2. **MEDIUM, design. A daemon restart resets creel's continuation count and its woken-set marks (D2 bound, D4 once-per-set wake, D5 restart bound).**
   - The creel daemon exits whenever its binary is replaced and the next verb starts a fresh one (`internal/creelengine/server.go:47-49`, `doc.go:36`).
   - The record does not say whether the count or the "woken" mark is kept across that restart.
   - If either lives in memory:
     - A strand at count 5 can take up to 6 more blocks after a deploy, past Claude Code's override at 8, which breaks "no override ever discards a block whose mail creel journaled read".
     - Every reconnected D4 waiter (D4 requires the reconnect) can be rung again for a set it already woke.
   - D5's restart bound names only the typed bell.
3. **LOW, design. The record does not say whether a blocked Stop also skips its stamp line (D2).**
   - The recording command writes a `lym_stamp` line before the payload (`settings.go:286-289`).
   - `ParseSessionSignals` pairs each payload with the nearest earlier untaken stamp, and holds back its consumed offset at a trailing run of stamp lines (`signals.go:75-76`).
   - "Appends no Stop record" does not say whether the stamp is skipped too. If the merged command keeps today's order (stamp first, then decide), every blocked stop leaves an orphan Stop stamp.
4. **LOW, design. D5's bell into a session launched before the deploy tells it not to read its mail, though no Stop delivery exists there (D5, Open risks third line).**
   - The bell says "creel hands the mail to you as the turn ends. Do not run lym creel read for it." (D4 bell text).
   - A pre-deploy session has no merged Stop delivery, so ending the turn delivers nothing.
   - The bell also steers it away from the one read path left to it, now that D6 removes the reminder.
   - Open risks names the bell but not that it strands the mail until the next tool call or a restart.

**Tool calls**

1. Grepped the Stop/`EventStop` readers, the flag-dir wiring and resume settings: the readers of D2's skipped record were not in context.
2. Read `signals.go` stamp handling: the stamp pairing that D2's skip meets was not in context.
3. Grepped reed for a launch-origin marker and the creel fold/journal for persisted state: needed for D5's strand filter and the D2/D4 restart state.
4. Read the fields of reed's `type Strand struct`: confirms which state can tell a shuttle launch from a hand-added strand.
5. Grepped creel doc and server for daemon restart: needed to know whether the daemon restarts on deploy.
