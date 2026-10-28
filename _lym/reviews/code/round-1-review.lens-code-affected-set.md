```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set I found, and whether the diff covers it:

- **`planindex.Index.ResolveGlyphs`**: the implementers are `planglyph.index` and the test fakes. The `envkit.noFindingsIndex` fake gained the method, and `unavailableIndex` in the loomshed tests is covered. Covered.
- **`NewDiscussionGate` and `ValidateLoomDiscussion` signature** (new `worktreeRoot` and `index` arguments):
  - Covered: the `shedrecipe/entries_gate.go` resolver, the `loomcli` verb, and all test callers (`gates_test`, `gatefindings_test`, `smoke_gate_test`, `parity_test`).
  - Covered: the `shedrecipe.Env` doc comments, `gates.go`, and `loomcli` help and `doc.go`.
  - **Not covered: `pattern/PATTERN-gate-slot.md:28`. See the finding below.**
- **`planContextFindings` signature and the `PlanContext` fields `TierTags` and `SeamTests`**: both builders (`loomcli/validate.go:279` and `shedrecipe/entries_gate.go:127`) fill them, and `loomcli/wiring.go` and `shedrecipe.Env.TierTags` carry the tier tags. Covered.
- **`//lym:seam` marker**: the PATTERN.md line lands with the declared text, and the `pattern/PATTERN-test-speed.md` section lands with the stacking rule. `impactset` `doc.go` and `GuardScanError` comments are updated. Covered.
- **Specs and stencils**: the discussion spec has the new sections. The plan spec has checks 54 to 56 and the context-check order line. The Discussion and Plan writer stencils each gain one pointer sentence. The chair stencil gets it through its include, and the darn stencil is unchanged, as D10 says. Covered.
- **`Pattern-Departure` trailer (commit `fbbe1de3`)**: the `PATTERN-leaf-packages` background (`pattern/PATTERN-leaf-packages.md:8`) does pin `gitkit.CopyRepo` to `lymcwd`, so the trailer names an entry that forbids that way. Valid.
- **`docs/overview.md:497`** lists the `ValidateLoomDiscussion` consumers only, which the diff doesn't change. No edit needed.

**Findings**

1. **LOW, `consistency`: the PATTERN-gate-slot pairs line is stale.** It is at `pattern/PATTERN-gate-slot.md:28`.
   - It describes `loomshed.ValidateLoomDiscussion` as `discussionparser.Validate` plus `discussionparser.CheckRelevant`.
   - The diff makes it also run `CheckDecisionIDs` and the citation check, and makes the Discussion gates depend on quarry through the index.
   - Line 29, `ValidateDarnDiscussion`, names no parts, so it stays accurate.
   - The line sits in a background file of an entry the diff touches (`PATTERN-gate-slot`'s gate/verb-parity invariant), and it was not updated in the same diff.
   - CLAUDE.md's "Docs land in the same commit" and the rubric's "repository's own rules" apply.
   - Fix: add `discussionparser.CheckDecisionIDs` and the citation check to the parenthetical.

Tool calls (3):
1. A grep for the `Index` implementers, the callers of `NewDiscussionGate`, `ValidateLoomDiscussion` and `planContextFindings`, `PlanContext` builders, docs naming the checks, and the `leaf-packages` gitkit text. My context lacked the full caller and implementer set.
2. A listing of the doc files plus greps of the doc comments and md files for the stale `ValidateLoomDiscussion` descriptions. My context lacked the `doc.go` and PATTERN background text for these.
3. A read of `PATTERN-gate-slot.md:24-34`, `docs/overview.md:497` and `loomshed/doc.go`. This confirmed the stale line and that the `overview.md` and `doc.go` text needs no change.
