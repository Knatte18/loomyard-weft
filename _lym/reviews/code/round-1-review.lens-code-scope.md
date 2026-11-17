```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the merge's range to the goal points of the decision record (code-scope lens).

**Per change**

- **C1: merge commit `bae0ec63`, a true merge of `main` up to `96b20f1c`.** Traces to G1 ("Run `lym fabric merge-in main`…").
- **C2: conflict resolution in `internal/beamcli/pr_test.go`.** Traces to G1, as the settlement of a listed conflicted path. The resolution drops the beam's `wantNot` field and moves the handed-over row to `wantNoErr: []string{"again"}`. It makes no change beyond reconciling the two sides.
- **C3: conflict resolution in `internal/lymdirs/doc.go`.** Traces to G1. It keeps the beam's `HandOverDirName` sentence and `main`'s `WeaverDirName` list.
- **C4: conflict resolution in `internal/reedengine/bindings_integration_test.go`.** Traces to G1. It keeps the beam's `recorderDeadline` constant over `main`'s 30-second literals, which have the same value (`bindings_integration_test.go:412`).
- **C5: commit `382a591f`, which rewrites comments in `internal/heddleengine/overview.go` and `overview_test.go`.** Traces to G2 ("fix what the merge broke, on either side").
  - Neither file changed on the beam side. All three rewritten comments arrive from `main`.
  - The darn-form verify gate lints from the merge base 0c013afa (`internal/loomshed/verifygate.go:128-129`). It therefore reads these lines as new. The first line is 124 characters against `LineLengthLimit = 120` (`internal/commentlint/linelength.go:13`). The other two blocks break after a comma with no clause-opening conjunction.
  - So the merged tree fails the gate's comment lint, and C5 is the fix the merge needs.
  - At HEAD, `lym loom lint-comments --base 0c013afa…` returns ok.
  - C5 changes comment wording only, with no refactor and no behavior change, so it complies with G3.

**Per decision**

- **G1, run merge-in and settle the conflicts:** C1, C2, C3, C4.
- **G2, make the gate pass and fix only what the merge broke:** C5. C2–C4 also keep the merged tree building: `go vet` of the four packages is clean.
- **G3, make no change the merge does not need:** a constraint, not a deliverable. Every change traces to G1 or G2, and none adds a refactor, a feature or a fix of an older fault.
- **G4, an up-to-date merge changes nothing:** not applicable, because the merge was not up to date.
- **G5, the Code-Review perch reviews the merge alone:** a constraint on review, not on the diff. Nothing is needed.
- **`Added after Discussion` entries:** the record has none.

**Findings**

None in the code-scope lens. No change goes beyond scope, and no applicable decision is left unbuilt.

The change description's claim of "no commit after it" (`_lym/landing/summary.md`) is wrong, because 382a591f follows the merge. That belongs to the handler's change-description check, not this lens, so I note it here and leave it.

Tool calls: none. My inherited context already held everything this lens needed.
