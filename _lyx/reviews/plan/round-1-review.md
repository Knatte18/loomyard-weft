---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-mark-guards.md:62
    summary: The lyx guard marker and the testtiming keep marker must each sit directly above the func line, so a test carrying both breaks one tool, and no card says how they coexist
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-tmux-tier-green.md:8
    summary: Card 11 rests on a false premise, since every dot-fill harness caller already pins a 140x42 window and the test passes here, so its prescribed fix is a no-op and the real cause is unnamed
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/07-lint-comments-verb.md:26
    summary: Card 7 says the verb takes no arguments in the line after it declares the --base flag
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/07-lint-comments-verb.md:22
    summary: The decision record's PATTERN-sandbox-coverage constraint for the new verb is carried by no card, and the plan never says loom stays covered at module level
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/07-lint-comments-verb.md:27
    summary: The Pairs list in the PATTERN-gate-self-check-parity background is not updated for the lint verb and gate pair, nor for the Webster-Burler gate's changed body
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/10-round-gate.md:32
    summary: Card 10's bound omits that the round gate only compiles the tmux and llm tiers and what catches a tmux-only failure, which D2 states as part of the bound
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/12-publish-verify.md:27
    summary: Card 12's bound says the key runs only at Publish but not that a landing into a parent outside require_pr_to_base therefore runs no tmux tier, which D4 states
---

### [BLOCKING] Two markers cannot both sit directly above the same func line

**Location:** `_lyx/plan/09-mark-guards.md:62`, with `_lyx/plan/08-impact-set-engine.md:37`
**Issue:**
Card 9 puts `//lyx:guard` "on the line directly above its `func Test…` line, the way `//testtiming:keep` sits above a kept test", and card 8's derivation returns an error for a marker "not directly above a top-level `func Test…` line".
`cmd/testtiming/keep.go` (`scanKeeps`) applies the same rule to `//testtiming:keep`: it reads only `lines[i+1]` and errors with "not directly above a top-level test function" otherwise (`cmd/testtiming/keep_test.go` pins two stacked keeps as that error).
Many of card 9's targets already carry a keep marker on their enforcing tests (for example `cmd/lyx/sandbox_coverage_test.go`, `cmd/lyx/tierpurity_test.go`, `internal/lyxcwd/enforcement_test.go`, `contracts/stencils/claims_test.go`).
Whichever marker the implementer puts nearer the func, the other is no longer directly above it.
Guard below keep: `cmd/testtiming`, which card 15 runs for the after-state, fails on the file.
Guard above keep: card 8's derivation fails, and card 10 turns that into a failed round gate on every round.
Neither card, the decision record nor the plan names the coexistence, and no card edits either parser.
**Fix:**
Settle the rule in the plan: a marker counts as directly above the func when only other directive lines (`//lyx:` or `//testtiming:`) separate it from the func line, in either order.
Card 9 adds `cmd/testtiming/keep.go` and `cmd/testtiming/keep_test.go` to its `**Edit:**` list, with a row for a keep above a guard, and its Intent and the `PATTERN-test-speed.md` background state the rule.
Card 8's Intent states the same rule for its parser, and `guards_test.go` gains a row for a guard stacked with a keep.

### [MEDIUM] Card 11's diagnosis does not hold

**Location:** `_lyx/plan/11-tmux-tier-green.md:8`
**Issue:**
The card says the harness window is too small for the split and asks to "give the harness the explicit window size the split needs, the way the other subtests pin theirs".
`newDotFillHarnessOn` already creates the harness with an explicit `-x`/`-y`, `CrossClientControl` already calls it with 140 and 42 like the other three callers, and the harness already sets `window-size manual`.
`go test -tags tmux -run 'TestSmokeDotFill$' ./internal/reedcli` passes on this machine (tmux 3.6), including under a pty of 8 rows by 40 columns.
So the prescribed change is already in place, the cause of the failure `docs/benchmarks/test-suite-timing.md` records on the measuring machine is unknown, and D4's ordering (card 12 after the tier is green) rests on this card.
**Fix:**
Reword the Intent: the implementer first reproduces the "no space for new pane" failure under the condition the benchmark document records (that machine, or the full `go test -tags tmux ./...` run), finds the cause, and fixes that in the test file.
Drop the claim that the harness lacks a pinned size, and keep the sentence that any other failure of the tier is this card's to fix.

### [LOW] Card 7 contradicts itself on the verb's arguments

**Location:** `_lyx/plan/07-lint-comments-verb.md:26`
**Issue:**
The Intent declares `lyx loom lint-comments [--base <commit>]` and in the next sentence says "It takes no arguments".
**Fix:**
Say it takes no positional arguments, and that `--base` is its one flag.

