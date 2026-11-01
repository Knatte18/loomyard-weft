```yaml
lens: guard-reach
outcome: reported
```

Task: the guard-reach lens. For each guard, refusal, branch and test row the cards add, I judged whether a real entry point can reach it.

**Per guard, reachable or not**
- Card 1 `CoverageOf` `Held` rows: reachable.
- Card 2 `planPass` early none for `Held`: reachable (adopt's hold, read through `DobbyRuns`).
- Card 2 `drivePasses` skipping the owner recheck for `Held`: reachable. A run dobby recorded owned elsewhere has its owner entry turned into the hold, and the next probe tick reaches the skip.
- Card 3 `checkFrontPayload` refusal of a changed or mistyped `front` or `on_land`: reachable through upsert, upsert-batch and merge.
- Card 3 fixed project or kind refusal: reachable. It runs before `checkProjectRules`' pair check.
- Card 3 `Store.Rename` refusal of a front entry: reachable for a released or draft front task. A running one meets `RunLockedError` first.
- Card 4 `ErrNothingLeft` match: reachable only in the race the card names.
- Card 4 / card 6 "any other cancel refusal fails the step": **not reached for an unreachable origin** (finding 1).
- Card 5 `runCheck` owned-elsewhere refusal: reachable by a hand `lym adopt check`, and by the settle action in a race.
- Card 5 owner read error row: reachable.
- Card 5 `startCheckWithTiers` resolve refusal: reachable (a beam with no pair while its holds stand).
- Card 6 `File` refusal of an unknown `OnLand`: reached only by `TestFile`. It is a fail-loud backstop over a closed caller set, so no finding.
- Card 6 `File` refusal of an open front task: reachable (card 13's case-1 gap and concurrent filings).
- Card 6 `Remove` step failures and the lock against `Settle`: reachable.
- Card 6 `Settle` unknown key, failed action, and gone or done entry: reachable.
- Card 6 `Settle` drop when `OwnedHere` reads false: **reachable only in a race** (finding 2).
- Card 7 front start past `max_runs`, the front hold, `max_runs: 0`, the covering pause, the backoff and the settle notice episode: reachable.
- Card 8 condition marker rows: reachable.
- Card 9 negative-value refusal: reachable.
- Card 11 `holdEntries` branches (owner entry turned into the hold, hold kept, other caller paused, new hold): reachable.
- Card 11 refused owner write leaving the holds: reachable.
- Card 11 `reconcileWarnings` filters: reachable. A prime code branch is never a shared stop; a beam pair branch is unstopped when this machine already owns the beam, or under `--keep-local`.
- Card 11 live-pair not held: reachable only under `--keep-local`, and the test row does not name it (finding 4).
- Card 12 green-end steps (hold lifted by hand, failed lift, `BeamUp` failure, refusal envelope, owner-halted child, removal of a draft, released, running or halted fix task): reachable.
- Card 13 cases 1–5, no-verdict rows and the refused create: reachable. Case 1 has a fall-through gap (finding 3).
- Card 13 case 2, another open front task: reachable across binaries, through a newer binary's action row filed on another machine via Bolt.
- Card 14 `held_failed_check`: reachable only when this call's check start failed, as the card says.
- Card 14 `ended`: reachable. `clearRunPhase` removes the records, at `internal/battencli/cancel.go:428`.
- Card 14 read failure: reachable.

**Findings**

1. MEDIUM · design · card 04 `plan:internal/battencli#Cancel`, as used by card 06 `fronttask.Remove` step 1.
   - **Summary:** an end-only cancel reads as a done cancel step, so the decision record's rule that a cancel which cannot read origin fails the step (D8) is never reached.
   - **Why:** `Cancel` returns only `cancelRun`'s error. An unreachable origin makes `judgeOrigin` return `endOnly` with a nil error (`internal/battencli/cancel.go:475-479`), and `cancelRun` returns the report with a nil error (`cancel.go:200-203`) before `clearRunPhase` clears the run claim.
   - **Effect:** `Remove` then drops the record (step 2), and step 3's board removal hits `RunLockedError`, because the entry still holds a run status (`internal/boardengine/runlock.go:150-152`). The `HandRemoval` it names, `lym board remove <slug>` alone, is refused the same way. D8 wants the record kept and the cancel step named.
   - **Fix:** `Cancel` returns an error when `report.EndOnly` is set, naming cancel's re-run once origin answers. A `TestRemove` row, or a `TestCancelRun` assertion, covers it.

2. LOW · design · card 06 `plan:internal/fronttask#Settle`, the `OwnedHere` false branch, with card 07 `daemon.tick`.
   - **Summary:** the branch that drops records on a machine that lost the subject is reachable only in a race, so a stale record outlives the move.
   - **Why:** a dobby whose beam reads owned elsewhere exits before it ticks (`internal/dobbyengine/loop.go` loop, "another machine owns this beam; the dobby exits"), so `SettleFronts` never runs on the machine that lost the subject.
   - **Effect:** the record survives there. If that machine adopts the subject again, `Settle`, now with `OwnedHere` true, finds the entry gone (it landed on the other machine, which already ran the action) and runs `adopt-check` a second time.
   - **Fix:** the card states how a record left on a machine that lost ownership is dropped, for example by adopt's hold step or a check at the dobby's exit. Otherwise it records the double check as a bound.

3. LOW · design · card 13 `plan:internal/adoptcli#failedEnd`, cases 1 and 4.
   - **Summary:** an own fix task that matches neither case falls through to a filing that `File` refuses.
   - **Why:** case 1 tests the empty status or a batten run record in `running`, `paused`, or `awaiting` with an admitted escalation; case 4 tests draft or `fixRunHalted`. An entry holding a run status whose batten run record does not read yet (between the claim and the status seed) matches neither.
   - **Effect:** `File` then refuses on its open-front check (card 6), and the notice names "fix its cause, then re-run `lym adopt`" instead of the task, its state and its hand step that D6 asks for.
   - **Fix:** case 1 also covers an entry with a run status and no readable record.

4. LOW · consistency · card 11 `internal/adoptcli#TestAdopt` row "a live pair … not held".
   - **Summary:** without `--keep-local` the row reaches the takeover stop, not the live-pair guard.
   - **Why:** for a slug the reconcile reads owned elsewhere, `takeOverSlug` already stops a live pair (`internal/adoptcli/adopt.go:509-532`). So the row passes on the stop, and `holdTargets`' `LivePair` exclusion, reachable only under `--keep-local` (D4), stays untested.
   - **Fix:** the row passes `--keep-local`, and asserts the run is not held and reports running here.

Tool calls:
1. `internal/dobbyengine/loop.go` loop lines and `clearRunPhase` in `internal/battencli/cancel.go`. My context lacked whether a dobby owned elsewhere ever reaches `tick`, and whether cancel removes run records, which `ended` depends on.
