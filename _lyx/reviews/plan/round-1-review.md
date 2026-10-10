---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/12-loom-arms-darn.md:49
    summary: D13's edited row gate slot gate.yaml absent or invalid, whose trigger must name the darn gate, is carried by no card
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/03-teardown-settles-board-claim.md:38
    summary: card 3 plans a table test over the unexported settleClaim and a new wire_test test for the one-line teardownRequest helper, per-symbol tests PATTERN-test-economy forbids
  - id: F3
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/11-darn-builder-and-routing.md:27
    summary: darnrecipe_test.go re-asserts that every row has an interrupt policy, which the TestInterruptPolicies_MatchAssembledRows union on the same card already pins, and adds a structural-check test where TestRecipe_StructuralCheckHasNoFindings takes a row
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-verify-log-per-run.md:30
    summary: card 1 targets websterengine newVerifyGate, but the retired paths.Log is read in NewVerifyGate and held as a fixed logPath field of verifyGateSeams, which with its verifygate_test.go fakes the card never lists
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-verify-gate-told-command.md:40
    summary: card 6's whole-diff form states no behavior for an empty told command, while the round form passes one with a warning, so the bound on the darn gate's skip path rests on the darn reader alone and is unstated
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-verify-gate-told-command.md:47
    summary: card 6's tests pin neither the whole-diff form's stop on a command read error nor its Publish failure record branch, two acceptance criteria of D6 no other card's tests reach at the gate
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-teardown-settles-board-claim.md:26
    summary: card 3 clears the board-lock-running claim but its Intent never states what the clearing lets through, the scope edits and the one landed-but-unmarked case D10 names
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-loom-arms-darn.md:38
    summary: cards 12 and 13 add a routing edge that lets a darn seed skip Discussion, Plan, Webster and every review row, but neither Intent states that skip or what bounds which runs carry a darn seed
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/04-halt-note-parent-fabric-status.md:23
    summary: Fabric.DirtyPaths returns every uncommitted path, while D11 names the parent pair's dirty tracked paths
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/plan/13-batten-admits-darn.md:19
    summary: card 13 extends the params test, but no test of childSeedParams exists in internal/battencli
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/11-darn-builder-and-routing.md:30
    summary: the verified-tree background opens with every plan-verify site running the plan's verify command, which neither card 1 nor card 11 rewords once the Darn gate, Publish and Finalize in a darn run run darn.yaml's command
---

### [BLOCKING] D13's gate-slot row edit has no card

**Location:** `_lyx/plan/12-loom-arms-darn.md:49` (and the refusal-spec edits on cards 2, 10 and 11)

**Issue:** D13 lists six loom-section rows of `contracts/specs/refusal-spec.md`: four new and two edited.
The plan carries five: `Finalize: parent run records commit failed` (card 2), `Darn verify gate exhausted` (card 10), `darn.yaml: verify_attempts below 1` (card 11), `darn.yaml: verify missing or empty` and the `Publish: verify failed` edit (card 12).
The sixth, `gate slot: gate.yaml absent or invalid` (refusal-spec.md:289), whose trigger today names "the Webster-Burler round gate, Publish's or Finalize's verify" and must name the darn gate among the sites, is edited by no card.
D13's own rationale, `PATTERN-refusal-way-forward`, and the acceptance criterion "the loom section of `refusal-spec.md` holds D13's four new rows and its two edited ones" all need it.

**Fix:** Add to card 6's or card 12's Intent the edit of the loom-section `gate slot: gate.yaml absent or invalid` row, its trigger naming the `Darn` row's verify gate beside the Webster-Burler round gate, with the way forward unchanged.
Both cards already list `contracts/specs/refusal-spec.md` or can add it.

### [BLOCKING] Card 3 tests two unexported helpers directly

**Location:** `_lyx/plan/03-teardown-settles-board-claim.md:38`

**Issue:** `PATTERN-test-economy`: "an unexported helper is tested through the surface that uses it", and a per-symbol test is a finding.
Card 3 plans "a new table test in `teardown_test.go` over `settleClaim`", an unexported pure function, and "`wire_test.go` gains a test asserting `teardownRequest` marks the removal landed", a new test function for a one-line unexported helper (`internal/battencli/wire.go:306`).
The public surface that covers both is `Teardown.RemovePair`, already driven with fake seams by `TestTeardown_CallSequence`, which card 3 extends anyway, and batten's `Worktree-Teardown` row, which reaches `teardownRequest` through `td.RemovePair(teardownRequest(slug))`.

**Fix:** Drive the claim rule's rows through `RemovePair` with fake `landed`/`settle` substrates (or a fake board and status read behind `settleBoardClaim`) as rows of `TestTeardown_CallSequence`, or a table over `RemovePair`, and assert the batten landed flag at the batten teardown row's own test instead of a new `teardownRequest` test.

### [BLOCKING] Card 11's new darn recipe test repeats existing tests

**Location:** `_lyx/plan/11-darn-builder-and-routing.md:27`

