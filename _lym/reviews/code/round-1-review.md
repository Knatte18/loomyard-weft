---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/darnwiring.go:240
    summary: wireDarn's choice of GateFixed from the pair slug and darn.gate_fixer has no test, so a regression back to the ungated prompt with the fixer on passes every test
    fix: In internal/loomcli/wiring_darn_test.go add rows to the existing TestWire_DarnRecipe (or one new table test beside it) that wire a darn recipe, call c.env.DarnTable with a pair slug and a non-empty darn.gate_fixer and assert the prompt carries the gated no-gate step, then call it with an empty darn.gate_fixer and with no pair slug and assert the ungated step. Reuse the existing fixture; add no per-helper test of writerFixerOn, which gatefixer_test.go already covers.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/darnengine/spec_test.go:205
    summary: TestDarnSpec_Refusals comment says the table builder reads no stencil, but DarnTable now reads a Step 4 fragment, and no row covers a missing fragment
    fix: In TestDarnSpec_Refusals reword the comment above the test so it says DarnTable reads the Step 4 fragment while the seat engine's Validate reads every seat's stencil. Add two rows that remove makeChangeStencil, and makeChangeGatedStencil with in.GateFixed set, through stencilkit.Remove, each expecting an error that names the removed stencil.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/markers_test.go:134
    summary: The marker-position table anchors the Plan and Discussion self_check markers under their headings but has no row for the new make_change marker under Step 4
    fix: In the position table of contracts/stencils/markers_test.go (beside the "plan self check" and "discussion self check" rows) add the row {"darn make change", "darn-template-write", "## Step 4 — Make the change", "{{.make_change}}"}.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/darnengine/spec_test.go:105
    summary: The gated TestDarnSpec row asserts several extra lines of the fragment's static prose beyond the one line that tells the two renderings apart
    fix: In the gated row of TestDarnSpec drop "tells the subagent to run no `lym gate test`", "act on that alone" and "each leaving the tree green" from wantInPrompt and wantNotInPrompt, keeping gatedNoGateStep, ungatedTestStep, ungatedSubagentGate and the haiku model check, which are enough to pin that the two renderings differ.
---

### [MEDIUM] wireDarn's GateFixed choice is untested

**Location:** internal/loomcli/darnwiring.go:240
**Issue:** The decision record's central ask is that the prompt follows the same condition the row gate uses to attach a fixer.
That condition is evaluated only in the `wireDarn` closure, through `writerFixerOn(c.recipe, darnWriteEngine, darnGateEntry, c.env.Setting, pairSlug)`.
`wiring_darn_test.go` has no hit for `GateFixed`, `gate_fixer` or `pairSlug`, and `gatefixer_test.go` tests `writerFixerOn` in isolation, not its use here.
A wrong gate or engine argument, a dropped slug or an unset field would leave every test green while the writer runs the gate again, the very defect this change fixes.
I read the code path by hand and found the closure correct and in agreement with `splitGateFixer` and `gateFixerSeam`, so this is a coverage gap, not a defect.

### [LOW] Stale refusals comment and no missing-fragment row

**Location:** internal/darnengine/spec_test.go:205
**Issue:** `TestDarnSpec_Refusals` says "the table builder reads none", which stopped being true when `DarnTable` began reading a fragment through `makeChangeList`.
No row exercises the new error path, wrapped as "darn: DarnTable: Step 4 list", so the focus question of a usable error for a missing fragment is settled by reading only.
The error does name the stencil and the directory (`stencilstore.Read`), so it is usable.

### [LOW] No position row for the make_change marker

**Location:** contracts/stencils/markers_test.go:134
**Issue:** The file header says the file pins the position of each Go-filled marker a row anchors on a heading, and the Plan and Discussion `self_check` markers have rows.
`make_change` follows the same Plan-Write pattern but nothing pins that it stays under `## Step 4 — Make the change`.

### [LOW] Gated row repeats fragment prose in its assertions

**Location:** internal/darnengine/spec_test.go:105
**Issue:** The record asks the test to pin both renderings, which the distinguishing step lines already do.
The extra assertions on "tells the subagent to run no `lym gate test`", "act on that alone" and "each leaving the tree green" assert a stencil's static prose, which the rubric's test economy rule flags.

## Rejected

- The lens-code-correctness lens returned no findings, and my own check of the focus questions agrees: `writerFixerOn` and the row gate use the same pair-slug, recipe, engine and entry inputs, and the ungated fragment renders the removed Step 4 text unchanged.
- The focus question on fragment resolution found nothing to flag: both fragments are in the `entries` registry that installs `darn-template-write`, and `stencilkit.Seed` seeds them.
- The summary's claims were checked against the diff and hold, apart from the peer-run Preflight sentence about `seat-stencil-marker-unfilled`, which the diff neither shows nor depends on.
