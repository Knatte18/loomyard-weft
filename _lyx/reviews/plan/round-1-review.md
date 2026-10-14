---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/03-loom-config-zero-rounds.md:56
    summary: Card 3 makes discussion_, plan_ and webster_max_bounces null in template.yaml, but lyx config --set on a null template key writes a value tagged !!null, so the per-perch soft-cap way forward and any operator set of those keys silently loads as null (inherit)
    fix: In card 3, add `internal/yamlengine#SetValues`, `internal/yamlengine#SetSparse`, `internal/yamlengine#TestSetValues` and `internal/yamlengine#TestSetSparse` to `**Edit:**` (the overlay run admits them). Add a `internal/yamlengine` bullet to the Intent saying that a value set on a template leaf whose tag is `!!null` takes the tag its own text resolves to, not the template's, in SetValues (set.go:134, where an existing null leaf's tag is also carried by applyExistingOverrides) and SetSparse (layers.go:204). Add one row to each of TestSetValues and TestSetSparse over a template whose plan_max_bounces is null (~), setting `plan_max_bounces=2`, asserting the written document holds plan_max_bounces as the integer 2 with no `!!null` tag and decodes into an `*int` as 2.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-arbiter-hold-reading.md:46
    summary: The hold re-reads Eligible on every pass, but the arbiter records its own circling accept or continue before a resume verdict, which makes the escalation non-pending, so Eligible turns false and stopHold ends the hold at since in the window after the verb removes the marker and before the driver applies the resume, the incident window D9 and its acceptance criterion name
    fix: In card 7, change the HoldReading bullet so that an awaiting stop whose Check refusal comes from the escalation no longer being pending reads Eligible true when an attempt exists at StopKeyOf(status) with no applied mark, since the arbiter's own circling accept or continue consumed the escalation. HoldReading must then read the attempt list before it decides Eligible for an awaiting stop. Add a TestHoldReading row at Plan-Bouncer awaiting with round-1-escalation.md, a round-1 circling decision file, and an attempt at the key whose envelope carries verdict resume and no applied mark, reading Eligible true with HandOnAt and AppliedAt zero. In card 8, extend the incident-shape row of TestNotice_EligibleStopHold with a reading whose arbiter marker is gone, whose attempt has a resume envelope and no AppliedAt, and whose clock is past noticeReportWait, and assert that no notice goes before AppliedAt or the backstop.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/03-loom-config-zero-rounds.md:74
    summary: Card 3 says loomCLI.reviewBudget compiles unchanged, but its loaded-config test c.cfg.ReviewMaxBounces > 0 (internal/loomcli/arm.go:261) reads a loaded run-wide review_max_bounces of 0, which card 3 now admits, as not loaded, so it reloads loom.yaml and on a load failure reports the template instead of the wired config
    fix: In card 3, move `internal/loomcli#loomCLI.reviewBudget` from `**Uses:**` to `**Edit:**` (the overlay run admits this). Replace the Intent bullet at line 74 with one saying reviewBudget tests a loaded config by c.cfg.ReviewHardMaxBounces > 0, which LoadConfig guarantees is at least 1, in place of ReviewMaxBounces > 0, and that its doc comment no longer names skip_merge_review.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/plan/03-loom-config-zero-rounds.md:58
    summary: "The settled design's point (4) rewrites <perch>_max_bounces: 0 to null in hub and worktree files, but card 3's MigrateConfig rewrites only a document that holds skip_merge_review, so a sparse worktree-layer file holding a per-perch 0, written to inherit over a hub value above 0, silently becomes zero rounds after the binary change"
    fix: In card 3's MigrateConfig bullet, either state this as a departure from point (4), naming the sparse worktree-layer case it leaves and why the hook, which sees bytes and no layer, cannot tell it from a run_settings 0, or have reconcileSparseFile's caller pass the layer so the 0-to-null rewrite runs on worktree-layer files and never on local ones. If the second route is chosen, add `internal/configsync` targets and a row to its migrate table.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/02-zero-round-perch-routing.md:52
    summary: Card 2 says Routing needs no zero-round handling because the projection carries the 0 as MaxBounces, but shedengine reads a ProducerDef MaxBounces of 0 as inherit (effectiveMaxBounces, run.go:634), so a zero-round perch's rows report the engine default in the progress projection. That contradicts the Routing doc sentence at internal/loomrecipe/loomrecipe.go:123, which card 2 keeps.
    fix: In card 2's loomrecipe bullet on Routing and in the doc.go and Routing doc-comment bullet, say that a zero-round segment's rows carry MaxBounces 0, which the progress projection reads as shedengine's inherited budget, and that the Routing doc's claim that the projection never reports the engine default for a review row excepts a perch at 0. Alternatively, name in the Intent the projection change that shows such a perch as zero rounds.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/04-darn-config-per-run-budget.md:50
    summary: Card 4 prescribes that darnengine.ValidateLocalValues refuses a document that does not parse, but names no test or input that reaches that refusal, and every caller's document has already passed configengine.ValidateLocal, which PrepareRunSettings runs first and which parses the same document
    fix: In card 4's ValidateLocalValues bullet, word the decode failure as a fail-loud guard that wraps the decode error marked configengine.ErrInvalid, with no refusal-spec row and no test row, instead of a refusal.
