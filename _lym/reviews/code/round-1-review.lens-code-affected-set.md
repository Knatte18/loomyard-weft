```yaml
lens: code-affected-set
outcome: reported
```

The code-affected-set lens found one MEDIUM, one LOW and one NIT; every other reader, test, doc and rule of the diff's changes is consistent with it.

**The set found, per change**

1. **`firstPassExited` reads `PassExitOriginWait` as owing the catch-up** (`starts.go:362`).
   - The one producer, `passExitCode`, maps `catchUpRefusal` to that code (`internal/battencli/drive.go:138`). It is consistent with the change.
   - The `catchUpRefusal` doc says "dobby reads that code as no failing pass" (`internal/battencli/catchup.go:30`). It still holds.
   - The later-pass tests of the same code (`passes_test.go:426`, `:445`) are unchanged and still hold.
   - The battencli run tests (`run_test.go:325-345`, `owner_test.go:363`) assert the exit code only. Not affected.
   - The package doc's start bullet (`doc.go:73-74`) and the catch-up-owed trigger (`doc.go:265-266`) are covered.
   - The `starts.go` file header ("posts the notice for a refused first pass") still holds for the other refusals.
   - `contracts/orchrole/orch-role.md:94` lists "a refused start" among dobby's own lines. That holds for the other refusals.
   - `docs/research/autonomy/stop-classes-2026-11-27.md:89,208` is a dated measurement report, which the rule exempts.
2. **`startReady` returns early while the catch-up is owed** (`starts.go:94`).
   - Its one caller is `tick` (`loop.go:746`), which already gates on `catchingUp`.
   - `doc.go:268` ("starts no task") is covered.
3. **`CatchUpWait.Since` and `waitingSince`.**
   - The one writer is `catchUpFailed` (`catchupwait.go:83-89`). The record is cleared in `catchUpSucceeded` (`catchupwait.go:119`).
   - A restart re-owes the catch-up (`loop.go:510`), so the persisted `Since` is carried across it.
   - Readers: `summarisePushes` (`pushsummary.go:95`) and `View.CatchUp` (`view.go:108`), which now emits `since` under `catch_up`.
   - No reader of `catch_up` outside dobbyengine was found.
   - `view_test.go:205-207` builds the record without `Since`. It still passes thanks to `omitzero`.
   - The field is additive and `omitzero`, so an older binary ignores it, as `PATTERN-tolerant-records` requires.
   - Two doc members of this change are left wrong (findings 1 and 2).
4. **The push summary lists the dobby's catch-up waiting on origin** (`pushsummary.go:95-99`).
   - The doc comments at `pushsummary.go:35-36` and `doc.go:277` are covered.
   - The notice's heading and the doc's list of what a notice names are left wrong (findings 2 and 3).
   - No stencil, spec or contract describes the summary's wording; the search found none outside research docs.
5. **Rule documents.**
   - `PATTERN-owner-machine` ("after absence, a catch-up from origin first") still holds.
   - `PATTERN-refusal-way-forward` is not touched: the batten refusal text and its way forward are unchanged.
   - No PATTERN changes are declared, and none land.
6. **Commit messages.**
   - The `Root-Cause:` trailer on `d208b2f6d` names the defect the diff removes: `firstPassExited` routing that exit to `refusedStart`.
   - There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.

**Findings**

1. **MEDIUM, consistency, `internal/dobbyengine/doc.go:271`.**
   The doc states without qualification: "A wait on origin posts no notice of its own; the push summary reports it past its threshold."
   - The summary runs only on the dobby wired with `Deps.PushSummary.Ledger`, the prime's (`doc.go:276`, `pushsummary.go:40`).
   - It reads only that dobby's own state file (`pushsummary.go:86`).
   - So a beam dobby's catch-up waiting on origin is never reported by the summary, and the sentence is false for every beam dobby.
   - The same gap affects the `CatchUpWait` story in the doc: a beam dobby's record is read only by the view.
2. **LOW, consistency, `internal/dobbyengine/doc.go:281`.**
   The doc says the notice lists "each owed branch with its minutes, a refused one with its way forward, and each waiting run with its producer". It leaves out the dobby's own catch-up line the diff adds (`pushsummary.go:98`), which has no producer.
3. **NIT, consistency, `internal/dobbyengine/pushsummary.go:137` and `:106`.**
   - The catch-up line is appended to `waiting` and posted under the heading "runs waiting on the network:", but the dobby's catch-up is no run.
   - The all-clear "no run waits on the network" then covers it only loosely.

**Tool calls**

- One grep over test files for `PassExitOriginWait`, `catch_up` and `CatchUpWait{`, plus `orch-role.md:85-100`. My context did not hold the existing tests or the readers of the record and its JSON key.
- One grep of the docs for the push summary's wording, plus `passes_test.go:415-460`. My context did not hold the summary's doc consumers or the existing origin-wait tests.
- One read of `doc.go:258-290`. My context held only the diff hunks of the package doc's catch-up and push summary sections, not the surrounding text the change makes stale.
