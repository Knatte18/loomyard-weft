---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/03-reflect-under-step.md
    summary: card 3 removes loomCLI.armedVerb but omits internal/loomcli/wiring_test.go, whose TestArmAt_RecordsTheArmingVerb reads it, and claims the field has no other reader
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/06-halt-notes.md
    summary: card 6 removes shouldReflectFriction but omits internal/loomcli/approve_awaiting_test.go, whose TestShouldReflectFriction_AwaitingNeverReflects calls it, and its ImpactSummary misstates the callers
  - id: F3
    severity: LOW
    location: _lyx/plan/00-overview.md
    summary: verify does not run cmd/lyx's TestLyDriveSkill_IsRecipeBlind, the existing pin on the recipe-blindness of the ly-drive skill text card 7 rewrites
  - id: F4
    severity: NIT
    location: _lyx/plan/03-reflect-under-step.md
    summary: card 3 leaves TestReflectFrictionRow_ReflectsWhenArmedForRun and its doc naming an armed verb that no longer exists
---

### [BLOCKING] Card 3 misses the second reader of `loomCLI.armedVerb`

**Location:** `_lyx/plan/03-reflect-under-step.md`, Edit list and the sentence "`loomCLI.armedVerb` has no other reader".

**Issue:** `grep -rn armedVerb` finds, besides `run.go:104`, `arm.go:147-150`, `cli.go:122-126`, `wiring.go:510` and `friction_test.go:254-259`, the test `TestArmAt_RecordsTheArmingVerb` at `internal/loomcli/wiring_test.go:678-693`, which asserts `c.armedVerb != "start"`.
Card 3 deletes the field, so `internal/loomcli`'s test binary stops compiling, and `go vet`/`go test ./internal/loomcli/...` in the plan's own `## verify:` fails.
`wiring_test.go` is in no card's Edit list, the Intent's "no other reader" claim is false, and the ImpactSummary ("one dead receiver field") omits the test that must go with it.

**Fix:** add `internal/loomcli/wiring_test.go` to card 3's Edit list;
state in the Intent that `TestArmAt_RecordsTheArmingVerb` is removed with the field, since the property it pins no longer exists;
correct the "no other reader" sentence to name the production reader and that test;
and make the ImpactSummary mention the removed test.

### [BLOCKING] Card 6 misses the second caller of `shouldReflectFriction`

**Location:** `_lyx/plan/06-halt-notes.md`, Edit list, the sentence "`shouldReflectFriction` is removed … along with its test", and the ImpactSummary.

**Issue:** `shouldReflectFriction` is called by `loomPostRun` (`arm.go:370`), by `TestShouldReflectFriction` in `internal/loomcli/friction_test.go:137-157`, and by `TestShouldReflectFriction_AwaitingNeverReflects` in `internal/loomcli/approve_awaiting_test.go:69-73`.
Card 6 removes the function but lists neither `approve_awaiting_test.go` nor what happens to that test, so the `loomcli` test binary stops compiling.
The ImpactSummary's "which only its own test called" is wrong twice: a production caller (`loomPostRun`) and a second test in another file call it.
The property the awaiting test pins, that `run` never reflects on `RunAwaiting`, is also not carried over: `halt_test.go`'s `loomPostRun` bullet covers only `RunBlocked` and a failed `runErr`, and its awaiting bullet covers step results only.

**Fix:** add `internal/loomcli/approve_awaiting_test.go` to card 6's Edit list;
state in the Intent that `TestShouldReflectFriction_AwaitingNeverReflects` is removed and its property moves to `halt_test.go` as a `loomPostRun` case asserting a `RunAwaiting` result writes no halt note, spawns nothing and reports `skipped`;
and correct the ImpactSummary to name `loomPostRun` as the production caller and both test files.

### [LOW] Verify does not run the existing ly-drive recipe-blindness pin

**Location:** `_lyx/plan/00-overview.md`, `## verify:`.

**Issue:** `cmd/lyx/lydriveskill_test.go`'s `TestLyDriveSkill_IsRecipeBlind` reads `plugins/ly/skills/ly-drive/SKILL.md` and fails on any shipped recipe name or a `.lyx/`, `lyx loom` or `lyx batten` literal.
Card 7 rewrites that file under the decision record's Constraint "ly-drive carries no row or recipe names", yet `## verify:` runs no `./cmd/lyx/...` tests, so a new `## Filing` or notification sentence naming `lyx loom` would pass this plan's verify and fail only later.

**Fix:** add `./cmd/lyx/...` to the untagged `go test` line of `## verify:`.

### [NIT] Card 3 leaves a test named after the removed armed verb

**Location:** `_lyx/plan/03-reflect-under-step.md`, `friction_test.go` bullets.

**Issue:** `TestReflectFrictionRow_ReflectsWhenArmedForRun` ("asserts run with Tier 2 on attempts the reflection") survives card 3, but once `newRelativeFrictionCLI` drops its verb parameter there is no armed verb, so its name and doc describe a condition that no longer exists.

**Fix:** have card 3 rename it to `TestReflectFrictionRow_ReflectsWhenTierTwoOn` and reword its doc to drop the verb.
