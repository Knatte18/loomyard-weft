---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/gateslot/tiers.go:157
    summary: The new gateslot.BatchGateKey is declared beside a second constant and three literals for the same key name, against PATTERN-code-conventions' one-named-constant rule
    fix: "Reference gateslot.BatchGateKey at every site that names the batch gate key. In internal/gateslot/tiers.go:157, replace the \"batch\" element of goReadGates with BatchGateKey (same package). In internal/runplan/cardgate.go:65, change render(\"batch\", ids...) to render(gateslot.BatchGateKey, ids...); the file already imports internal/gateslot. In internal/weaverengine/strand.go, delete the gateKeyBatch constant at line 26 and use gateslot.BatchGateKey at its uses, lines 119 and 139; the file already imports internal/gateslot. In internal/webstercli/beginbatch.go:76, replace \"batch\" in []string{\"card\", \"batch\"} with gateslot.BatchGateKey; the file already imports internal/gateslot. Leave weaverengine's stopKindBatch alone: it is a stop kind, not the gate key."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:237
    summary: No test covers the main packages joining a narrowed batch tier under --main-packages, so dropping own = withMainPackages(...) at scope.go:248 would fail no row
    fix: "In the new row \"--gate batch runs tier1 over the cards' packages alone ...\", while tier1 has no covers, write a package main in the root module (as the --main-packages row near line 835 does for ./tool), run f.gate(t, f.pair, nil, \"--gate\", \"batch\", \"--card\", \"1\", \"--main-packages\"), and assert with f.wantArgs that tier1's root-module invocation is the card's package plus the main package (./lib and ./tool), while tier0's root-module invocation holds the dependents and the main package. Derive the exact invocation list, the nested module's \".\" and the guard runs included, from the row's existing expectations. That ordering is not verified here."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/scope.go:345
    summary: A batch tier that runs whole anyway (carry false, --no-carry, or a command without a target placeholder) is still narrowed, so the notice misstates it and the guard step reruns the dependents' tier1 guard tests that the whole run already ran
    fix: "In ownPackageTiers, leave out a tier that runs whole whatever the scope. Append number only when the tier is declared, number != 0, !slices.Contains(tier.Covers, 0), !tier.NoCarry, !request.noCarry and gateslot.HoldsTargets(tier.Command). gateslot.HoldsTargets is exported and already used in tiers.go, and testRequest carries noCarry. Extend the ownPackageTiers doc comment to say that a tier that runs whole is not narrowed."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:237
    summary: No test covers a narrowed tier's //lym:watch skip being taken over the cards' own packages (ownSkip) rather than over the dependents' skip
    fix: "In the new row's fixture, add a //lym:watch *_linux.go test to the dependent package consumer, in an untagged file so tier0 runs it. With tier1 not covering tier0, assert that tier0's invocation carries -skip ^(<that test>)$ and tier1's narrowed invocation over ./lib carries no -skip. The pair has a parent branch, so the changes are read."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-spec.md:331
    summary: The verify-model paragraph still says the card and batch keys both run over the cards' packages and their dependents, contradicting the per-batch bullet the diff adds at lines 345-346
    fix: "Rewrite line 331 so it no longer claims the dependents for every batch tier, for example: \"The `card` key runs per card over the cards' packages and the packages that depend on them, and the `batch` key per batch, scoped as its bullet below states;\". Keep the rest of the paragraph."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/cli.go:125
    summary: The --card flag help still names the cards' packages and their dependents as the scope, which no longer holds for a narrowed --gate batch tier
    fix: "Change the --card usage string to \"plan card id whose packages, and the packages that depend on them, are the scope; under --gate batch a tier above tier0 that does not cover tier0 runs the cards' packages alone; repeatable\"."
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:29
    summary: The change description names the decision record ("the record asks for this one key's behaviour"), which the rubric's Names-of-the-tooling bullet forbids
    fix: "In the Design decisions bullet at _lym/landing/summary.md:29, replace \"and the record asks for this one key's behaviour\" with \"and the change concerns this one key's behaviour\"."
---

### [BLOCKING] The batch key's name is not one named constant

**Location:** `internal/gateslot/tiers.go:157`, with sites in `internal/runplan/cardgate.go:65`, `internal/weaverengine/strand.go:26` and `internal/webstercli/beginbatch.go:76`.

**Issue:** The diff adds the exported `BatchGateKey = "batch"` (`internal/gateslot/tiers.go:128`) and uses it in `decodeGates` and `ownPackageTiers`.
Other sites still spell the name out:
- `goReadGates` in the same file still holds the literal `"batch"`.
- `internal/runplan/cardgate.go:65` still holds the literal.
- `internal/webstercli/beginbatch.go:76` still holds the literal.
- `internal/weaverengine/strand.go:26` keeps its own `gateKeyBatch = "batch"`.

