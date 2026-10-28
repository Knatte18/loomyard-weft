# Fixer report — plan, round 1

Every finding's premise held against the plan and the code; all twelve are fixed, two of them differently from the proposal.
`lym loom validate-plan` passes after the fixes, with only the informational `create-new-unit` for card 4's new test file.

## Changes

- **F1** (card 9), fixed as proposed: the Intent skips a file target whose path does not end in `.go`, as D8 holds only non-test Go code; the D8 fixture holds `internal/d/template.yaml` and a row editing only it passes; the spec text card 9 adds states the skip.
- **F2** (card 7), fixed as proposed: the `test-tier-unstated` Detail names the tags the naming lines hold, and the way forward gains the join-with-`||`-or-keep-one and drop-`untagged`-or-the-tag clause; the `tier0 tier1` and `untagged` with `tier0` rows assert the tags and the clause in the Detail.
- **F3** (cards 8 and 9), fixed as proposed, except for the done-card rows. Card 9 drops its four loomcli test targets and their sentences. Card 8 drops `TestValidatePlanCmd`, `_Overlay` and `_ReworkReportsFindingsForFirstCardMismatch` and their sentence, and keeps the `TestGateParity_PlanGate` case and the `TestPlanWriteEntry_GateConfig` row. Both cards drop their `ValidateDispatch` row and keep their `language: csharp` row. Neither card drops its done-card row, after the self-check: `seamTestFindings` and `docGoFindings` each take `plan` and `cards`, so only their own done-card row pins that each walks `cards` and not `plan.Cards` (`internal/loomshed/gates.go:259-269`).
- **F4** (card 4), fixed as proposed: the Create group names the file target `internal/loomshed/citations_integration_test.go` in place of the handles `TestValidateLoomDiscussion_Citations` and `unavailableIndex`; the Tests paragraph keeps the test, the fake and the `//go:build tier1` line.
- **F5** (card 8), fixed as proposed (option 1), covering one more row: the `internal/c_test#TestDeps` row asserts no `seam-test-unlisted` and one `glyph-not-found` for that ref. The `internal/c_test#` row shares the defect, since `lym quarry resolve 'internal/loomshed_test#'` answers `not_found`, so it makes the same assertion.
- **F6** (card 8), fixed as proposed, widened after the self-check: the reader-error sentence and the spec text card 8 adds say the halt is the parent's, and the operator corrects the marker or the unparsable test file the error names in the task worktree and re-runs `lym loom validate-plan`. The proposed wording named only a marker, but the reader also returns the `GuardParse` error for an unparsable file, which has no marker.
- **F7** (cards 6 and 5), **fixed differently**. The proposed bound says the scan skips a `NumField` or `VisibleFields` call whose receiver's type it cannot place in the module. That is not done, because the scan parses with `go/parser` and holds no type information, so it cannot know a receiver's type. Instead, a test reflecting over an out-of-module struct marks it by its import path, such as `//lym:seam time#Time`. A marker whose unit's first path segment names no directory under the scan root is not checked against a declaration. So the unmarked state's way forward ("add a marker") reaches a pass, and a marker under a module directory is still checked. Card 6 gains fixture rows for `time#Time`, which passes, and `p/missing#T`, which fails. Its "names a struct" sentence is scoped to module units. Card 5's PATTERN background text states the rule.
- **F8** (card 4), **fixed differently**: `gitrepo` offers no index read, so instead of keeping the HEAD read and recording the narrowing, card 4 creates `internal/gitrepo#Repo.TrackedRootEntries`, a go-git read of the first path segment of every index entry, staged entries included. The Path bullet reads the tracked roots through it, and the HEAD-tree narrowing bound is gone. Card 4 creates `TestTrackedRootEntries` in the existing tier1 `internal/gitrepo/worktree_test.go`, extends `internal/gitrepo/doc.go` with a bullet of its own, and drops the `CurrentSHA`, `SubdirsAtRevision` and `FilesInDirAtRevision` Uses entries. Its no-commit row becomes an empty-index row, and a row for a staged-only root entry fails. The alternative needed an Added after Discussion entry recorded through the parent, which leaves this fixer's write surface, while the index read removes the narrowing at its source.
- **F9** (card 4), fixed as proposed: the Qualified symbol bullet removes a trailing parenthesised argument list of any content; a `real.Do(x)` row passes.
- **F10** (card 3), fixed as proposed: `internal/planglyph#resolveTargets` is under Edit, and the Intent says its doc comment names its consumers as a group.
- **F11** (card 4), fixed as proposed: `internal/loomcli#loomCLI.wireLightweight` is under Edit, and the Intent says its doc comment names `PlanIndex` among the fields the validate verbs read.
- **F12** (card 3), fixed as proposed: the empty-list sentence and the `TestResolveGlyphs` row are dropped.

## Self-check

- fix-consistency, MEDIUM — card 6's out-of-module marker bound contradicts its own "names a struct" sentence and narrows D5 without a decision-record entry: fixed as proposed for the sentence, now scoped to module units; the decision-record entry is deferred (see below).
- fix-consistency, LOW — card 4's `gitrepo/doc.go` placement leaves "All three are go-git local reads" stale: fixed as proposed, `TrackedRootEntries` gets a bullet of its own and that group stays as it is.
- fix-consistency, LOW — `TestTrackedRootEntries` names no fixture helpers: fixed as proposed, card 4's Uses lists `internal/gitrepo_test#newRepo`, `#writeFile`, `#commitAll` and `#runGit`, and the Tests sentence names them.
- fix-claims, LOW — the same "All three" count: a duplicate of the fix-consistency LOW above, fixed by the same change.
- fix-proposal, MEDIUM — F3's dropped done-card rows pin each check's own `cards` iteration: fixed as proposed, both cards' done-card rows are restored.
- fix-proposal, LOW — F6's step does not fit a `GuardParse` reader error: fixed as proposed, the sentence and the spec text name the marker or the unparsable test file.
- fix-affected-set — no findings.

## Disputed

None.

## Deferred

- **F7's D5 narrowing, raised by the fix-consistency self-check.** D5 (`_lym/discussion/decision-record.md:95`) says the scan refuses a marker whose glyph names no struct. Card 6 now leaves an out-of-module marker unchecked against a declaration, and the review's own F7 fix carried the same narrowing. Recording it as an Added after Discussion entry falls to the parent, because the decision record is outside this fixer's write surface, the plan directory.