**Issue:** `darnrecipe_test.go` pins "that every row has an interrupt policy", but the same card changes `TestInterruptPolicies_MatchAssembledRows` (`internal/loomrecipe/interruptpolicy_meta_test.go`) to union the rows of `New` and `NewDarn` in both directions, which already pins that.
It also pins "that the structural check has no findings", the behavior `TestRecipe_StructuralCheckHasNoFindings` (`internal/loomrecipe/recipe_test.go:87`) pins for loom's recipe; a second embedded recipe is a row of that test, not a new test.
`PATTERN-test-economy`: "A redundant test is a finding exactly as a missing one is."

**Fix:** Drop the interrupt-policy assertion from `darnrecipe_test.go`, and make the darn recipe a row of `TestRecipe_StructuralCheckHasNoFindings` (listing `internal/loomrecipe/recipe_test.go` under `Edit:`).
Keep the darn shape, the routing, the stamped budget and the below-1 refusal in the new file, since no existing test covers them.

### [MEDIUM] Card 1 misses webster's verify seams

**Location:** `_lyx/plan/01-verify-log-per-run.md:30`

**Issue:** Card 1 retires `Paths.Log` and lists `internal/websterengine#newVerifyGate`.
In the code, `paths.Log` is read in the exported `NewVerifyGate` (`internal/websterengine/verifygate.go:153-157`), which builds `verifyGateSeams{readLog: … os.ReadFile(paths.Log) …, logPath: paths.Log}`.
`verifyGateSeams.logPath` is a fixed `string` (`verifygate.go:97`) that `newVerifyGate` puts in every report (`:249`, `:261`, `:343`), and the gate reruns once on a failure, so a per-run log needs `readLog` and `logPath` to follow each result's `Log`, a change to the `verifyGateSeams` type.
`internal/websterengine/verifygate_test.go:63` sets `logPath: "/verify/verify.log"` on its fake seams.
None of `NewVerifyGate`, `verifyGateSeams` or that test file is on the card, so card 1 does not compile as written.

**Fix:** Replace `internal/websterengine#newVerifyGate` with `internal/websterengine#NewVerifyGate` and `internal/websterengine#verifyGateSeams` (plus `newVerifyGate` if its reads change), add `internal/websterengine/verifygate_test.go`, and state in the Intent that the seams read the log of the result they report on.

### [MEDIUM] The whole-diff verify form leaves an empty command unbounded

**Location:** `_lyx/plan/06-verify-gate-told-command.md:40`

**Issue:** The round form "passes an empty command with a warning".
For the whole-diff form, card 6 states only what a command *read error* does; a reader returning `("", nil)` is unspecified, so as written the gate's empty-pass path may stay open in the form D7 says must never pass on a warning ("An empty verify would make the gate pass on a warning … wrong for a recipe whose only safety net is that gate").
Today the one thing keeping it shut is `darnengine.VerifyCommand` never returning `""` (card 8), a guarantee in another package that card 6's Intent does not name as the bound.
The rubric's attack-surface rule asks the introducing card to state what the new path can skip and what bounds it.

**Fix:** State in card 6's Intent that the whole-diff form treats an empty command as an error and never passes it, or that its only bound is the told reader's never-empty contract (card 8), and pin whichever is chosen in the gate scenario.

### [MEDIUM] Card 6's tests miss two whole-diff behaviors D6 accepts on

**Location:** `_lyx/plan/06-verify-gate-told-command.md:47`

**Issue:** The whole-diff scenario asserts "the lint base, the full command and the recorded pass".
Two of D6's acceptance criteria are not pinned at the gate by any card:
"With a Publish failure record present, the darn gate runs the record's failing tests, and for a `publish_verify` failure also the `tmux` tier", and "the darn gate … stop[s] on the reader's error … and none skips its verify".
Card 12's `wiring_darn_test.go` reaches the reader's refusal at wiring time, not the gate's own call-time stop, and the landing gate's existing read-error rows (`internal/landingshed/verifygate_test.go:306`, `publish_verify_test.go:288`) cover Publish and Finalize, not the loomshed gate.

**Fix:** Add to card 6's loomshed verify gate scenario a step with a failure record present in the whole-diff form, asserting the failing tests and the `tmux` tier join the command, and a step whose told command returns an error, asserting the gate returns it with the message kept and runs nothing.

### [MEDIUM] Card 3 does not state what clearing the claim lets through

**Location:** `_lyx/plan/03-teardown-settles-board-claim.md:26`

**Issue:** Clearing the run-form status releases the `board-lock-running` guard on the entry.
D10 names both halves: what the clearing lets through (scope edits on an entry whose batten run is halted or never seeded, and the one landed case whose Finalize failed its done mark and returned Stuck on its push), and what bounds it.
Card 3's Intent gives the rule and the post-removal ordering but never says what the clear now lets through, nor that the landed-but-unmarked case ends cleared, not done, and needs `lyx board set-status`.

