---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-webster-sites-slotted.md:18
    summary: D6 status visibility covers only verifytree.Verify sites; the card-verify rerun (card 6) and the plan gate's slotted type load (card 9) wait for a slot with no waiting state any status reader sees
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-gate-cli.md:39
    summary: card 10 introduces the run-inside-the-parent's-slot path keyed on gateslot.InheritEnv but states no bound for a forged variable, the bound D4 names (D7's deny refuses agent commands naming it; an operator shell is outside the bound)
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/15-batch-gate.md:35
    summary: cards 15 and 22 plan tests for new functions (renderBatchGate, PublishFailureNote) in packages whose test-budget rows are already full, without listing cmd/lyx/testdata/test-budget.yaml
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/14-card-gate-tier1.md:8
    summary: package docs that state the old behavior are left untouched; internal/websterengine/doc.go still describes the card gate as building and testing everything with no batch-wide verify (cards 14, 15), and internal/loomshed/doc.go still says the round gate never runs the tmux tier (card 21)
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/plan/16-gate-route-stencils.md:12
    summary: card 16 leaves the text-only push line to a merge-in from main that no fork performs; the branch does not carry 0cdd14ac9, so the rewritten bullet must write that line itself
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/21-round-gate-tier3.md:29
    summary: card 21 handles a fallen-back derivation but not an empty impacted set, where Packages is empty with no fallback and the tmux pass would render as a bare go test -tags tmux over the root package
---

The plan was judged against `_lyx/discussion/decision-record.md` alone.
No `_lyx/loom/rework/` directory exists, so this is generation 0.
Every Decision (D1–D14 and the two added after Discussion) and every Constraint is carried by some card, with the gaps the findings below name.
No PATTERN constraint violation was found, so no finding is BLOCKING.

Focus verdicts that raised no finding:

- **Card order:** no card depends on a later one.
  Cards 5–9 and 14 use only what cards 1, 2 and 5 create, card 15 uses card 10's `SlotBusyExit` and card 14's `gateScope`, cards 20–22 use card 19's codec and card 21's `checkedPublishFailure`, and card 17's refusal precedes card 18's move.
- **Verify coverage:** the only nested module is `internal/planglyph/testdata/glyphchain`, which no card targets.
  `## verify:` runs every root package's untagged and `integration` tiers, vets `tmux` and `llm`, and follows the template's order.
- **Edit glyphs and re-sign arrows:** I checked every changed signature against the code (`verifyrun.Run`/`run`, `verifytree.Verify`, `loomshed.NewVerifyGate`, `rerunCardVerifies`, `cardGateCommand`, `RenderForkPrompt`, `RenderRecoveryPrompt`, `verifyWaiting`, `configcli.dispatch`, `impactset.commandFor`, `ReadRubric`, `burlerRoundProfile`).
  Each one carries its arrow.
  The arrow-less members (`websterengine.NewVerifyGate`, `landingshed.newVerifyGate`, `landingDeps`, `reviewWaitingFor`, `setModule`, `editOne`, `goListLoader.load`, `verifyGate.check`) keep their signatures, because the new value reaches them through `Geometry`, `Deps`, `configDirs` or struct fields.
- **Extra scope (cards 12, 17, 18, 23):** card 18's `SeedsFromPrime` flag is needed, not extra.
  `ReconcileHubWideAt` (`internal/configsync/configsync.go:350`) seeds from the prime only for a module with open maps, and `landing` has none, so D10's "seeds the hub file from the prime's copy" needs the flag.
  Card 12's `PATTERN-no-denied-recovery.md` edit is licensed by that constraint row.
  Cards 17 and 23 stay inside D10 and D13.
- **Attack surface:** cards 12, 13, 17 and 18 each state what passes and what bounds it.
  Card 10's unslotted outside-hub path does too.
  Only the inheritance path lacks its bound (F2).
