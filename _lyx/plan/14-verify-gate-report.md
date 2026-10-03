# Card 14 — the verify-gate report and the card hint

**Create:**
- `internal/websterengine/verifygatereport.go`
- `internal/websterengine/verifygatereport_test.go`

**Edit:**
- `internal/websterengine/integrationreport.go`
- `internal/websterengine/verifyparse.go`
- `internal/websterengine/verifyparse_test.go`
- `internal/websterengine/runlevel.go`

**Uses:**
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/state.go`

**Intent:**
D4: the findings and the per-run gate report the webster verify gate (card 15) writes on a failed evaluation, and the touch-based card hint that replaces bisect.
No caller writes the report yet.

The parser's failure identity gains `Package`, the import path `parseVerifyFailures` already knows for a test or package identity and empty for the opaque one, so the hint never splits an identity string.

`verifygatereport.go` declares:

- `VerifyGateReportPath(reportsDir string) string`: `verify-gate.yaml` in the reports dir, replacing `integration.yaml`.
- `VerifyGateReport{Attempt, Cap int; Dirty []string; Failures []IntegrationFailure; LogPath string; Hint []string; FixCommits []string}`, with a writer that replaces the file atomically; `Dirty` carries a clean-tree failure's paths, which run no verify and so have no identities or log.
- `cardHint(modulePath string, failures []IntegrationFailure, shas, labels []string, changedPaths func(sha string) ([]string, error)) ([]string, error)`: each failing package's import path maps to a worktree directory through `go.mod`'s module path, and a label is hinted when one of its SHAs' changed paths names a file directly in such a directory.
  The gate passes `commitChangedPaths` over its worktree as `changedPaths`, so the hint itself spawns nothing.
  The trail is `accumulatedCardSHAs`, which pairs every recorded `CardSHAs` commit with its `NN-slug` label; it stays in `runlevel.go`, and its doc comment names the hint as its consumer in place of bisect.
  An opaque identity, or a package outside the module, contributes no hint.
- `renderVerifyGateFindings(r VerifyGateReport) string`: the findings text the gate returns, naming the attempt and the cap, then either the dirty paths or each failing identity with its output tail, the full log path, and the hint (or that no recorded card touched a failing package).

The hint never claims more than it knows: it lists every card whose commit touched a failing package, in trail order, and Merriam judges it with the plan in hand.

Tests in `verifygatereport_test.go` (tier 1, a fake `changedPaths` and a temp dir):
the hint names the card whose commit touched a failing package's directory and leaves out one whose commit touched only another package;
an opaque failure yields no hint;
the report written for two identities, a hint and two fix commits reads back equal;
the findings text names the attempt, the cap, each identity, the log path and the hint, and for a dirty report names the paths.
`verifyparse_test.go` pins `Package` on a test, a package and an opaque identity.

**ImpactSummary:** additive within websterengine: the identity type gains one field every existing reader ignores, and the new report, hint and findings helpers have no caller until card 15.