**Fix:** Add one sentence to card 3's Intent naming what `claimClear` lets through (scope edits on a halted or unseeded run's entry, and the push-failed Finalize case) and its bound: never while batten is `running`, never with a `Finalize` Done in the child history, and only after the session has ended and both worktrees are gone.

### [MEDIUM] The recipe-selected arm states no bound on the skip it opens

**Location:** `_lyx/plan/12-loom-arms-darn.md:38` (and `_lyx/plan/13-batten-admits-darn.md:15`)

**Issue:** From card 12 on, a seed recording `darn` makes every loom verb build a graph with no Discussion, Plan, Webster, Bouncer or Burler row, so a run carrying that seed skips every review but PR-Gate.
Card 13 opens the route by which such a seed is written.
Neither Intent states this skip or what decides which runs may take it: that the seed's recipe is written only by batten's child seeding from the board entry's operator-set `recipe` field (or by hand through `lyx shed seed`), that `lyx loom start` never rewrites a recorded recipe, and that the recipe is read once at arming.
The focus directive asks this for card 12, and the rubric requires it for every new routing edge.

**Fix:** Add to card 12's Intent (or card 13's) a sentence naming what a darn seed skips and its bound: the recipe comes only from the operator's board field through batten's closed admit set, `lyx loom start` keeps a recorded recipe, and PR-Gate stays the one review point.

### [LOW] DirtyPaths is wider than D11's tracked paths

**Location:** `_lyx/plan/04-halt-note-parent-fabric-status.md:23`

**Issue:** D11 says the section lists "the parent pair's dirty tracked paths", matching the merge guard's own `scopeTracked` check (`internal/fabricengine/mergeguards.go:110`).
Card 4 has `Fabric.DirtyPaths` return "every uncommitted path of either side", which over `Status` can include untracked files the guard never objected to, so the note could attribute the halt to a path that did not cause it.

**Fix:** Have `DirtyPaths` return the tracked uncommitted paths, the scope the dirty-worktree guard checks, and say so in the Intent and its integration step.

### [LOW] Card 13 names a test that does not exist

**Location:** `_lyx/plan/13-batten-admits-darn.md:19`

**Issue:** "the params test covers darn": `internal/battencli` has no test of `childSeedParams`; the nearest, `TestChildSeedFor_CarriesRecipeDriverAndParams`, tests `childSeedFor` with params already built.
So the card either plans an unnamed new test of an unexported helper or an assertion nowhere.

**Fix:** Name where the darn child's `parent` param is asserted: a row in the write-seed test the card already extends, or the batten lifecycle integration test that reads the child's seed.

### [LOW] The verified-tree background's opening line goes stale

**Location:** `_lyx/plan/11-darn-builder-and-routing.md:30` (and `_lyx/plan/01-verify-log-per-run.md:52`)

**Issue:** `pattern/PATTERN-verified-tree.md` opens with "Every plan-verify site runs the plan's verify command through `verifytree.Verify`."
After D6, the Darn gate, and Publish and Finalize in a darn run, run `darn.yaml`'s told command, not a plan's.
Card 11 adds the Darn gate to the site list and card 1 rewrites the failure-record clause, but neither rewords that sentence, so the background contradicts its own new site list.

**Fix:** Have card 11 reword the opening to the told verify command (the plan's in loom, `darn.yaml`'s in darn).

## Verdicts on the focus directive's questions

- **Fidelity.**
  D1-D12 and every Constraint are carried.
  D8 is on card 1, D9 on card 2, D10 on card 3 (with the attack-surface gap in F7), and D13 on cards 2, 10, 11 and 12 except the gate-slot row (F1).
  No card adds scope the record does not license: card 5's merge-base read serves D6's lint base, and card 7's reed color serves D4's color segment.
- **Card order.**
  No card uses a symbol or file a later card creates or changes.
  Cards 6, 11 and 12 add `Env.VerifyCommand`/`VerifyMergeBase`, then `DarnVerifyAttempts`, then fill them, in that order.
  Card 7's `darn` table row arming loom's graph before card 12 is harmless, since no darn seed is written before card 13.
- **Attack surface.**
  Card 11 states the budget floor and its refusal, card 10 the spent-budget halt and its way forward, card 12 the reject rework row's bound, and card 6 the Publish skip on the same tree and command.
  The open gaps are F5 (card 6's empty command), F7 (card 3) and F8 (cards 12-13).
- **Granularity and test economy.**
  The gaps are F2, F3 and F10; cards 1, 2, 4, 5, 8, 9 and 10 extend existing tests or add a file for a surface no test covers.
- **Edit cards.**
  No `Edit` card lists only non-test Go file paths where its Intent names symbols.
  Every re-signed member checked (`loomshed.NewVerifyGate`, `loomcli.landingDeps`, `loomcli.loomSeedFor`) carries the arrow; the other edited members keep their signatures in the current code.
  F4 is a wrong member, not a missing arrow.
- **Verify coverage.**
  `## verify:` runs every targeted package through `./...`, integration tier included, compiles `tmux` and `llm`, and follows the template's order; no card targets a nested module.