- **Test economy:** card 1's `pool_integration_test.go` is not redundant, because no `internal/lock` test pins release on process death.
  Card 10's split between a tier-1 `cli_test.go` and a tier-2 integration test covers distinct behaviors.
  Card 15's rows are new behavior.
- **Granularity:** card 19's codec is a cross-package public surface (verifytree, read by landingshed and loomshed), so it earns its own card apart from card 20's writer.
  Cards 21 and 22 serve two surfaces (the round gate's command and the rubric note).

### [MEDIUM] Card-verify and plan-gate slot waits are invisible in status

**Location:** `_lyx/plan/06-webster-sites-slotted.md:18`, `_lyx/plan/09-plan-gate-slotted.md:31`

**Issue:** D6 decides that "a Go-side site's wait is recorded in the `verifytree` running marker as a waiting state", and its rationale is that a stall behind another run's Publish must be explainable from status without reading panes.
D4 lists the in-process card `**Verify:**` rerun and the plan gate's `caller-uncovered` `go list` as Go-side gate runs that acquire a slot.
Card 4 adds the waiting state only inside `verifytree.Verify`.
Card 6's rerun (`rerunCardVerifies` through `verifyrun.Run`) and card 9's `goListLoader.load` acquire through `Pool.Acquire` directly and record nothing a reader sees.
Card 11's status hooks read only the `verifytree` marker and `lyx gate test`'s wait records.
A record-batch or plan gate stalled behind a Publish therefore shows no waiting gate in `lyx loom status`.

**Fix:** Have cards 6 and 9 record their wait the way `lyx gate test` does: write a `gateslot.WriteWait` record under `gateslot.WaitDir(<anchor>)` before acquiring and remove it once the slot is held or the wait ends.
Add `plan:internal/gateslot#WriteWait` and `plan:internal/gateslot#WaitDir` to their `Uses:`; card 1 precedes both.
Card 11's `gateWaiting` then reports these waits, with its wording widened from "gate test waiting" to name the site.
The alternative is to write the `verifytree` marker in its waiting state from these sites.

### [MEDIUM] Inheritance escape hatch carries no stated bound

**Location:** `_lyx/plan/10-gate-cli.md:39`

**Issue:** Card 10 adds a routing edge: with `gateslot.InheritEnv` set to a slot that `Pool.Inherited` reports as held, `lyx gate test` runs without acquiring.
This edge lets a caller skip slot acquisition.
D4's Bound names what bounds it: the variable can be forged to skip acquisition while some slot is held, D7's deny refuses any agent command naming it, and an operator shell is outside the bound.
Card 10's Intent and doc.go list state the outside-hub bound but not this one.
Card 1's `Inherited` entry states no bound either.
The rubric requires the card that introduces the edge to state in its `**Intent:**` what it lets through and what bounds it.

**Fix:** Add a sentence to card 10's Intent and to its `doc.go` bounds.
It should say that a forged `LYX_GATE_SLOT` naming any held slot runs unslotted inside that holder's slot, that card 12's deny refuses an agent command naming the variable, and that an operator shell or a script hiding the name is outside the bound (D4's and D7's Bounds).

### [MEDIUM] New-function tests in packages at their budget row, with no budget edit

**Location:** `_lyx/plan/15-batch-gate.md:35`, `_lyx/plan/22-webster-review-failure.md:34`

**Issue:** `cmd/lyx/testdata/test-budget.yaml` pins `internal/websterengine: 192`, and the package has exactly 192 top-level tests.
Card 15 adds rows for the new `renderBatchGate` to `cardgate_test.go`.
That file's two tests, `TestCardGateCommand` and `TestRenderCardGates_OneLinePerCard`, drive `cardGateCommand` and `renderCardGates` only, so the rows need a new top-level test.
Card 22 adds rows for the new `PublishFailureNote` to the `publishfailure_test.go` that card 21 creates.
Card 21's budget rise covers its own tests, and a separate function's rows most likely mean another top-level test in `internal/loomshed`.
CLAUDE.md requires a card whose new top-level tests push a package past its row to list the budget file under `Edit:`, and neither card does.
`TestTestBudget_PackagesStayWithinBudget` would fail `go test ./...` at Webster's gate.

**Fix:** Add `cmd/lyx/testdata/test-budget.yaml` to the `Edit:` lists of cards 15 and 22 and state the row rise.
Alternatively, state that the new rows go into an existing top-level test (for example a `renderBatchGate` case in `TestRenderCardGates_OneLinePerCard`, and the note rows inside card 21's table).

### [MEDIUM] Package docs left describing the replaced behavior

**Location:** `_lyx/plan/14-card-gate-tier1.md:8`, `_lyx/plan/15-batch-gate.md`, `_lyx/plan/21-round-gate-tier3.md:8`

**Issue:** The decision record's Scope puts "the touched packages' `doc.go`" in the same change, and CLAUDE.md requires the module doc to follow an observable behavior change.
`internal/websterengine/doc.go:76-80` says "there is no batch-wide verify distinct from its cards' own gates".
It also describes a card's gate as one that "builds and tests everything, runs the integration-tagged tests of the card's package directories".
Card 14 replaces that gate with tier 1 of the card's packages, and card 15 adds the batch gate.
Neither card lists `internal/websterengine/doc.go`.
`internal/loomshed/doc.go:19-22` says "The round compiles the `tmux` and `llm` tiers and never runs them".
Card 21 makes the round gate run a `tmux` pass while a failure record is present, yet edits only `impactset`'s doc.
The plan's method lists doc.go only where the card author recalled a doc paragraph, so it misses docs that state the old behavior.

**Fix:** Add `internal/websterengine/doc.go` to card 14's `Edit:` list, with the card-gate paragraph rewritten to D1.
Have card 15 rewrite the "no batch-wide verify" sentence to D2, either by also listing the file or by placing both rewrites on card 14 if card 15 should not touch it.
Add `internal/loomshed/doc.go` to card 21's `Edit:` list, with the round-gate paragraph stating the record-driven tier 3 pass.

### [MEDIUM] Text-only push line left to a merge that no fork performs

**Location:** `_lyx/plan/16-gate-route-stencils.md:12`

**Issue:** The decision "CLAUDE.md push rule keeps the text-only line" requires the rewritten bullet to keep main's second line.
Card 16 says the bullet "keeps, or gains after a merge-in from `main`" that line.
`0cdd14ac9` is not an ancestor of this branch's HEAD, so the line is absent from the worktree's `CLAUDE.md`.
A Webster fork performs no merge-in.
On this wording a fork can rewrite the bullet without the line and leave it to Publish's merge-in.
That merge then meets a conflict on the same bullet, and resolving it toward the branch reverts main's line.

**Fix:** State that card 16 writes the line itself, directly under the rewritten bullet.
The line is main's wording verbatim: "A commit that changes only text no build or test reads, such as `docs/` and this file, is pushed without the test run."
That way the merge-in at Publish meets identical text.

### [LOW] Empty impacted set renders a package-less tmux pass

**Location:** `_lyx/plan/21-round-gate-tier3.md:29`

**Issue:** `impactset.buildCommand` (`internal/impactset/derive.go:188-196`) emits no test step when the impacted set is empty, as when only Markdown changed.
In that case the new `Derivation.Packages` is empty while `Fallback` is also empty.
Card 21 substitutes `./...` only when the derivation fell back.
For a `publish_verify` record with an empty set, the pass therefore renders as `go test -tags tmux` with no package argument, which tests only the root directory's package.
The card leaves that behavior unstated, and `publishfailure_test.go` has no row for it.

**Fix:** State in card 21 that an empty `Packages` with no fallback skips the impacted-set `tmux` pass and keeps the named-test steps.
Add the matching row to `publishfailure_test.go`'s command table.
