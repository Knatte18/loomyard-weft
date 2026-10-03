---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:88
    summary: D6 replaces configreg's literal name list with properties but states no bound; a module dropped from Modules() no longer fails any test
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:114
    summary: D9 downgrades wantProducerTable's routing columns with no Bound line; the OnDone-swap proof is a single check, not a statement of what the perch-shape property lets through
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:170
    summary: D14 has no bound, and D20's claim that shedverbs/step_test.go keeps every assertion D14 deletes is false; loomcli's status-line tests carry cases shedverbs lacks
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:215
    summary: D18's Tmux Test Isolation allowlist is an escape hatch with no stated admission rule, and D18's own over-approximation argument leaves no legitimate entry
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:222
    summary: D19 exempts tmuxkit from the os/exec ban but does not ban importing tmuxkit from another kit, as the Testkit invariant does for lyxbin, so a kit can launder tmux spawns past Tier Purity
  - id: F6
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:325
    summary: The socket-directory acceptance criterion excludes sockets whose servers answer, which also excludes a leaked live test server, so the criterion cannot fail on the leak it guards
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md:248
    summary: D20's audited-and-unchanged paragraph mixes a constraint the plan needs (leave these invariants' text alone) with per-invariant audit justifications that belong in support-log.md
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md:45
    summary: Rationale lines for D1, D4, D5, D6, D8, D9, D10 and D12 are churn statistics that carry no requirement for the plan writer; the evidence belongs in support-log.md
  - id: F9
    severity: NIT
    location: _lyx/discussion/decision-record.md:332
    summary: Two Notes lines restate a Decision (the reed -L line restates D17; the validate.go line restates D14)
  - id: F10
    severity: NIT
    location: _lyx/discussion/decision-record.md:69
    summary: D4's clause ending in "execs go" reads as a garbled sentence; it means the LookPath skips and the go env GOMOD execs are removed
---

The record is well bounded overall: D2, D5, D7, D10, D12, D13 and D17 state what they let through and what limits it.
The findings below are the places where a removed or downgraded guard, or a new escape hatch, has no such statement, plus one uncheckable acceptance criterion and some relocation polish.

### [MEDIUM] D6 removes the check that notices a dropped config section, with no bound

**Location:** `_lyx/discussion/decision-record.md:88` (D6)

**Issue:** `TestNames` in `internal/configreg/configreg_test.go` pins membership as well as order.
D6 replaces it with "sorted, unique, and each with a template that resolves", so deleting an entry from `Modules()` passes every remaining configreg test, and `TestModules_SeedOnly` only checks flags on the modules still present.
A dropped section means `lyx config` stops seeding and reconciling that module's file, with no test failing.
D5 states the analogous loss for deleted verbs and names `registration_test.go` as the independent oracle; D6 states nothing.
A derivable oracle exists: every production `func ConfigTemplate() string` lives in a package that `Modules()` imports, one per module.

**Fix:** add a Bound line to D6 stating that a property test no longer notices a dropped module, and name what does.
Recommended: a scan on `scankit` asserting that every production package declaring `ConfigTemplate` is registered in `Modules()`, which is an edit-free oracle parallel to `registration_test.go`.

### [MEDIUM] D9 downgrades the routing pin without a Bound line

**Location:** `_lyx/discussion/decision-record.md:114` (D9)

**Issue:** D9 drops `OnStuck`, `OnDone`, `Segment` and `MaxBounces` from `wantProducerTable` and keeps only the concrete type, replacing them with the perch-shape property.
"Proof: swapping two `OnDone` targets still fails a test" demonstrates one case and does not say what the property now lets through.
I checked the focus directive's example against the code: a Bouncer wired to another segment's Burler is still caught, because the pair would carry unequal `Segment` values or leave an orphan.
Other changes are still covered elsewhere:
- `TestSequence_FullRunBlocksAtPublish` covers `OnDone` ordering through Publish.
- `TestRecipe_PublishRoutesThroughPRGate` covers the PR-Gate and PR-Rework edges.
- `TestBounceRouting_BudgetExhaustionBlocks` covers Discussion-Bouncer's `max_bounces: 5`.

