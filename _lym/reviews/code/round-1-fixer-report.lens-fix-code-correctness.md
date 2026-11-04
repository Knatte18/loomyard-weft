```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed branch with concrete inputs. I found one MEDIUM behavior change in the F1 own-fix and one LOW test-wording issue.

**Findings**

1. **MEDIUM** — `internal/creelengine/sweep.go`, `waitsForHubOrch`, used in `sweepSubjects` and `releaseGoneHolder`.
   - **Problem:** the guard keys on `msg.From` being a program. `releaseGoneHolder` returns a message to `Path[last]`, else to `From`. A program's mail that a strand forwarded to a hub orch has a strand as its return step. Example: `ly:BATTEN` posts to beta, beta forwards to `ly:orch`, and the orch is removed.
   - **Before the fix:** the sweep returned that mail to beta.
   - **After the fix:** `From` is a program, so `waitsForHubOrch` is true. The mail stays unread under the orch's name. The forwarder is never told and the sweep never sees it.
   - **Why it matters:** the doc comments and the summary say a program has "no mailbox to take a return". That is false when a strand forwarded the mail.
   - **Evidence:** `releaseGoneHolder` computes `next := msg.From; if n := len(msg.Path); n > 0 { next = msg.Path[n-1] }`, and `waitsForHubOrch` never looks at `Path`.
   - **Suggested change:** add `len(msg.Path) == 0` to the guard, or skip only when the return step is not a strand (`!isStrandName(next)`). Both pass the added rows.
   - **Test gap:** no row covers forwarded program mail to a removed orch.

2. **LOW** — `internal/creelengine/deliver_test.go`, row `UnreadProgramMailToAnUnlistedHubOrchOutlivesTheSweepAndWaitsForItsStrand`.
   - It uses `ly:gone:orch`, which has never been listed. It does not cover an orch that was listed and then removed, which is the other path through `waitsForHubOrch` and the one that changes behavior.
   - It does fail with the defect in place. Without the guard, `ly:gone:orch` is a sweep subject. The first miss and the confirm at 10 s release it, and the mail goes undelivered because its sender is a program, so `readBodies` returns nothing.
   - A second row posting `program(batten)` to the listed `orch`, removing `orch` with `without(e, orch)`, running the sweeps and then listing it again would pin this path.

**Checked, no finding**
- **Sweep rows.** `RemovedHolderIsSettledWithin60sAtTheCeiling` and `PostInsideTheConfirmInterval...` still hold. Their senders are a strand and EXTERNAL, which `waitsForHubOrch` excludes.
- **`takeDoneEntries` and the nil seam.**
  - The `OwnedElsewhere` skip and the `ClearNotifyOnLand != nil` guard behave as intended.
  - A nil seam on a requested run still posts and drops its memory. The entry was already pruned, so nothing retakes it.
- **Tick order.**
  - The notice step runs at `loop.go:586`, before the start step reads `LoadConfig` at `starts.go:79`.
  - A failed config read on a tick therefore reaches the notice step with `modeRead` false. `ADoneEntryIsNotTakenUpWhileTheModeReadFails` fails if the gating is removed, because the entry would be cleared on tick 1 with no line.
- **`DobbyTasks`.**
  - A done entry with an unreadable parent gets an empty `Parent`, so `takeDoneEntries` skips it. A released task still aborts the read.
  - `OwnedElsewhere` is now set for flagged non-released entries, and its consumers filter through `released()`, so nothing else is affected.
- **`landedNotice`.** It has a single call site (`cli.go:557`) and `fields` is not mutated before it.
- **End-to-end test.** With the sweep guard removed, the mail returns undelivered and the strand reads nothing, so it fails as intended.

**Tool calls**
1. `grep` for `LoadConfig()` and `landNotices(` in `dobbyengine` — I needed to know whether the failing-config tick reaches the notice step; my context lacked `tick`'s body.
2. `sed` and `grep` on `starts.go` for `released`, `heldByEntry` and `OwnedElsewhere` — I needed to know whether the newly set `OwnedElsewhere` on non-released entries feeds the start step; my context lacked those lines.
