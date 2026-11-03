```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I looked for what it forces outside the lines it touches.

**Affected set, per change**

1. **`darn-template-write.md` Step 4 becomes the `{{.make_change}}` marker.**
   - Set found: the stencil header comment, `contracts/stencils/stencils.go` entries and vars, `markers_test.go`, `darnengine` `DarnTable` and its `doc.go`, and `spec_test.go`.
   - Registry: both new fragments are registered in `entries`, so the registry-driven stencil install and `stencilkit.Seed` see them.
   - Other users of the old text: no other stencil, doc or test references the darn Step 4 text, and none other uses `darn-template-write` except `internal/shedrecipe/entries_darnwrite_test.go` (lines 37 and 100), which uses it only as a seat stencil name.
   - Anchor row in `markers_test.go`: **not covered.** See the finding below.
2. **`subagent_model` moves from the writer stencil into the fragments.**
   - Set found: `DarnTable`'s values map and `markers_test.go`.
   - Marker pin: the fragments' marker sets are pinned (lines 56–57). The old `darn-template-write` marker set had no row in `markers_test.go`, so nothing stale remains.
   - `DarnInputs.SubagentModel` is still read, now through `makeChangeList`. Covered.
3. **`DarnInputs.GateFixed` and the new `wireDarn` parameter `pairSlug`.**
   - Callers of `wireDarn` and `DarnTable`: only `wiring.go:966` and the closure in `darnwiring.go:240`. The new argument is passed consistently.
   - Docs: `darnengine/doc.go` and `loomcli/doc.go` both land in the diff. Covered.
4. **PATTERN entries and `pattern/` background files.**
   - `PATTERN-gate-fixer`, `PATTERN-stencil-ownership` and `PATTERN-gate-slot` need no new text. The fragments are read at call time through `stencilstore`, and the gate-slot Verify sites are unchanged.
   - The record declares no PATTERN changes. Nothing is missing.
5. **Commit message and departures.** There is one commit and no `Pattern-Departure:` trailer. The commit message matches the diff.
6. **Other stencils that tell the darn writer to run `lym gate test`.** `grep` over `contracts/stencils/darn/` finds only the two fragments. The gated fragment has the only remaining mention inside "run no `lym gate test`". Covered.

**Findings**

- **LOW, consistency.** `contracts/stencils/markers_test.go:136-137` has anchor rows for the Go-filled `{{.self_check}}` markers of the Plan and Discussion stencils. The file header (lines 1–3) says it pins "the position of each Go-filled marker a row anchors on a heading". The new `make_change` marker has no such row, though it follows the Plan-Write pattern. Nothing pins that it still sits under `## Step 4 — Make the change`. A suggested row is `{"darn make change", "darn-template-write", "## Step 4 — Make the change", "{{.make_change}}"}`. Evidence: `markers_test.go:134-137`, `darn-template-write.md:58-60`.

Tool calls (4):
1. `grep` for "Step 4", `make_change` and `darn-fragment` references outside the diff, and for other users of the gated Plan fragment. My context lacked a repo-wide reference search.
2. `grep` for "lym gate test" in the darn and gatefix stencil dirs, plus a directory listing. Needed to check that no other darn instruction tells the writer to run the gate.
3. `grep` of `markers_test.go` and `registry_test.go` for anchor rows and marker pins of the Plan and Discussion `self_check` marker. Needed to compare with the new `make_change` marker.
4. `grep` over `*_test.go` for other users of `darn-template-write` or `DarnTemplateWrite`. Needed to find tests or fixtures affected by the stencil change.
