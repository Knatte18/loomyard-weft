---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/holdwrite_integration_test.go
    summary: "WriteHold's former-name rule (Registry.sameMachine) has no test row, so reducing it to a plain name equality breaks no test."
    fix: "In TestWriteHold (internal/boardengine/holdwrite_integration_test.go) add a row that registers the writer under a new name through RegisterMachine so its old name is a former name, seeds a standing hold under that former name, then calls WriteHold with the same kind and subjects under the current name. Assert Wrote is false, Standing is nil and origin holds one hold."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/upgrade.go:writeHold
    summary: "A re-run of a cut first read, or a retry after a failed Drop, posts the hold notice again, since writeHold notifies even when WriteHold wrote nothing and found no other machine's hold."
    fix: "In UpgradeDeps.writeHold, post the notice only when written.Wrote is true or written.Standing is non-nil, and return written.Wrote as now. Add a TestUpgradeRead row whose second run finds its own identical hold standing and asserts no second notice."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/tenderengine/bolthold.go:SyncBoltHold
    summary: "A run that already has a bolt-hold entry is skipped whatever machine the entry records, so after the hold moves to another machine CoveredError and the held refusal name the old machine."
    fix: "In SyncBoltHold's case for a held slug with an own entry, when that entry carries pauserecord.BoltHoldCaller and its Machine differs from req.Machine, lift it with Lift and write a fresh one through Pause with HoldMachine req.Machine. Add a TestSyncBoltHold row for a hold that changed machine."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/pauserecord/doc.go:17
    summary: "The package doc and the OwnerCaller comment in record.go still describe the owner caller as the live pause of a run owned by another machine, though no code path writes it now."
    fix: "Reword the OwnerCaller doc comment in internal/pauserecord/record.go and the owner paragraph of internal/pauserecord/doc.go to say the name stays declared only so the upgrade's first read recognises and drops legacy entries; drop the wording that presents it as the current hold of runs owned elsewhere."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/holdwrite.go:56
    summary: "WriteHold appends a second hold when the same machine already holds one of another kind or other subjects, and no test pins whether two holds stand or the second is refused."
    fix: "Add a TestWriteHold row where one machine writes an adopt hold and then an adopt-fix hold, and assert what the one hold write contract states: either both stand on origin or the second is refused. If the contract is to have one hold per machine, change the append in WriteHold to match."
---

### [MEDIUM] Former-name rule of WriteHold is untested

**Location:** internal/boardengine/holdwrite_integration_test.go
**Issue:** `WriteHold` reads a standing hold as the writer's own through `Registry.sameMachine`, which resolves both names to registry ids. D3 requires every machine-name comparison to go through the registry. The three `TestWriteHold` subtests cover a written hold, another clone's hold and a refused kind. None seeds a former name, so a change to `a == b` keeps them green while a renamed machine reads its own earlier hold as another machine's.

### [LOW] The first read's notice repeats on a re-run

**Location:** internal/dobbyengine/upgrade.go:writeHold
**Issue:** D5 says the first read posts one notice. `WriteHold` answers an identical repeat with `Wrote` false and `Standing` nil, and `writeHold` still calls `Notice`. A first read cut before its drops, or one whose `Drop` failed, runs again from `catchUp` and posts the notice again.

### [LOW] A bolt-hold entry keeps the machine of the first hold

**Location:** internal/tenderengine/bolthold.go:SyncBoltHold
**Issue:** A run with a live entry of its own scope is skipped, so a `bolt-hold` entry written for one machine's hold stays when another machine's hold takes its place before the lift. The refusal then names the wrong machine, against D5's wording that the refusal names the hold's machine.

### [LOW] Owner caller still described as live

**Location:** internal/pauserecord/doc.go:17
**Issue:** Card 23 retired every writer of the `owner` caller. The package doc and the constant's comment still read as its current design. Its readers stay, since they judge legacy entries until the first read drops them, so the wording is the only gap.

### [LOW] Second hold of one machine is unpinned

**Location:** internal/boardengine/holdwrite.go:56
**Issue:** After the pull, a hold of the same machine with another kind or other subjects is appended next to the first. `HoldOf` then answers with the first. No test says whether that is the contract.

## Rejected

- Affected-set F2, owner readers (`Coverage.OwnerHeld`, `OwnerEntryExempt`, `OwnerPauseHeld`, the parent-lock release) left in place: D5 retires the `owner` caller with its writers, and the readers still judge legacy entries until the upgrade's first read drops them. Not a scope gap.
- Tests F3, failing `EndSubject` and `CatchUp` rows of the dobby hand-over: `TestRun_ForeignOwner` in `internal/dobbyengine/handover_test.go` has rows for a failed catch-up and a failed archive that count each step, so the behaviour is pinned.
- Tests F4, forgetting a hand-over memory: `internal/dobbyengine/loop_catchup_test.go:294` asserts two catch-ups after the run stops answering hand over and answers again.
- Tests F5, `ApplyPauseIntents` slugs: the fake in `internal/dobbyengine/catchup_test.go` records the slugs it receives in the call list that `TestCatchUp` compares; its failure path is a plain `errors.Join`.
- Scope lens: found no unrequested scope and no unbuilt decision.

## Focus departures

- Directive: "New tests flagged contained in the redundancy report (TestOwnerPreflight_TeardownResume, TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves, TestPlanPass, TestReadyToStart): compare assertions against the covering tests before filing."
  I did not compare those four tests' assertions with their covering tests' assertions, so this review files no test-economy finding for them and does not clear them either.
  The reading budget went to the correctness and affected-set paths, and no fork was given this comparison.
