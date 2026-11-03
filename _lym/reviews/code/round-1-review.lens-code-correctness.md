```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. I ran each changed function or branch with a concrete case.**

Per-function list:
- `DriftRecheck.pass`, no scope record, scope off, or a waiver: Done with no compare. Correct (`recheck.go:441-447`).
- `DriftRecheck.pass`, `ComparedAt == ""`: Stuck with no compare, earlier compare fields untouched. Correct.
- `DriftRecheck.pass`, HEAD equals `ComparedHead`: Done with no compare. Correct.
- `DriftRecheck.pass`, dirty tree: Stuck. Correct.
- `DriftRecheck.pass`, compare step undeclared: Stuck. Correct.
- `DriftRecheck.pass`, no acceptances, baseline compare passes or drifts: `recordCompare(head, passed, false)` keeps `BaselineHead` and does not touch the profile head. Correct.
- `DriftRecheck.pass`, acceptances but `AcceptedProfileHead != ComparedHead` (empty or older): lapse, then Stuck with no compare. State is consistent, because `ComparedHead` and `ComparePassed` stay as they were. Correct.
- `DriftRecheck.pass`, accepted-profile compare passes: records HEAD as both the compared head and the profile head, and keeps the acceptances. Correct.
- `DriftRecheck.pass`, accepted-profile compare drifts: records the failure, lapses, then Stuck.
  - The bounce to Drift-Verify then compares against the baseline with the acceptances gone, so the formerly accepted tests are compared again.
  - Correct, apart from F1 below.
- `DriftRecheck.recordCompare` and `lapse`: both go through `UpdateScope`. `lapse` runs after `recordCompare`, so the profile head ends up empty. Correct.
- `DriftRecheck.Call`: `commitOnExit` commits the durable drift directory on Stuck and on errors. Correct.
- `RunAcceptedCapture`: a failing, timed-out or no-file capture is a result that did not pass, and a stale accepted directory is emptied first.
  - A `captureInto` error after the `RemoveAll` leaves the profile empty.
  - `AcceptedProfileHead` was already `!= HEAD`, so `keepAcceptedProfile` never trusts the empty directory. Correct.
- `RunCompare` with `ProfileDir`: an empty value falls back to the baseline directory. Correct.
- `CaptureAccepted` and `CompareAccepted` wiring: both use `Skip = scope.Skip`, so accepted tests are captured and compared (`driftwiring.go:204,217`). The baseline compare at line 190 uses `SkipList`, which also skips accepted tests. This answers the focus question: accepted tests stay in the accepted-profile compare. Correct.
- `keepAcceptedProfile`, all Done paths:
  - A compare pass goes through it (`verify.go:193`), and so does an agent Done (`verify.go:581`), which covers a resume onto a live agent.
  - Waiver, scope off and no scope return Done before any acceptance exists. Correct.
  - A capture that does not pass is a `haltFrom` Stuck. `ComparedHead` is already HEAD, `AcceptedProfileHead` is still not HEAD, and a resume re-runs the compare and the capture. Consistent.
  - Nothing here sets `gate_halt_at`, which is fine.
- `Accept` and `Withdraw` clearing `AcceptedProfileHead`: correct, because the profile no longer matches the acceptances.

Findings:

- **F1 — MEDIUM, design.** `recheck.go:505-508` and `recheck.go:489-494` (`lapse`, `recheck.go:537-542`).
  - A recheck that finds drift, or that finds no profile at the compared head, lapses every acceptance. Acceptances whose output did not change are lapsed too.
  - Under scope full, parent-ruled acceptances are dropped along with the rest, and Drift-Verify's agent can only re-accept them as unruled, which forces a new parent decision halt for each.
  - The code comment calls this a choice made because the compare cannot tell which test changed. The goal asks only that a changed accepted test count as drift again.
  - Lapse only the tests the accepted-profile compare names as differing, or keep the `Ruling` of re-accepted tests. If that is out of scope, record it as a decision for the parent.

- **F2 — LOW, design.** `verify.go:213-223`. After a capture that does not pass, a resume repeats the full baseline compare at the same HEAD before retrying the capture.
  - No wrong state results; it is only cost.
  - It could be avoided by checking `scope.ComparedHead == head &amp;&amp; ComparePassed` before re-comparing, but this is optional.

- **F3 — LOW, design.** `recheck.go:456-458` and `recheck.go:489`.
  - HEAD equal to `ComparedHead` returns Done without comparing, even when acceptances exist and `AcceptedProfileHead` is not HEAD, which is the capture-failed-then-skipped case.
  - Verify always runs first and either captures or halts, so this is unreachable in the normal order.
  - After a manual `lym loom goto` past Drift-Verify it would pass with acceptances that no profile backs.
  - This is an edge case, not a defect in the normal order.

Tool calls:
1. `grep` of `haltFrom` and the `Skip` handling in `driftwiring.go`, because my context lacked `haltFrom`'s body and which Skip each wiring closure uses.
