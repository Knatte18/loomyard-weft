# Card 42 — fabric-words-outside-fabric

**Edit:**
- `internal/preflightshed/wayforward_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/websterengine/auditseverity_test.go`

**Intent:**
Fix PR #321 finding 7 for the test files: the operator's rule is that the two fabric side names belong only inside fabric, and nothing outside it (code, tests, specs, docs or skills) may use them.
This branch added occurrences in three test files; reword each without changing what any test asserts.
Card 43 rewords the spec.

- `wayforward_test.go`: the file comment's example rows become "a dirty task worktree, a `_lyx` checkout off its paired branch", and the `FabricReady` fixture reason becomes `"no _lyx worktree"`.
- `verbs_test.go` (around the fabric-sync regression test and `failingFabricOpen`):
  - read the `_lyx` repository's last commit through the anchor's `_lyx` link, `filepath.Join(h.PrimeWorktree(), h.Location.AnchorRel, "_lyx")`, instead of calling hubforge's side-named accessor, since git resolves the link to the same repository;
  - the `git log` failure and HEAD messages say "the `_lyx` repository";
  - `failingFabricOpen`'s comment and injected error say "fabric commit failed".
  The existing `WEFT_SKIP_GIT` environment variable name stays, as the finding allows.
- `auditseverity_test.go`: the linked-directory fixture's local variable becomes `lyxTarget`, at its declaration and every use.

After the edit, `git diff <merge-base> -- internal/preflightshed internal/webstercli internal/websterengine` adds no line containing either side name except `WEFT_SKIP_GIT`.

**ImpactSummary:** Renames a local variable, comments and test messages in three test files; no production code or assertion changes.