---

### [BLOCKING] Null per-perch budgets break `lyx config --set` on those keys

**Location:** _lyx/plan/03-loom-config-zero-rounds.md:56

**Issue:** Card 3 changes `template.yaml` so that `discussion_max_bounces`, `plan_max_bounces` and `webster_max_bounces` become `~`.
`configengine.Set` writes through `yamlengine.SetValues` for the base layer and `yamlengine.SetSparse` for the worktree and local layers.
SetValues assigns only `.Value` on the template leaf at `internal/yamlengine/set.go:134`, and an existing null leaf's `!!null` tag is copied onto that leaf by `applyExistingOverrides`.
SetSparse builds the new node with `Tag: templateLeaves[pair.Key].Tag` at `internal/yamlengine/layers.go:204`.
In both cases the leaf ends up tagged `!!null` with value `2`.
yaml.v3's encoder keeps an explicit tag whenever the value's own resolved tag differs from it and the tag is not `!!str` (encode.go:444-456), so the file gets `plan_max_bounces: !!null 2`.
On decode, `ShortTag()` returns `!!null` and `prepare` treats the node as null, so the key loads as nil, which is inherit.
The commands this plan's own refusals name therefore do not work: card 1's hard-below-soft refusal names `lower the soft cap: <Route of softKey>` with `--set <perch>_max_bounces=<n>`, and an operator setting a per-perch budget also hits it.
No template in the repo holds a null leaf today, so this path has never been exercised.
`run_settings` is not affected, because `PrepareRunSettings` marshals typed JSON values itself.

### [BLOCKING] Eligible-stop hold drops when the arbiter records its own circling decision

**Location:** _lyx/plan/07-arbiter-hold-reading.md:46; _lyx/plan/08-batten-eligible-stop-hold.md:43

**Issue:** Card 7 sets `Eligible` from `Check(status, "", "", NewReaders(location))` on every pass, and card 8's `stopHold` ends the hold at `since` as soon as `Eligible` is false.
At an awaiting escalation, `Check`'s escalation reader is `shedadapters.PendingEscalation`, which returns not pending as soon as a circling decision file exists for the round (`internal/shedadapters/circling.go:157-166`, where `decided` is the file-exists flag).
The arbiter stencil requires that "A `resume` at a review escalation needs its circling decision recorded first" (`loom-template-arbiter.md:70`), and the arbiter records that decision itself with `lyx loom circling accept|continue --by arbiter`.
The applied mark that would end the hold is written only when the re-launch consumes the attempt (`internal/loomcli/arbiterconsume.go:59`).
So once the arbiter has recorded its decision, `Eligible` reads false, the hold ends at `since`, and `stateChangeReady` measures `noticeReportWait` from `since`, which has long passed.
The notice then goes with `report none yet` between the verb's `finish` removing the marker and the driver applying the resume.
That is the third window D9 says the hold covers, the acceptance criterion "after the verb removed the marker and before the driver acts on the verdict", and the shape the record infers for `tender`.
D9 also says no relay follows a `resume`.
Card 8's incident row feeds static readings to `stopHold`, and card 7's rows include no escalation with a recorded decision, so neither test catches this.

### [LOW] `reviewBudget`'s loaded-config test breaks at a run-wide budget of 0

**Location:** _lyx/plan/03-loom-config-zero-rounds.md:74

**Issue:** `loomCLI.reviewBudget` takes `c.cfg` as already loaded only when `c.cfg.ReviewMaxBounces > 0` (`internal/loomcli/arm.go:261`).
Card 3 admits `review_max_bounces: 0` as a loaded value, so under the full wiring that config reads as unloaded.
`reviewBudget` then loads loom.yaml a second time, and if that load fails it warns and reports the template's budgets instead of the run's.
The card asserts the function compiles unchanged and lists it under `Uses` only.

