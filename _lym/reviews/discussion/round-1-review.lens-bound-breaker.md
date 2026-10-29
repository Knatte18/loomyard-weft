```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's claims and bounds and checked them against the Claude Code 2.1.312 binary and the creel code.

**Claims attacked**
- D1, additional context goes into the continuation list unless a mode check holds: **held**. The guard is `Ugt`, which returns true only when `Sr()` is true. The Stop runner's module imports `Sr,Ur,zae` from chunk-gsa29gat, where `Sr()` is `CLAUDE_CODE_SIMPLE || --bare`.
- Open risks bullet 1 and Q10, "the tier3 smoke run shows whether a lym session continues the turn": **broken**, because research settles it now (F1).
- D1, Claude Code's cap of 8 counts these continuations and creel's cap of 6 holds unchanged: **held**.
  - `blockingErrors.length>0` raises `stopHookBlockingCount` against `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP ?? 8`.
  - Claude Code resets its own count to 0 on every tool turn (`next_turn` spreads `pi`, which sets `stopHookBlockingCount:0`). So it resets at least as often as creel does.
- D1, the operator reads the mail once as `Stop hook feedback: <lines>`: **broken** for longer claims (F2).
- D1, `Stop hook error:` lines come only from blocking errors: **held**. The `to` list is fed only from `we.blockingError`.
- D2 claims: **not verified**. I found no rendering string for additional context, but I found no counterexample either.
- D3, the pause window is the same one a boundary claim already has: **held**. `ClaimAtBoundary` reads the pause outside `s.do` (`internal/creelengine/verbs.go`).
- D3, a ring whose claim comes back empty does not ring and the waiter stays registered: **held**. `ringHook` already requires the claimable set to match, and under a pause or hold `wakeHookStrand` returns false with `waiting` true.
- D3, exit 0 is no wake and its stderr is dropped: **held**. In `_tt`, only `Ae.code===2` enqueues the wake.
- D3, the rewake is enqueued with priority `next`: **held**. `Bin({...priority:"next",stopHookActive:!0})`.
- D3, an older daemon's ring with no lines makes the bell follow: **held**. The bell fires after `ringAnswerWindow`, 30 s (`internal/creelengine/deliver.go:26`).
- D3, loss happens only between the confirmed acknowledgement and Claude Code reading the exit, or at session death or hook timeout: **broken** (F3).
- D3, "The ring is answered only by that acknowledgement": **broken in wording, harmless in effect** (F4).
- D3, the claim starts the Stop count over as a ring does today: **held**. `ringHook` calls `resetStopBlocks`.
- Constraint, "each message is still delivered once ... the one exception is unchanged": **broken** by the F3 path.
- D4, the rewake text and labels and their defaults: **held**.
  - `Qe=H??'Stop hook blocking error from command "${y}":'` and `rt=We??G??"Stop hook feedback"`, with body `${Qe} ${lt}`.
  - The schema carries `@internal` on both keys.
  - `We` comes only from plugin hooks, and lym's waiter is not one.
- Constraint, the 8 KiB budget fits every field: **held**. Each D2 field is separate, and the rewake body is not capped.
- Constraint `PATTERN-tolerant-records`, an older daemon ignores the mark field: **not verified**, the call budget ran out before I could read the daemon's request decoding.
- D5 bound, a hook that never reaches the daemon is not logged: **held**.

**Findings**

**F1 — MEDIUM, design. Open risks bullet 1, Q10 and D1.**
- **Summary:** the mode-check fact D1 rests on is left to the tier3 smoke run, but research settles it now.
  - `Ugt` is `CLAUDE_CODE_SIMPLE || --bare`.
  - reed strips every `CLAUDE_CODE_`-prefixed key from the launch environment, and lym passes no `--bare`. So no lym session can hit the mode check.
  - When the mode is on, the attachment is also skipped when messages go to the API (`case"attachment":{if(Ugt(Mr))continue`). The model would never see the mail, though it is journaled read.
- **Fix:** record this evidence in D1, and close the risk bullet and Q10.
- **Evidence:**
  - Claude Code 2.1.312 binary: `Ugt` at byte offset 248572013, `Sr` at 238925344, the Stop runner's `if(!Ugt(xe))He.push(xe)`, and the module import line at 248422491.
  - `internal/shuttleengine/claudeengine/scrub.go:8` returns `[]string{"CLAUDE_CODE_"}` as a prefix.
  - `internal/reedengine/lifecycle.go:451` applies the scrub.

**F2 — MEDIUM, design. D1 rationale bullet 3 and the Goal ("shows the mail to the operator once").**
- **Summary:** dropping `systemMessage` leaves the stop-hook summary row as the operator's only copy, and that row cuts its sub-lines to 5.
  - The renderer passes them through `Is(..., {maxLines:eu})` with `eu=5`.
  - So a claim of more than 5 lines, or of lines that wrap, shows the operator a cut copy.
  - Today's `Stop says:` line shows the whole mail.
- **Fix:** state this bound in D1, or choose the operator copy with it in mind.
- **Evidence:** Claude Code 2.1.312 binary near byte offset 274855786: `case"stop_hook_summary"` builds `f.map((N)=>`Stop hook feedback: ${N}`)` and passes it to `Is(..., {maxLines:eu})`, with `eu=5` defined in the same chunk.

**F3 — LOW, design. D3's loss bounds and the Constraints bullet "Each message is still delivered once".**
- **Summary:** there is a loss path the record does not name.
  - `hook-wait` writes stderr, then acknowledges.
  - The daemon's `handOut` calls `reading.Ack()`, which journals the mail read, and only then writes the confirmation.
  - If that write fails, or the daemon stops in between, `awaitAckConfirmation` returns `connLostError`. Under D3, `hook-wait` then exits 0 and Claude Code drops the stderr.
  - The mail is journaled read and never shown.
- **Fix:** name this window in D3's bound, with D5's log line.
- **Evidence:**
  - `internal/creelengine/server.go`, `handOut`: `reading.Ack()` comes before `d.write(conn, Response{OK: true, ...})`.
  - `internal/creelengine/client.go`, `awaitAckConfirmation`: the `connLostError` return.
  - Claude Code 2.1.312 binary, `_tt`: only `Ae.code===2` wakes.

**F4 — NIT, consistency. D3, "The ring is answered only by that acknowledgement".**
- **Summary:** existing paths also mark a ring answered.
  - `wakeHookStrand` marks it answered when a probe reads a turn in progress.
  - `ClaimAtBoundary` marks it answered on any hand-out.
  - The effect is benign, but the "only" is false.
- **Fix:** say that the ring's own claim marks it answered at the acknowledgement rather than at the hand-out.
- **Evidence:** `internal/creelengine/deliver.go:274-277` (`markRingAnswered`), and `internal/creelengine/verbs.go`, `ClaimAtBoundary`'s `mark.answered = true`.

**Tool calls**
1. Grepped the binary for `stopHookBlockingCount` assignments: the context lacked how Claude Code resets its count on tool turns.
2. Read `ringAnswerWindow`/`bellGrace`, the waiter entry's timeout in `settings.go`, and the launch-env code: the context lacked the bell window value and how lym builds the launch environment.
3. Grepped the binary for the `next_turn` guards, and read `creelStopHookCommand`: needed to confirm the count reset and the Stop command's flag and exit rule.
4. Grepped for the scrub in `reedengine`: needed to know whether `CLAUDE_CODE_SIMPLE` reaches a lym session.
5. Grepped for `EnvScrub` constructors: located where the engines build their scrub lists.
6. Read the Claude engine's scrub list: confirmed the `CLAUDE_CODE_` prefix is stripped.
