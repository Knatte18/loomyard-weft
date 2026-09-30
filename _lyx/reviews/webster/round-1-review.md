---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/fabricengine/remove.go:1
    summary: The rewritten file header is hard-wrapped at a fixed column mid-phrase, violating the Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/fabricengine/livestate_verbs_test.go:564
    summary: removeCase's rewritten doc comment breaks mid-phrase at a fixed column instead of at a clause boundary
  - id: F3
    severity: LOW
    location: internal/fabricengine/mutation_record_integration_test.go:3
    summary: The rewritten file header and TestMutationRecord_RemoveDirtyWarpRefusalRecordsNothing doc are fixed-column wrapped, one with two sentences sharing a line
  - id: F4
    severity: LOW
    location: internal/fabricengine/remove_refusal_remedy_integration_test.go:3
    summary: The rewritten file header and the two new or renamed test docs are fixed-column wrapped mid-phrase
  - id: F5
    severity: NIT
    location: internal/fabricengine/refusalof_test.go:65
    summary: The RemovePreFlightDirtyError case comment still calls the Remove refusal path the anomaly path, which the diff retired
---

Review range: `_lyx/shed/archive-tag-before-dirty-check/status.json` gives `product.parent: main`.
`git merge-base main HEAD` is `724b78b50`, so the range is one commit, `015c710c2`, which implements card 1.

## Evidence gathered

- `go build ./...`, `go vet` (plain and `-tags integration`) on `internal/fabricengine`, `go test ./...` and `go test -tags integration ./...` all pass.
- Re-run with `-count=1`: every `TestRemove_*` and `TestMutationRecord_Remove*` test passes, including the three new ones.
  The 10 dirty-state Remove cells of `TestCrossProduct` pass too, at both anchors.
- `! grep -rn nameStrandedPortalTeardown --include='*.go' .` passes.

## Focus directives, settled

**Refusal order in `Topology.Remove`.**
Slug, prime, target-exists, both merge checks, the warp probe and dirty check, and `refuseDirtyWeftWorktree` all return before `archiveWeftTip` (remove.go:99-157).
`rec` is appended to only by `archiveWeftTip` and later steps, so every one of those refusals returns an empty record.
`force` gates exactly the two dirtiness blocks and nothing else.
The header and doc-comment claims match the code.
The dropped "worktree dir already gone" rationale was dead: a missing `target` returns "worktree not found" at remove.go:108.
"A failure after the archive leaves the tag in place for the re-run to reuse" matches `archiveWeftTip`'s `existing == tip` branch.
The Remove doc's "After every refusal" overlooks the destructive-gate refusals `surfaceRefusal` raises after the archive, but that wording is the plan's own ("say it runs after every refusal"), so it is not a finding here.

**Callers.**
`fabriccli`'s `fabric remove` and `battenshed`/`battencli`'s teardown match the `ErrPairSiblingDirty`/`ErrArchiveFailed` sentinels and base texts.
No production caller matches the `lyx fabric reconcile` suffix.

**Delete targets.**
`nameStrandedPortalTeardown`, `TestRemove_ResumedRemoveReusesArchiveTag`, `TestRemove_RefusalWithNothingStrandedOmitsRemedy` and `portalAndLauncherPermittedRoots` have no remaining `.go` references outside `_lyx`.
One prose reference to the retired anomaly survives, in `refusalof_test.go` (F5).

**`TestRemove_ReusesSameTipArchiveTag` pins the reuse path.**
The planted tag lives in the local weft repo, where `archiveWeftTip` looks it up.
Without the `existing == tip` skip, the unconditional `git tag <name> <tip>` fails on the existing name, the call returns `create archive tag`, and the test's `Remove error` fatal fires.
The idempotent re-push is not what the test relies on, so its coverage equals the deleted test's.

**`TestRemove_WarpStatusProbeFailurePushesNoTag` reaches the probe.**
`ReadOriginFor` reads the weft worktree and its errors are ignored.
`mergeBlocksMutation` answers `false, nil` on a `newPaired` or weft-gitdir failure.
`mergeSourceInFlight` globs the weft repo.
The test asserts `check warp worktree status` and passes, so the probe is what refuses.

**Livestate dirty cells.**
With `PermittedRoots` nil, `AssertNoUnpermittedChange(t, before, after, nil)` (livestate_matrix_test.go:240) requires a zero diff.
`AssertRecordMatchesDiff` checks the empty record against the empty unfiltered diff.
Both pass.

### [LOW] remove.go file header is fixed-column wrapped

**Location:** internal/fabricengine/remove.go:1-5

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, with further breaks only at an independent-clause boundary.
The rewritten header breaks at a fixed column mid-phrase instead: "merge-in-progress and the / no-force", "the weft / origin", "so a refused / call".

**Fix:** Rewrap lines 1-5 with semantic line breaks: one sentence per line, with a break before ", and only then" where a clause boundary helps.

### [LOW] removeCase doc comment is fixed-column wrapped

**Location:** internal/fabricengine/livestate_verbs_test.go:564-565

**Issue:** The rewritten second line breaks mid-clause, at "dirtiness checks / before its archive".
The in-switch comment at :596-597 breaks at ", so a refused cell leaves", an independent-clause boundary, so it complies.

**Fix:** Put the doc sentence on one line.

### [LOW] mutation_record_integration_test.go comments are fixed-column wrapped

**Location:** internal/fabricengine/mutation_record_integration_test.go:3-5 and :24-28

**Issue:** The file header's first sentence breaks at "a Remove / refused" and "rollbackAdd / is".
The test's doc comment breaks mid-phrase too, and line 26 carries the end of one sentence and the start of the next ("is empty. The pre-flight's").

**Fix:** Rewrap both comments with one sentence per line, breaking further only at clause boundaries.

### [LOW] remove_refusal_remedy_integration_test.go comments are fixed-column wrapped

**Location:** internal/fabricengine/remove_refusal_remedy_integration_test.go:3-5, :7-9, :27-28, :75-76

**Issue:** The header's two paragraphs, the `TestRemove_DirtyRefusalLeavesPairIntact` doc and the `TestRemove_WarpStatusProbeFailurePushesNoTag` doc all break at a fixed column mid-phrase, for example "Remove refusal — a dirty worktree or / a failed status probe" and "and asserts / the no-force refusal".
The `assertPairIntact` and `assertNoArchiveTag` docs are single sentences on single lines and comply.
Lines 13-14 are untouched legacy and out of scope.

**Fix:** Rewrap each changed comment with semantic line breaks.

### [NIT] refusalof_test.go still names the Remove "anomaly path"

**Location:** internal/fabricengine/refusalof_test.go:64-66

**Issue:** The diff deletes livestate_doc_test.go's "One known refusal-with-side-effects anomaly" section because a dirty Remove refusal no longer has side effects.
This case comment still calls the Remove pre-flight refusal "the Remove anomaly path", which now points at a retired concept with no remaining definition.
The numbered mutation-test history in livestate_doc_test.go is a record of past runs and stays, per the card.

**Fix:** Name the path as what it is now, Remove's dirty pre-flight refusal, and drop "anomaly".