### [LOW] The 0-to-null migration skips sparse worktree-layer files that the settled design names

**Location:** _lyx/plan/03-loom-config-zero-rounds.md:58

**Issue:** The settled design's point (4) says loom's Migrate hook rewrites an existing `<perch>_max_bounces: 0` to null "in hub and worktree files".
Card 3 makes the rewrite conditional on `skip_merge_review` being present, and only full files carry that key.
`configsync.reconcileSparseFile` runs the hook over sparse layer files too (`internal/configsync/configsync.go:408`).
A sparse worktree-layer file holding `plan_max_bounces: 0`, written under the old meaning to inherit over a hub value above 0, would keep its 0 and switch that perch's review off.
The card argues the narrowing only for run_settings and local files, not for this worktree-layer case, and its Intent does not mark the narrowing as a departure from point (4).

### [LOW] The routing projection reports the engine default for a zero-round perch

**Location:** _lyx/plan/02-zero-round-perch-routing.md:52

**Issue:** Card 2 has `applyReviewBudget` give a zero-round segment's rows `MaxBounces` 0, and leaves `Routing` alone because "the projection carries the 0 as `MaxBounces` already".
Since `Routing.MaxBounces` is left at 0, `shedengine`'s `effectiveMaxBounces` (`internal/shedengine/run.go:634`), which `progress.go:133` uses, resolves a row's 0 to the shed value and then to `defaultMaxBounces`.
Status progress would therefore show the engine default for that perch.
That contradicts the doc sentence card 2 keeps at `internal/loomrecipe/loomrecipe.go:123`, "so the projection never reports the engine default for a review row".

### [LOW] Darn's unparseable-document refusal has no reaching input

**Location:** _lyx/plan/04-darn-config-per-run-budget.md:50

**Issue:** Card 4 prescribes a refusal in `darnengine.ValidateLocalValues` for a document that does not parse.
The card names no test or input that reaches it.
Its one caller path is `configreg.PrepareRunSettings`, which builds the document with `yaml.Marshal` and runs `configengine.ValidateLocal` on it first (`internal/configreg/runsettings.go:56-71`).
`ValidateLocal` parses the same bytes, so an unparseable document is refused upstream before this check runs.

## Rejected

- Card order: every `Uses` of a `plan:` handle names a symbol an earlier card creates. Card 2 runs before card 3 and keeps loom.yaml's present meaning in `perchMaxBounces`, and `TestWire_ReviewKeysReachTheEnv` and `TestWire_MergeReviewSeams` stay coherent at each card's commit. Card 3's darn branch of `perchHardCap` through `reviewbudget.Route` gives the same hub-wide text before card 4 opens the local layer. Not a defect.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover `contracts/stencils`, `cmd/lyx`, `internal/reviewbudget` and `internal/arbiterstop`, and the section follows the stencil's stated order exactly. Not a defect.
- tree-sitter link: none of `arbiterstop`'s planned imports, nor `locationkit`, reaches `github.com/tree-sitter/go-tree-sitter` (`go list -deps` checked), so `battencli` importing `arbiterstop` does not trip the link guard. `battenshed`'s import allowlist is untouched, since `ChildArbiterStop` and `stophold.go` need only `time`, `context` and `shedengine`.
- Test budget rows: the counts match the tests on disk (loomengine 57 to 58, battenshed 113 to 114, reviewbudget 3 without `TestMain`, arbiterstop 1 then 2).
- Test economy for `TestCheckResolved`, `TestCheckLocal`, `TestEscalationEligible`, `TestHoldReading`, `TestMigrateConfig` and `TestNotice_EligibleStopHold`: each pins a new public surface or new behaviour no existing test covers. The verb rows in `TestArbiterVerb_SpawnPath` assert refusal text, not the Go driver's boolean fact, and `configsync`'s floored-module table drives reconcile, not the new rewrite rules.
- Re-sign arrows: every Edit member whose signature changes carries one (`loomrecipe#Routing`, `loomCLI.reviewBudget`, `Config.Caps`). The rest keep their signatures or are struct types.
- Fidelity for D5 to D9: the narrowing to `hard-cap` and `circling`, one eligibility function shared by the verb, the Go driver and batten, D7's fork-first transient rule, D8's refusal hand-off, D9's stencil line, its hold end conditions, backstop, restart rule and stop-report-file case, and the `orch-template-role.md`, `battenshed/doc.go` and `loomcli/doc.go` updates are each carried by a named card and test row. The exception is the window F2 reports.
- Card 3's `skip_merge_review` migration is licensed by the settled design's point (5).
