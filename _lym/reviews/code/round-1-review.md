---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/machinecli/set.go:193
    summary: A hub whose selected entries all come back in Lost reports the outcome nothing to sweep, or pushed, beside its lost list
    fix: In sweepHub (set.go:193-209) give a hub with len(written.Lost) > 0 an outcome that does not read as nothing to sweep, for example a new outcomeKept constant, and in setName (set.go:259) skip the upgrade to outcomePushed for it. runSet already counts it unfinished through len(hub.Lost) > 0, so the exit stays non-zero. Add a row to TestName_Set with Recorded empty and Lost non-empty that asserts the outcome label.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/machinecli/set.go:171
    summary: The prime-owner branch of the collision check has no test, so deleting it would pass every test
    fix: Add a TestName_Set row in internal/machinecli/set_test.go whose fake owners for hubA are a boardengine.Owners value with its Prime field set to osl46. Assert a refusal that names hubA and boardengine.OwnerWritePrime, that the machine file is unchanged, and that no owner write was sent.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/machinecli/set.go:249
    summary: The refusal on a failed write of the machine file has no test
    fix: Add a TestName_Set row in internal/machinecli/set_test.go whose locate seam hands a path under a regular file, so the directory resolves and the read finds no file but state.WriteJSON fails. Assert the refusal names the file path and the re-run `lym machine name osl46`, and that f.requestTotal() is 0.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/ownerset.go:97
    summary: The refusal of a replace set combined with Slugs, Prime, Claim or Takeover has no test row
    fix: Add rows to the outcomes table of TestSetOwners in internal/boardengine/ownerset_integration_test.go, one per combination, each asserting the refusal error and that the board and origin are untouched. A table row keeps it inside the existing test.
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/ownerset.go:33
    summary: The OwnerRequest type comment describes an owner write and a claim but not the replace-set mode its Replace field adds
    fix: Extend the OwnerRequest doc comment at ownerset.go:33-34 with one sentence saying that Replace makes the request take over the prime and the open beam projects recorded under the names it lists.
---

The diff makes the change the decision record settles.
The replace-set selection runs after the write's own pull, touches only open beam projects and the prime, and ignores empty names (`internal/boardengine/ownerset.go:240-253`, pinned by `ReplaceSetSelectsAfterThePull`).
`setName` follows D5's order: name check, file read, hub reads, collision refusal, file write, sweep (`internal/machinecli/set.go:216-265`).
The declared `PATTERN-owner-machine` text is already the overview line in `PATTERN.md`, and the background file gains the sweep clause.
No commit carries a `Pattern-Departure:` trailer, and none is needed.
The wording criterion marked Review is met: `recoveries` (`set.go:33-40`) names each D6 window recovery and says to lift a pause only after the hub's sweep reports done.
No finding is raised on the five redundancy-report tests.
`TestFromThisHost`, `TestName_Show`, `TestName_Set`, `TestName_SweepsHubs` and `TestSetOwners` each pin a behavior no other test covers.
`_lym/landing/summary.md` is absent, so no change description was checked.

### [LOW] Lost-only hub reads as nothing to sweep

**Location:** internal/machinecli/set.go:193
**Issue:** When a competing origin write takes every selected entry, `Recorded` is empty, so `sweepHub` leaves `outcomeNothing`, and `setName` turns it into `pushed` when the board was ahead. The `lost` list and the non-zero exit still appear, but the outcome label contradicts them.

### [LOW] Prime collision branch untested

**Location:** internal/machinecli/set.go:171
**Issue:** `checkHub` refuses a name the hub's prime owner holds. No `TestName_Set` row or integration case sets a prime owner equal to the new name, so removing the branch fails nothing.

### [LOW] Machine file write failure untested

**Location:** internal/machinecli/set.go:249
**Issue:** The refusal naming the file and the re-run after a failed `state.WriteJSON` is reached by no row. `dirAtPath` fails earlier, at read time.

### [LOW] Replace-set guard untested

**Location:** internal/boardengine/ownerset.go:97
**Issue:** The refusal of `Replace` together with `Slugs`, `Prime`, `Claim` or `Takeover` is a new behavior of `SetOwners`. The only `Replace` cases set `Machine` and `Replace` alone.

### [NIT] OwnerRequest comment omits the replace-set mode

**Location:** internal/boardengine/ownerset.go:33
**Issue:** The type comment says the request records `Machine` as owner of the root and beam projects, or claims one entry. The `Replace` field comment and `doc.go` cover the new mode, but the type comment does not.

## Rejected

- Lens code-correctness F2, a hub listed twice when the registry path and the resolved path differ by symlink or case: speculative, with no code path or fixture in the diff that produces two spellings of one hub, and the sweep it would repeat is idempotent.
- Lens code-affected-set finding 1, the `PATTERN-owner-machine` background and the machine file before the hostname: the lens itself found the clause present and called it not a defect.
