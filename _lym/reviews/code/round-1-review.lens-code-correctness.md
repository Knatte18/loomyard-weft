```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Task: run each changed function or branch on a concrete case, with the cases and a verdict first, then the findings.**

**Cases run**

| Function / branch | Case run | Verdict |
|---|---|---|
| `deliverPromptAtHold` (`hold.go:130`) | Operator turn start, then Stop, prompt never typed | Correct up to the send; see F1 |
| `deliverPromptAtHold` | Prompt's own turn start present, then Stop | Correct: false, notice sent |
| `deliverPromptAtHold` | `PromptLine == ""`, covered, or `turnStartRead` | Correct: returns false |
| `deliverPromptAtHold` | Send error | Logged at Warn, falls through to the hold notice, retried at the next held turn end. Correct |
| `deliverPromptOnAttach` (`hold.go:147`) | Attached, zero `PromptOffset`, only the skill-load turn start in the file | Correct: the skill-load turn start no longer matches, prompt is typed |
| `deliverPromptOnAttach` | Output files all present, or not attached | Correct: no send |
| `deliverPromptOnAttach` | Run covered by a pause | Wrong: no covered check, see F3 |
| `DeliverPendingPrompt` (`run.go:869`) | Prompt turn start past the offset | Correct: false |
| `DeliverPendingPrompt` | Second call after a send | Correct: false |
| `DeliverPendingPrompt` | Send returns `nil` having typed nothing | Wrong, see F1 |
| `DeliverPendingPrompt` | Send fails | `promptDelivered` stays false and the next held turn end retries. Correct |
| `DeliverPendingPrompt` | Lock moved before `loadRunState` | Correct: `promptDelivered` and the offset persist are now serialised under `sendMu` |
| `promptTurnStartedSince` (`send.go:859`) | Exact trimmed match; operator text; `/`-leading line | Correct for the registered engines' `typedInstruction` and for the unmodified paste text. A `/`-leading prompt would never match |
| `promptTurnStartedSince` | Engine without `SessionSignalParser`, or unreadable events file | Returns false, so a delivered prompt is typed once more at the first held turn end. Not an issue for claude or codex, which both implement the parser |
| `Open` / `OpenLaunch` / `writeOpenFiles` | No board directory | Correct: nothing written, nothing marked |
| `Open` / `OpenLaunch` / `writeOpenFiles` | Board directory | Correct: unique per-token dir, so no overwrite of worktree settings and a second open gets a new dir |
| `Open` / `OpenLaunch` / `writeOpenFiles` | Invalid permission mode | Refused before any write. Correct |
| `Open` / `OpenLaunch` / `writeOpenFiles` | `settings.json` written, then the append-prompt write fails | Wrong: directory left behind, see F2 |
| `Open` / `OpenLaunch` / `writeOpenFiles` | `AddStrand` fails | `RemoveAll(sessionDir)`. Correct |
| `creelOpenStopHookCommand` | Flag present and mail claimed | `exit 0` with the mail output. Correct |
| `creelOpenStopHookCommand` | Flag absent, or an echo printed with exit 1 | `true` gives exit 0 with output kept. Same exit status the run's merged command gives. Correct |
| `buildOpenCmd` | Empty paths | Unchanged line. Correct |
| `buildOpenCmd` | Paths present | Both flags on the fresh and the resume line. Correct |
| `creelFlagDirForHook` | Empty board directory, non-Windows, Windows POSIX conversion error | Correct |
| `buildSettings` refactor (`creelWaitHookEntry`) | Run path | Same entry, built once. Correct |
| `attached: true` in `reconstruct` | Attached run | Correct |

**Findings**

**F1. MEDIUM, design. A held-turn-end redelivery can type nothing and still count as delivered, so the reported failure persists.**
- **Where:** `internal/shuttleengine/run.go:895-898` (`DeliverPendingPrompt`), through `sendVerified` at `run.go:1593` and `retryLanded` at `send.go:648`.
- **Trace:**
  - `loadSkillsThenPrompt`'s send fails with `ErrSubmissionNotLanded`, leaving a pending-send record for the prompt text.
  - The operator types, which writes a turn start past the recorded offset.
  - At the held turn end, `deliverPromptAtHold` calls `sendThroughPause`, then `sendVerified`, then `retryLanded`.
  - `retryLanded` returns true because the text matches and `turnStartedSince` (any turn start) holds. It types nothing.
  - `DeliverPendingPrompt` then sets `promptDelivered = true` and returns `(true, nil)`. The hold notice is suppressed and no retry ever follows.
- **Why it matters:** this is the Darn-Write scenario the Goal names, and the new `promptTurnStartedSince` check is bypassed one layer down. The new tests use `echoReed` and do not exercise a pending-send record.
- **Evidence:** `send.go:627-648`, `run.go:1583-1598`.

**F2. LOW, design. A failed open leaves a half-written session directory.**
- **Where:** `internal/shuttleengine/claudeengine/command.go:195-209` (`writeOpenFiles`).
- **Trace:** `MkdirAll` and the `settings.json` write succeed, then the append-prompt write fails. `OpenLaunch` returns the error and `Open` (`open.go:79-82`) returns without removing `sessionDir`. Only the `AddStrand` failure path removes it.
- **Also:** nothing removes the directory after a normal session exit, so each open leaves one behind under `.lym/shuttle-open/`.

**F3. LOW, design. `deliverPromptOnAttach` does not check for a covering pause.**
- **Where:** `internal/shuttleengine/hold.go:147-153`.
- **Trace:** `deliverPromptAtHold` guards on `run.covered`, but the attach path does not. `DeliverPendingPrompt` then enters `sendThroughPause`, which holds `sendMu` and blocks inside `waitWhileCovered` until the pause lifts. `Wait` meanwhile runs no liveness or deadline polling.

**Tool calls**
1. `grep` for `sendVerified`: read it to see whether it transforms the typed text. It does not, and it routes through `retryLanded`.
2. `grep` for `sendVerified` definitions: located the definition at `run.go:1584`.
3. `Read` of `run.go:1575-1645`: needed the `sendVerified` body.
4. `grep` for `retryLanded`: needed its body and condition; it uses an any-turn-start check.
