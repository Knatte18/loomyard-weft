```yaml
lens: code-correctness
outcome: reported
```

**Correctness list (code-correctness lens)**

1. `isBackgroundLaunch`
   - Input: an Agent block with `subagent_type:"fork"` and no `run_in_background`. It returns true, so it is counted.
   - Input: a Bash block without the flag. It returns false, so it is not counted.
   - Input: an Agent block with no `input`. The nil map read gives false, so it is not counted.
   - Correct.
   - The two callers of the fallback are `events.go:49` (the Stop event kind) and `signals.go:154` (the turn-end signal).
   - The fallback runs only when the Stop payload has no `background_tasks` list.
   - In that case a fork-only outstanding set now makes the turn end `EventWaiting` and `SessionBusy/background`. It was idle or done before.
   - That matches how the payload path already treats forks. It is correct, but it changes behavior for sessions that never call await-forks.

2. `unfinishedForks`
   - Input: a transcript with a Monitor shell and a fork. It returns the fork only. Correct.
   - Input: an `Agent` launch with `run_in_background` (a non-fork subagent). It counts as a `BackgroundFork`, so await-forks also waits on it. Acceptable.

3. `transcriptHasUnmatchedLaunch`, as reused by `OutstandingForks`
   - Input: a launch whose tool_result is not yet in the transcript. `l.ids` is empty, so the launch is skipped and the answer is not outstanding. **Wrong for the wait**, because ready can come back true early; see F1.
   - Input: a launch whose result carries no recoverable id. Same skip, same early-ready outcome. For the fork result, `agentId` is present in the structured result, so this is mostly theoretical.
   - Input: a notification line earlier than `resultAt`. It is ignored (`n.at <= resultAt`). The fork then stays outstanding and no later notification clears it, so ready is never true.
   - Input: a fork that never gets a notification (killed, lost across a resume). The wait stays `ready:false` forever, and the stencil says to re-run it indefinitely; see F2.

4. `Claude.OutstandingForks`
   - Input: a missing transcript. It returns an error, not an empty list. Correct.
   - Input: an empty `sessionID`. It reads `.jsonl`, finds nothing and returns an error. Correct.
   - `workdir` is the runner's `paneCwd`, which equals `anchorPath` for `NewRunner`. This is the same derivation the fork audit uses. Correct.

5. `Runner.AwaitForks`
   - `toldErr` is returned first. Correct.
   - An unknown guid is wrapped as "is not a shuttle strand", but `FindRun` can also fail on an I/O or decode error. The message misreports the cause in that case; see F3.
   - An unregistered engine gives the `engineFor` error. Correct.
   - An engine without the capability gives an explicit error. Correct.

6. `awaitForks`
   - `cap=0`: one read. It returns `(outstanding, false)` if forks are running and `ready` if none. Correct.
   - Negative cap: same as `cap=0`. Correct.
   - Read error: returned. Correct.
   - Cancelled ctx with forks running and cap remaining: returns the ctx error. Correct.
   - Cancelled ctx with cap elapsed: returns not-ready without an error. Harmless.
   - Sleep is `min(pollFloor, remaining)`, so at most one wake a second and never past the cap. Correct.
   - The real clock is `realClock{}` at `run.go:157`, and `DefaultAwaitCap` is 250 s, under the stencil's 300000 ms Bash timeout. Correct.

7. `awaitForksCmd`
   - Empty `LYM_STRAND_NAME` gives an error envelope and exit 1, with a way forward. Correct.
   - A `ResolveStrandGUID` error gives an error envelope. Correct.
   - The not-ready output is `{ready:false, outstanding, message}` and the ready output is `{ready:true}`. This matches the sibling `await` envelope shape. Correct.
   - `PersistentPreRunE` skips only `awaitVerb`, so `await-forks` resolves hub, config and reed, and populates `c.reed`. Correct.

8. `taskFields`
   - Input: nil. It returns an empty non-nil slice. This preserves the old `state` behavior. Correct.

**Findings**

- **F1 — MEDIUM, design.** `internal/shuttleengine/claudeengine/background.go:196-220`, via `OutstandingForks` at `:345`.
  - A fork launch whose tool_result has not been written yet, or whose result id is unrecoverable, is dropped from the outstanding list.
  - `OutstandingForks` can therefore report empty, and the wait returns `ready:true` early.
  - The shared `transcriptHasUnmatchedLaunch` cannot tell "result pending" from "never launched".
  - It matters for this wait because an empty answer ends the wait, which is the very risk `OutstandingForkReader`'s doc comment names.
  - No test covers a launch with no result line.

- **F2 — LOW, design.** `internal/shuttleengine/await.go:289-309` and the stencils.
  - A fork that never produces a notification keeps the wait at `ready:false` indefinitely, and the stencils tell the caller to re-run it forever.
  - The stencils give no exit for a wait that has stayed not-ready for many calls.

- **F3 — NIT, design.** `internal/shuttleengine/await.go:271-273`.
  - Any `FindRun` error, including an I/O or decode error, is reported as "is not a shuttle strand".

**Tool calls**
1. `grep DefaultAwaitCap` and `clock:` in `internal/shuttleengine`: to confirm the 250 s cap is under the Bash timeout and that the runner clock is real. My context did not show either value.