Some changes are not covered.
The Plan and Webster segments' `max_bounces` value can change freely, because the property only demands equality within a pair.
A review pair's segment label can also be renamed consistently without failing a test.
A plan writer reading only the record cannot know which of these losses are accepted.

**Fix:** add a Bound line to D9: the property no longer pins `max_bounces` values or segment labels beyond pair equality.
`OnDone` order stays guarded by `sequence_test.go`, and the PR edges by `TestRecipe_PublishRoutesThroughPRGate`.
Keep the swap proof as an acceptance check.

### [MEDIUM] D14's deletions are not all duplicates, and D20 misstates where they are kept

**Location:** `_lyx/discussion/decision-record.md:170` (D14); `_lyx/discussion/decision-record.md:248` (D20)

**Issue:** D14 deletes `loomcli`'s tests "that duplicate `shedverbs`' own" and has no Bound line.
D20 justifies the deletion by saying `internal/shedverbs/step_test.go` "keeps every assertion D14 deletes from `loomcli`".
That is false on two counts:
- The status-line tests live in `internal/shedverbs/status_test.go`, not `step_test.go`.
- `loomcli`'s `TestPrintStatusLinesOnChange` asserts cases that `shedverbs`' `TestPrintStatusLinesOnChange_ChangeOnlyPrinting` does not, namely `TransientUnavailableIsAlsoDeduped`, `ReprintsAfterReturningToAnEarlierLine` and `FirstLineIsAlwaysPrinted`.
  Its `UnchangedRepeats` row also cites a live observation, which makes it protected under D21.

`loomcli`'s `TestRenderStatusLine` likewise has an empty-tails case that `shedverbs` covers only with a different label.
Only `TestStepEnvelope_FieldMapping` is named as moving.
`loomcli`'s own wiring of `shedverbs` stays asserted by the loom-specific tests D14 keeps (`TestStatusCmd_EnvelopeKeySet`, `TestStepCmd_BusyRefusal_BeforeBootstrap`), so that part of the focus question is not a gap.

**Fix:** give D14 a Bound line.
A `loomcli` case with no counterpart in `shedverbs` moves to `shedverbs` before its test is deleted, and the deletion removes only cases asserted there.
Correct D20's Shed Verb-Set entry to name `shedverbs`' own tests rather than `step_test.go` alone, and drop the "keeps every assertion" claim.

### [MEDIUM] D18's allowlist has no admission rule

**Location:** `_lyx/discussion/decision-record.md:215` (D18)

**Issue:** the Tmux Test Isolation scan carries "an allowlist (entry plus reason)", and nothing bounds what may go on it.
D18's own Bound argues that every tagged package should carry the call even when it never starts tmux, since it costs one temp directory.
"This package does not start tmux" is the most likely reason someone would offer for an entry, and that argument rules it out.
An allowlist with no stated admission criterion lets any package opt out, which is the leak D17 closes.

**Fix:** state in D18 that the allowlist starts empty.
An entry is admitted only for a package whose `TestMain` cannot call the entry point, and its reason names that obstacle.
"Does not start tmux" is not a valid reason.

### [MEDIUM] tmuxkit's exemption can be laundered through another kit

**Location:** `_lyx/discussion/decision-record.md:222` (D19); Constraints at `_lyx/discussion/decision-record.md:280`

**Issue:** for `lyxbin`, the Testkit invariant (`CONSTRAINTS.md:94-95`) both exempts it from the `os/exec` ban and bans other kits from importing it.
The import ban exists so that "the kit-on-kit exemption [does not hand] another kit a transitive `go build`".
D19 copies the exemption to `tmuxkit` but not the import ban.
Another kit may therefore import `tmuxkit` and wrap its per-test fixture.
D18's Tier Purity rule counts `tmuxkit` exports as spawns, so an untagged test calling the wrapping kit would get past Tier Purity while spawning tmux.
The review-only bound on `tmuxkit` running only `tmux` is sufficient: it matches `lyxbin`'s precedent, which is review-bound too.
The missing import ban is the gap.

