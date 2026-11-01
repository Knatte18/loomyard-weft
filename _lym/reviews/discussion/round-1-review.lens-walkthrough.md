```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each Decision run end to end on one concrete case, with the end state checked against the Decision's purpose.

**Decisions walked**
- D1. Case: `lym adopt payments` on Neve, with the beam owned by OSL-1046. The steps run reconcile, local-commit stop, pairs, holds, owner write, `BeamUp`, check, report. Purpose met.
- D2. Case: the prime's code branch is `diverged` after the reconcile. It becomes a warning naming `lym fabric pull` and adopt goes on. Purpose met.
- D3. Parent rulings with no mechanism of their own; walked through D4 to D9.
- D4. Case: run A is moved, not done and owned by OSL-1046; task B is released with an empty status. Both get `HoldCaller` entries before the owner write and are listed in the check record. Purpose met at adopt time; the slot effect on the fix loop is under D6.
- D5 green. Case: payments goes green while the fix task stands halted. The lift, `BeamUp` and the first pass for A run, then the halted fix is cancelled. Purpose broken: B never starts (finding 1).
- D5 red / D6. Case: payments goes red with 3 held moved runs and the default `max_runs`. The fix task is filed, but dobby never starts it. Purpose broken (finding 2).
- D6 re-run. Case: a hand `lym adopt payments` while `adopt-fix-payments` is released and not started. D4 holds the fix task, the check is red, and nothing is filed because the fix task is open. Purpose broken (finding 3).
- D6 slug. Case: two beams whose first 22 characters match. Both map to one fix slug. Purpose broken (finding 5).
- D7. Case 1: the board entry's own incident, a run that landed on the other machine. It never reports `landed` (finding 4). Case 2: a fix task halted on a machine failure. It reports `running_here` (finding 6).
- D8. Case: the fix task lands while dobby is restarting. `on_land` never runs (finding 7). Case: root. The fix task has no project, so its parent is the prime and the prime's dobby spawns `lym adopt check root`. Purpose met.
- D9. Case: the task body names "stop if the Go toolchain is older than X". The writer writes the stop report and marker, and the run halts. Purpose met.

**Findings**

1. BLOCKING · design · D5 green end and D8 hold-lift rule. Cancelling the halted fix task leaves it `draft` with `front` set. It is neither done nor removed, so the front start hold never lifts. Every released task in the worktree, including D4's held task B, then waits forever. The next failed check also files nothing, because the draft fix task still reads open. Cancel without `--remove` also leaves the pair, so the slug cannot run again. Evidence: `internal/battencli/cancelcmd.go:491-504` (`clearRunClaim` writes `StatusDraft`), `cancelcmd.go` cancel Long ("--remove must run before the slug runs again"), and decision-record.md D8 ("lifts once it is done or removed") and D6 ("still open is never filed twice").

2. BLOCKING · design · D6 with D4. Held moved runs take `max_runs` slots, so the fix task can be filed and never started. A held run is not owned elsewhere after the owner write, so `slotsTaken` counts it. With 3 held runs and the default `max_runs: 3`, `startReady` breaks before reaching the fix task. The check stays red, nothing lifts the holds, and nothing reruns the check. Evidence: `internal/dobbyengine/starts.go:76-79,142-164` and `internal/dobbyengine/template.yaml:1`.

3. BLOCKING · design · D4 second bullet with D6. D4's hold of "each task under an adopted subject that is released with an empty status" has no owner condition, so a hand re-run of `lym adopt <subject>` holds a released, not-started `adopt-fix-<subject>` task. D6 prescribes that re-run. If the new check is red, it files nothing because the fix task is open, and the fix task stays held. That is a deadlock only `lym tender resume` breaks. D4 does not exclude front tasks. Evidence: decision-record.md D4 bullet 2, D5 "Adopt starts the check … whose record lists a hold still standing", D6 "never filed twice"; `internal/dobbyengine/starts.go:46-55`.

4. BLOCKING · design · D7 `landed`. `landed` reads "whose board entry reads done after it [the reconcile]". Land prunes every done entry from the board, so a run that landed on the other machine has no entry after the reconcile. The state does not fire in the board entry's own incident, and the D7 acceptance bullet for `landed` cannot be met as written. The record states the pruning itself in D8 ("Land prunes the landed entry"). Evidence: `internal/battencli/land.go:103-109` (`PruneBoard` → `board.Prune(true)`) and `internal/boardengine/board.go:388` ("Prune removes every done entry").

5. MEDIUM · design · D6 slug rule. "adopt-fix-" is 10 characters and `MaxSlugLength` is 32, so the subject is cut to 22 characters. Two beams sharing their first 22 characters get one slug. Beam B's failed check then sees beam A's open fix task as its own and files nothing; B's holds stay and B's check never reruns. B's green end could also cancel A's halted fix task. The record states no rule that keeps the cut unique. Evidence: `internal/boardengine/task.go:350`; decision-record.md D6 bullet 1 and D5 cancel bullet.

6. MEDIUM · consistency · D7 `running_here`. The fix task is listed under `running_here` ("the fix task included"). A fix task halted on D9's machine-failure stop, the designed outcome the operator must act on, would read `running_here`. D7's own purpose is to tell whether a run "should run or should not". Evidence: decision-record.md D7 state list and D6 "halts as any darn stop does".

7. MEDIUM · design · D8 `on_land`. The action is modelled on `notify_on_land`, whose memory never remembers a run first seen past its Land row. If the fix task lands while its dobby is down or restarting, `adopt-check` never runs: the subject stays held with no check started and no notice. D8 states no bound or recovery for this case, though a hand `lym adopt <subject>` would recover it. Evidence: `internal/dobbyengine/landnotice.go` `landNotices` doc ("A run first seen past its Land row is never remembered, so it gets no notice"); board note `landing-watch-limits`.

Tool calls:
- `internal/dobbyengine/landnotice.go` head and the `AdoptScratchDir` grep: to see how the `notify_on_land` memory D8 copies behaves; the manifest named the file but not its contents.
- `internal/battencli/land.go` grep: to learn whether Land marks an entry done or prunes it, for D7's `landed`.
- `boardengine.Prune` doc and a `StatusDone` grep: to confirm that the prune removes done entries.
