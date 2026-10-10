# Card 10 — status-policies-read-the-classifier

**Edit:**
- `internal/planglyph#statusFindings` -> `func statusFindings(plan *planparser.Plan, worktreeRoot string, results []quarry.ResolveResult) []Finding`
- `internal/planglyph#candidateList` -> `func candidateList(candidates []constrainedCandidate) string`
- `internal/planglyph#createFindings` -> `func createFindings(plan *planparser.Plan, worktreeRoot string, index map[string]quarry.ResolveResult) []Finding`
- `internal/planglyph#resolvePass`
- `internal/planglyph#DetectDrift`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `internal/planglyph/resolve_test.go`
- `internal/planglyph/create_test.go`

**Uses:**
- `plan:internal/planglyph#classifyAmbiguity`
- `plan:internal/planglyph#constrainedCandidate`

**Intent:** Route the resolve status policy and the Create inversion through the classifier (decision D6).
In `statusFindings`, the `ambiguous` arm calls `classifyAmbiguity(worktreeRoot, r.Candidates)`: a partitioned answer passes with no finding, like `multipart`, for every target the policy covers (Edit, Delete, a Rename's endpoints, `Uses:`); an unpartitioned one is `glyph-ambiguous` as today, its detail listing each candidate through `candidateList`.
Bound on the weakened guard: only an answer whose candidates provably never coexist in one build environment passes; the free-tag cap and every read or parse failure fall back to `ambiguous`, no tag is held true by the building toolchain, a Rename's old side never accepts several declarations (card 13), and the Delete direction of the done-check keeps blocking on any surviving declaration (card 11).
`candidateList` renders each candidate as `ID (file, constraint)`, with the file and constraint omitted when the candidate carries no file, so one renderer still serves both ambiguous arms.
`createFindings` keeps its verdict: an ambiguous Create target, partitioned or not, is `create-already-exists`, its detail now naming each candidate's file and constraint through the classifier's description.
`resolvePass` and `DetectDrift`'s post-repair revalidation pass their `worktreeRoot` through.
Update the `glyph-not-found, glyph-ambiguous, glyph-rejected` bullet and the `create-already-exists` bullet of `doc.go`: a member whose every declaration sits in files no single build environment selects together is one member, so a partitioned ambiguous answer resolves; the detail of an unpartitioned one names each candidate's file and build constraint.
In `loom-plan-spec.md`'s validation-checks preamble, where `internal/planglyph`'s emitters are named, add one sentence saying `glyph-ambiguous` is not raised for a member declared once per mutually exclusive build-constraint set, which resolves as one member.

Tests: add subtests to `TestStatusFindings` (`resolve_test.go`) over fixture repositories: a member declared in a `//go:build linux` file and a `//go:build !linux` file, and one declared in `_linux.go` and `_windows.go`, each yield no finding for an Edit, a Delete, a Rename old side and a `Uses:` reference; one declared in `//go:build linux` and `//go:build amd64` files stays `glyph-ambiguous` with both constraints named in the detail.
Update `TestCandidateList` for the constraint rendering, and `TestCreateFindings_AmbiguousIsAlreadyExists` to assert a partitioned target is still `create-already-exists` with each constraint named.
Update every `statusFindings` and `createFindings` call in both test files for the new parameter.

**ImpactSummary:** Three unexported re-signs with callers in planglyph.go, drift.go and two test files; the only verdict change is a partitioned ambiguous answer passing the status policy.
