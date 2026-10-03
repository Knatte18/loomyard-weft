# Card 7 — webster-git-helpers

**Edit:**
- `websterengine/auditseverity_test.go`
- `websterengine/beginbatch_test.go`
- `websterengine/gitwrap_test.go`
- `websterengine/integration_test.go`
- `websterengine/integrationfix_test.go`
- `websterengine/parentmerge_test.go`
- `websterengine/rebaseline_test.go`
- `websterengine/recordbatch_test.go`
- `websterengine/recoverbatch_test.go`
- `websterengine/runlevel_test.go`
- `websterengine/suspect_test.go`
- `webstercli/acceptaudit_test.go`
- `webstercli/sync_integration_test.go`
- `webstercli/verbs_test.go`

**Uses:**
- `gitkit/query.go`

**Intent:**
Fold the webster pair's git scratch-repo helpers into `gitkit` (#324 §6, D4), which needs card 2's helpers.

Delete `mustGit` and `commitFile` in `websterengine/beginbatch_test.go`, their `gitwrap`-prefixed twins `gitwrapMustGit` and `gitwrapCommitFile` in `websterengine/gitwrap_test.go`, and `mustGit` and `commitFile` in `webstercli/verbs_test.go`; callers use `gitkit.Git` and `gitkit.CommitFile`.
`newScratchRepo`/`gitwrapNewScratchRepo` are scratch-repo initialisers rather than git plumbing: each package keeps at most one, a few lines over `t.TempDir()` and `gitkit.Git`, and the `gitwrap` copy merges into the `websterengine` one wherever the external/internal package split allows.

Remove `beginbatch_test.go`'s comment that keeps the helper "package-local rather than shared, since test-helper packages are deliberately not shared across modules" (D1).
Every touched file is integration-tagged, and both packages already run under the hermetic `TestMain`.
Assertions are unchanged; apply D10 to every touched test.

**ImpactSummary:** Integration-tagged websterengine and webstercli tests only; six private git helpers deleted, callers routed through gitkit.