So one key name now has two declared constants and three literals.
`PATTERN-code-conventions` (Named constants) requires "one named constant, declared once and referenced everywhere, never a repeated literal".
Every caller in that list already imports `internal/gateslot`.

### [MEDIUM] The main packages in a narrowed batch tier are uncovered

**Location:** `internal/gatecli/scopes_integration_test.go:237`

**Issue:** `computeScope` adds the main packages to the narrowed set (`own = withMainPackages(own, mainsDir, mains)`, `scope.go:248`).
The summary names this a design decision: the last batch's `--main-packages` call exists to catch `cmd/lym` failures before the plan-level run.
The `--main-packages` rows near `scopes_integration_test.go:835` pass no `--gate batch`, and the new row passes no `--main-packages`.
Dropping that line would leave the last batch's tier1 without the main packages, and no test would fail.

### [LOW] A batch tier that runs whole is still treated as narrowed

**Location:** `internal/gatecli/scope.go:345`

**Issue:** `ownPackageTiers` picks tiers by number and covers alone.
`scopeInvocations` runs a tier whole when its `NoCarry` is set, which `carry: false` or `--no-carry` sets (`tiers.go:123`, `tiers.go:299`).
It also runs a tier whole when the tier's command holds no target placeholder (`tiers.go:307`).
For such a tier three things go wrong:
- `computeScope` still prints that it runs over the cards' packages alone (`scope.go:268`).
- `listedPackages` records only the own packages for it (`scope.go:368`).
- `ranWhole` sees `forTier(1).whole` as false (`tiers.go:411`).

So the guard step reruns each dependent's tier1 `//lym:guard` test that the whole run has already run.
Before the diff those guard tests counted as covered.
Commit aad258757's message confirms that `--no-carry` makes every tier run whole.

### [LOW] The narrowed tier's watch skip is uncovered

**Location:** `internal/gatecli/scopes_integration_test.go:237`

**Issue:** For the narrowed tiers, `computeScope` scans the watch markers a second time over the own packages (`ownSkip`, `scope.go:257-266`).
No row places a `//lym:watch` test in a dependent under `--gate batch`, so `ownSkip` and `sc.skip` are equal in every test.
Swapping one for the other would fail no row.

### [LOW] The plan spec's verify-model paragraph still gives every batch tier the dependents

**Location:** `contracts/specs/loom-plan-spec.md:331`

**Issue:** Line 331 says the `card` and `batch` keys both run "over the cards' packages and the packages that depend on them".
The bullet this diff adds at lines 345-346 says each batch tier above tier0 that does not cover tier0 runs the cards' packages alone.
The spec now holds both statements.

### [LOW] The --card flag help states the old scope

**Location:** `internal/gatecli/cli.go:125`

**Issue:** The usage string says the card's packages "and the packages that depend on them, are the scope".
Under `--gate batch` that no longer holds for a narrowed tier, which the call's own stderr notice and `doc.go` now state.

### [LOW] The change description names the decision record

**Location:** `_lym/landing/summary.md:29`

**Issue:** The Design decisions bullet reads "and the record asks for this one key's behaviour".
That names the task's decision record, which the rubric's "Names of the tooling" bullet flags in the change description.

## Rejected

- code-scope NIT: `ownPackageTiers` narrows every batch tier above tier0, not only tier1. The board entry asks for tier1 by its role, the batch key's tier above tier0 on this hub. A rule keyed on the number 1 would hard-code a tier. The docs the diff changes state the general rule, and the lens itself calls it defensible.
- code-tests LOW: no row pins a narrowed tier1 guard in a card's own package being left out of the guard step. The per-share lookup in `listedPackages` is pinned by the new row, which would fail if the lookup read `sc.modules`. The defect the lens names, dropping the own share alone, is contrived. The plain `--card` row at `scopes_integration_test.go:515` pins the same exclusion through the same code path.

Focus verdicts, recorded for the parent:
- `forTier` readers: run planning, vet, empty scope, skip and report lines all behave correctly for the per-tier shares, apart from F3's whole-running tier.
- Guard coverage: every guard test is run by a listed run or by the guard step. The one needless double run is F3's case. A guard module other than the card's, and a failed `go list`, behave as before.
- Narrowing reach: tier0, the card key and the plan-level `webster` key keep their scope. Webster (`runplan.RenderBatchGate`) and Weaver (`weaverengine/strand.go:119`) both reach the narrowing through `--gate batch`.
- The summary's figures match `docs/research/webster/gate-time-2026-11-28.md`: 141.8 s over 140.5 packages, 32.6 s over 12.4 packages, 4.3 against 18.9 min, and 14.6 min saved.
- The main packages were already in the 32.6 s baseline: `4d8aa9d58` landed on 2026-11-08, before the sample window.
- The `two-way` door holds, since the diff changes no record format, config meaning or outward push.
