```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens — what the diff forces outside the lines it touches.

**Per change, the set found and whether it is covered**

1. **`ScopeRecord` gains four compare fields.**
   - Writers: `internal/driftengine/baseline.go:145` and `internal/driftengine/verbs.go:85` build a fresh `ScopeRecord`, so a new scope resets the compare fields. Covered.
   - Readers: `Verdict` and `compareText` only.
   - Docs: `driftengine/doc.go` is updated.
   - A grep for `gate_halt_at` and `halt_named_at` found no other doc or spec that lists the record's keys. Covered.
2. **Describe stencil gains the `drift_verdict` marker.**
   - Fillers: `DescribeTable`, the only caller, which always fills the marker, using `noDriftVerdict` when the verdict is empty. The `describeStencilFixture` in `describe_test.go` is updated. Covered.
   - Header comment: `landing-template-describe.md` lines 1-7 list no individual markers, so nothing is stale. Covered.
   - Beam recipe: `NewBeamDescribe` reads no stencil and needs no marker. Covered.
3. **PR-Review stencil gains the marker.**
   - Filler: `sessionSpec`, the only one, which fills it non-empty. Covered.
   - Header comment: updated.
   - Other copies of the stencil text: `stencils.go` only embeds the file. Covered.
4. **`Deps.DriftVerdict` added.**
   - Builders: `loomcli/landingdeps.go` fills it, and `envkit.go` lists it as nil-legal (commit `71d8c0a1`).
   - Consumers: Finalize, PR-Review `openPullRequest` and `sessionSpec` all go through `driftVerdict()`, which handles a nil seam. Covered.
   - Describe takes the verdict as a plain string in `DescribeInputs`, filled in the `wiring.go` closure. Covered.
5. **PR body and landing commit message gain the section.**
   - Docs: `landingshed/doc.go` and `loom-final-summary-spec.md` are updated.
   - Other readers of the body: the PR-Review refresh path goes through `openPullRequest`, so it is covered. `Finalize` passes the same body to `bodyWithCredit` and to `LandingMessage`, so both messages carry it.
   - Tests: the Finalize message row and the PR-Review body row cover both.
6. **PATTERN.md.**
   - `PATTERN-stencil-ownership`: the new stencil text restates no other producer's format. OK.
   - `PATTERN-leaf-packages` and `PATTERN-fabric-isolation`: `landingshed` does not import `driftengine`; only `loomcli` wires them. OK.
   - `PATTERN-named-constants`: `verdictHeading` and `noDriftVerdict` are each declared once. OK.
   - No declared PATTERN changes were in the directive, so nothing is owed there.
   - The commit message carries no `Pattern-Departure:` trailer, and none is needed.

**Findings**

- **F-A, LOW, `consistency`, `contracts/stencils/landing/landing-template-describe.md:39`, `internal/landingshed/doc.go:343`.**
  - The describe stencil tells the agent to drop or rewrite a hand-check item for a compare "this check settled". The writer's `summary.md` can still end with the stale "run the compare by hand" line, and the darn writer's stencil is unchanged.
  - With the Code-Review perch off, Describe is skipped. The PR then carries the stale request above the appended section, as `summary.md` itself admits ("a request for a manual compare … stays in place").
  - The board entry's Expected says a summary line asking for a check Drift made "is dropped or rewritten from the verdict". The diff only does that when Describe runs.
  - Evidence: `summary.md` design decisions, and the lack of any Go-side removal in `withSection`.
  - This is a gap against the entry's third bullet. Whether it is a defect is the handler's call.

No other member of the affected set is left wrong.

**Tool calls (3 of 6)**
1. A grep for the record keys, the stencil names and the marker names across docs, stencils and Go. My context lacked the other consumers and docs of the record, and the stencil fillers.
2. A grep of the landing stencils and the final-summary spec for the compare wording, to find other places that tell the agent to ask for a manual compare.
3. A read of the head of the describe stencil, plus a grep for the marker name in `shedrecipe`, `stencilstore` and `stencils.go`. I needed this to check that the header comment and the other marker lists were not stale.