### [LOW] PATTERN-sandbox-coverage is carried by no card

**Location:** `_lyx/plan/07-lint-comments-verb.md:22`
**Issue:**
The decision record's Constraints require the new verb to be exercised by the sandbox suite or excluded with a reason.
`cmd/lyx/sandbox_coverage_test.go` is module-keyed and `loom` is already covered by the core suite's S8 (`**Covers:** loom`), whose own note says a verb gap inside a covered module is never fixed through the exclusion list.
No card says this, so an implementer may add a wrong exclusion or a needless scenario.
**Fix:**
Add one sentence to card 7: `loom` stays covered at module level by the core suite's `**Covers:** loom` scenario, so the verb needs neither a suite row nor an exclusion.

### [LOW] The gate-parity background lists pairs and misses the lint pair

**Location:** `_lyx/plan/07-lint-comments-verb.md:27`
**Issue:**
`pattern/PATTERN-gate-self-check-parity.md` has a Pairs list naming each gate with its verb and shared function.
Card 7 and card 10 add the card gate, the round gate and `lint-comments` sharing `commentlint.Lint`, and change what Webster-Burler's `verify` gate calls, yet neither edits that background.
The Scope line "Docs and PATTERN updated in the same commits as the code" licenses the edit.
**Fix:**
Add `pattern/PATTERN-gate-self-check-parity.md` to card 7's `**Edit:**` list, with a Pairs entry for the lint pair, and have card 10's PATTERN edit adjust the Webster-Burler `verify` gate's line.

### [LOW] Card 10's bound leaves out the compile-only tiers

**Location:** `_lyx/plan/10-round-gate.md:32`
**Issue:**
D2's bound also says the round gate never runs the `tmux` or `llm` tier and only compiles them, that a failure only a `tmux` test shows is caught where `publish_verify` runs the tier, and that an `llm`-only failure passes every gate.
Card 10's Intent bound names only the fallbacks and the landing gates, so a reader of the card cannot tell that every impacted package's `tmux` tests are skipped in a round.
**Fix:**
Add those two sentences to the bound in card 10's Intent.

### [LOW] Card 12's bound leaves out the Publish-only consequence

**Location:** `_lyx/plan/12-publish-verify.md:27`
**Issue:**
D4 states that Publish is a no-op for a parent outside `require_pr_to_base` and Finalize never runs the key, so such a landing runs no `tmux` tier and only Finalize's plan verify gates it.
Card 12 says only "it runs only at Publish".
**Fix:**
Add that consequence to the bound in card 12's Intent.

## Focus questions settled

- **Fidelity:** every Decision (D1–D11) and Constraint is carried by a card, except as filed in F4 and F5. Open risks and the bounds under D2, D4 and D5 are carried, except as filed in F6 and F7. No card scope lacks a licence: card 9's `cmd/testtiming` edit, which F1 adds, is licensed by D5 and the Scope acceptance criterion.
- **Attack surface, cards 8, 10, 12:** card 8 states its fallback list and bound; card 10 and card 12 are short of D2 and D4 as filed in F6 and F7. The trust-class statement in card 12 is present.
- **Attack surface, cards 9 and 5:** card 9 states what an unmarked scanning test lets through and that the full plan verify catches it at Publish or Finalize. Card 5 states the allowed-list reason requirement and the stale-entry failure, which bound an allowed-list edit; not a defect.
- **Sequencing:** 11 precedes 12, and 2 precedes 3 and 4. Not a defect.
- **Verify coverage:** `## verify:` builds and vets every tag and runs the untagged and `integration` tiers over `./...`, so every targeted package's tests run except `tmux`-tagged ones. Those are compiled by `go vet -tags tmux`, run by card 11's `**Verify:**` and by Publish, as D4 designs. Not a finding.
- **Granularity and test economy:** no per-symbol split and no test duplicating existing coverage. Card 7's verb test and card 6's engine test pin different surfaces (the JSON envelope against the findings), and card 10's gate test pins the gate.
- **ImpactSummary:** every `Edit` and `Delete` card carries a one-line blast-radius conclusion; none merely restates its Intent.
- **Measured claims checked:** the before count of 44 test binaries linking tree-sitter reproduces (`go list -test -deps -tags integration,tmux,llm`), and `configengine.Load` keeps a file key its template lacks, so the new `publish_verify` key does not break this run's Publish.
- **Card 9's file list:** several targets scan only their own package directory and card 9's own rule leaves them unmarked; over-inclusion of a candidate file is harmless, so not a finding.
