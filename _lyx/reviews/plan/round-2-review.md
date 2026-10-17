---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/05-run-bounded-lookup.md:65-67,89,91
    summary: wireLanding's seed-read failure is reachable with no reaching test, and card 5 justifies it as unreachable and as an exempt raw read failure although the error also covers decode failures
    fix: In card 5's Intent, make three edits. (1) Replace the sentence at lines 65-66 ("For the self run an undecodable seed is refused before `wireLanding` ... `wireLanding` still returns `seedRunStart`'s error unwrapped, as a propagated read failure.") with one saying `wireLanding` returns `seedRunStart`'s error unwrapped, that `loomCLI.resolveRecipe` and the seed read at `sharedbootstrap.go:100` refuse an undecodable seed first only when the addressed run is the self run, because `resolveRecipe` reads the seed of `c.runID` while `seedRunStart` always reads the self seed, and that the error covers a decode failure as well as a read failure. (2) Extend the closing sentence of the `TestWireLanding_TellsLanguageSkills` paragraph (line 89) so that, after the `RunStart` assertion, the test overwrites `shedrun.SeedFile(location, shedrun.SelfRunID)` with `{not json`, calls `c.wireLanding()` again and asserts a non-nil error whose text carries that seed file path; this pins the acceptance criterion that a seed that exists but does not decode fails the landing wiring. `TestWireLanding_TellsLanguageSkills` and `loomCLI.wireLanding` are already `Edit` targets, so no target is added. (3) In the `ImpactSummary` (line 91) replace "the only new refusal is the approve and reject verbs' transient read of the run's start" with a conclusion that names both new failure branches, the verbs' `runStart` read and `wireLanding`'s `seedRunStart` error.
---

### [LOW] wireLanding's seed-read failure has no reaching test and is mislabelled

**Location:** `_lyx/plan/05-run-bounded-lookup.md:65-67`, `:89`, `:91`

**Issue:**
Card 5 gives `wireLanding` a new failure branch, the error `seedRunStart` returns for a seed that exists but cannot be read or decoded.
The decision record's D1 bound and its acceptance criterion both say that such a seed fails the landing wiring, so the plan must pin it.
No row of card 5 or card 4 reaches the branch through `wireLanding`.
Card 4's `TestSeedRunStart` reaches `seedRunStart` alone, and the `SeedUnreadable` rows reach the verbs' `runStart` seam.
The sole `wireLanding` test extension asserts `RunStart` equals the stamp, which is the success path.

The card's reason for adding no test is that an upstream check rejects the input first.
That holds for the self run only.
I read `loomCLI.resolveRecipe` (`internal/loomcli/arm.go:127-133`): it calls `shedrun.ReadSeed(location, c.runID)`.
`resolveRunID` (`arm.go:89`) sets `c.runID` to `args[0]` for every verb but a few, so for `lyx loom run <other-run-id>` or `lyx loom step <other-run-id>` the upstream check reads that run's seed.
`seedRunStart` always reads the self seed, and `loomPreRun` reaches `wireLanding` at `arm.go:412`.
So with a healthy other-run seed and an undecodable self seed, the branch is reachable and nothing tests it.
Even for the self run, no test pins `resolveRecipe`'s refusal of an undecodable seed (`TestResolveRecipe` in `arm_seed_test.go` has only the found and not-found rows), so the card rests D1's "approve and reject refuse with it" on behavior no test holds.

The card also calls the branch "a propagated read failure", which `refusal-spec.md` exempts as a raw I/O failure.
`seedRunStart`'s error wraps `shedrun.ReadSeed`'s, and `ReadSeed` fails on a decode error and on a seed whose driver or params keys fail validation, as well as on a read error.
Those are not raw I/O failures, so the exemption does not cover them.
`PATTERN-refusal-way-forward` binds `internal/loom*`, and the decision record's own Constraints list it.

The `ImpactSummary` says the only new refusal is the verbs' transient read.
That contradicts the Intent, which gives `wireLanding` a second one.

Round 1's F2 removed a `{not json` assertion from `TestWireLanding_TellsLanguageSkills` on the ground that no entry point reaches the branch for the self run.
Fixer-report round 1 did not dispute anything, so this finding does not reverse a disputed item; it adds the non-self run-id route that F2 did not weigh, which makes the branch reachable, and I rate it LOW because that route is rare.

The focus directive's first question for `approveVerb` and `rejectVerb` settles as follows.
Each verb's `runStart` refusal is reached by the named `SeedUnreadable` row, and its "transient, re-run lyx loom approve" clause is a way forward the spec accepts and edits no state file.
For an on-disk undecodable self seed the verbs never reach it, because `resolveRecipe` rejects the seed first, and the card says so openly.
I do not flag that branch; it is a transient read failure, reached by the seam row.

**Focus 2, settled with no finding:**
`internal/landingshed/doc.go:6` is a line of the Publish paragraph (lines 3-9) as the file stands, and no earlier card edits that file, so card 5's one added sentence has a definite place.
The decision record's note that `doc.go` calls the lookup "the newest pull request ... whatever its state" does not match the file; that text lives only in `FindPullRequest`'s comment, and the card correctly adds a sentence rather than amending a phrase.
Card 4 creates `runstart.go` with a header naming the seed read, and card 5 extends that header with the run-bounded lookup in the same file.
The header leads the file and is unique, so the edit is definite, and after both cards it names both things the file holds.
