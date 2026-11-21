```yaml
lens: code-scope
outcome: reported
```

code-scope lens: trace each change in the diff to a goal point or an Added-after-Discussion entry of the decision record.

**Decisions**
- G1 (Goal, Seen): the red-first gate could not judge a diff that also changed a non-Go file, because the overlay could not put that file back, so the run fell back to a "No test seam" waiver.
- G2 (Goal, Idea): the overlay restores what the check needs, or puts back any tracked file, so a doc or spec change beside a fix no longer waives the gate.
- A1 (Added after Discussion, parent, 2026-11-21): waive red-first for this run and keep the stencil edit to `contracts/stencils/darn/darn-template-write.md`.

**Changes → decision**
- C1. Overlay logic, which traces to G1 and G2.
  In `internal/gatecli/redfirst.go`, `baseOverlay` now takes a base `redundancyTree`, and the new `embeddedAtBase`, `embedPatterns`, `embedPatternMatches` and `goEmbedDirective` find embedded files.
  `holdsGoFileAtOrAbove` and the unrestorable refusal in `redFirstCall.run` are removed.
  It follows the first route the Idea offers: restore only what the base build reads, and leave other non-Go files as they are.
- C2. Help text, file header and `runRedFirst` doc in `redfirst.go`, `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `internal/gateslot/redfirst.go` and `docs/overview.md`: forced by C1, whose removed refusal makes them stale.
- C3. The exit-5 text in `internal/loomshed/redfirstgate.go` `redFirstFailure`: forced by C1, since the unjudged exit now has one cause.
- C4. The stencil `contracts/stencils/darn/darn-template-write.md:111`: forced by C1 and licensed by A1.
- C5. Tests: `redfirst_test.go` rewrites the `TestBaseOverlay_…` table, `redfirst_integration_test.go` gets two `TestGateRedFirst` rows and an embedding `spec` fixture, and the loomshed test row changes. All forced by C1 and C3.
- C6. Commit 2bf05f59 rewraps comment lines in `gatecli/doc.go`, `redfirst.go`, `redfirst_test.go` and `gateslot/redfirst.go`. All four files carry C1/C2 edits, so this is the gate's comment-lint fix of lines C1/C2 changed, forced by them.
- C7. Commit fe4aa057 pins the loomshed test's exit-5 text: forced by C3.

**Decision → changes**
- G1: C1, with C2–C7 following from it.
- G2: C1. The integration row `ANonGoFileNoProductionFileEmbedsLeavesTheVerdictToTheTests` shows a non-embedded non-Go change no longer forces unjudged.
- A1: C4. The waiver itself is the parent's record and needs no diff change.

**Findings**
None. Every change traces to G1/G2/A1 or is forced by a traced change, and every decision has a change that builds it.

One note, not filed: the Idea's wording "leaves non-Go files that no test reads at HEAD" would also restore files production code reads at run time. The diff instead leaves them at working-tree text and names that bound in `internal/gatecli/doc.go` (the new "Bound:" line). The Idea offers alternative routes rather than a requirement, and this bound tends towards the green exit rather than a false red. I don't count it as a scope gap.

Tool calls:
1. `git show --stat 2bf05f59 fe4aa057`: the manifest showed these two commits' messages but not which files they touch, which I needed to rule out unrequested scope.