**Fix:** add to D19 and the Constraints line: no non-test file under `internal/testkit/` other than `tmuxkit` itself imports `internal/testkit/tmuxkit`, enforced by the rebuilt Testkit scan, as for `lyxbin`.

### [MEDIUM] The socket acceptance criterion cannot tell a leak from an operator hub

**Location:** `_lyx/discussion/decision-record.md:325` (Acceptance criteria)

**Issue:** the criterion excludes a new socket "by checking which servers answer".
A leaked test server that is still running answers too, so it would be excluded as an operator hub.
The criterion's second clause, "no tmux server started by a test is still running", is the one it then cannot check.
Only a dead socket would be caught.

**Fix:** exclude a new socket only when its key equals `reedengine.ServerName` of one of the operator's own hub paths.
Any other new socket in `/tmp/tmux-<uid>/` fails the criterion, whether or not its server answers.

### [LOW] D20 carries audit narrative alongside the constraint it needs

**Location:** `_lyx/discussion/decision-record.md:248` (D20, audited-and-unchanged paragraph)

**Issue:** applying the completeness-before-leanness test, the list of invariants whose text stays unchanged carries a constraint the plan writer needs: do not edit those sections.
The parenthetical justifications are audit deliberation that tells the plan writer nothing to do, for example "no hand-kept mirror found", "its tests already range `Names()`" and "its exclusion list carries reasons".
One of them, Shed Verb-Set's, is also wrong (F3).
The Rationale line restates the directive.

**Fix:** keep the list of unchanged invariants and the clause about enforcement moving onto `scankit` in D20.
Move the per-invariant justifications to the support log's Interview item 10.

### [NIT] Churn-statistic Rationale lines carry no requirement

**Location:** `_lyx/discussion/decision-record.md:45`, `:72`, `:86`, `:93`, `:112`, `:122`, `:137`, `:157`

**Issue:** these lines record why the task exists ("edited in 29 of 32 commits", "three commits made the same one-word edit", "gofmt re-aligned ~20 lines").
They are evidence for the deliberation, not reasons the plan writer acts on, and the tallies go stale.
The Rationale lines of D3, D7, D11, D15, D16 and D24 carry constraints or design reasons the plan needs and stay.
D10's line also carries "each helper is a de facto second writer already", which supports D10's Bound.

**Fix:** move the churn evidence to the support log and drop those Rationale lines, keeping D10's second-writer clause.

### [NIT] Two Notes lines restate a Decision

**Location:** `_lyx/discussion/decision-record.md:332`, `:336`

**Issue:** the reed `-L` line repeats D17's body and the 61-byte key from D17.
Only the package location `internal/reedengine` is new.
The `validate.go` line repeats D14's reason.

**Fix:** drop the `validate.go` line.
Fold `internal/reedengine` into D17's sentence and drop the reed line.

### [NIT] D4's first sentence is garbled

**Location:** `_lyx/discussion/decision-record.md:69` (D4)

**Issue:** "the `exec.LookPath("go")` skips and `go env GOMOD` execs go" reads as a broken sentence.

**Fix:** "the `exec.LookPath("go")` skips and the `go env GOMOD` execs are removed."

## Verdicts on focus questions that produced no finding

- D19's review-only bound on `tmux`: sufficient as stated, since it matches `lyxbin`'s review-bound exemption; the mechanical gap is the import ban (F5).
- D17 and Auto-mode assumptions: the `-L`-only addressing and `TMUX_TMPDIR` inheritance are stated as assumptions the plan can check.
  The code confirms that every reed argv uses `-L`, and Open risks names the scratch-built-environment exception.
- D18's "untagged entry point spawns nothing": holds.
  The sweep runs one `kill-server` per socket in the kit's own fresh directory, and Test Tier Purity keeps untagged files from starting a server, so the directory is empty.
