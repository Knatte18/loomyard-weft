# Card 14 — webster-audit-way-forward

**Create:**
- `internal/websterengine/findingtext.go`
- `internal/websterengine/findingtext_test.go`

**Edit:**
- `internal/websterengine#RunDeps`
- `internal/websterengine#runExitAuditCrossCheck`
- `internal/websterengine#pendingAuditFindingsError`
- `internal/websterengine#pendingPathsWayForward`
- `internal/websterengine#AcceptPendingAudit`
- `internal/websterengine#freshPendingDrop`
- `internal/websterengine#headBeforeEveryStart`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#uncheckableSuspects`
- `internal/shedadapters#NewWebsterProducer`
- `internal/shedadapters#WebsterProducer.Call`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `internal/shedadapters/webster_test.go`

**Uses:**
- `internal/websterengine#wayForwardSteps`
- `plan:internal/websterengine#contractFileStatus`
- `internal/websterengine#checkSuspectPaths`
- `internal/websterengine#trackedRel`

**Intent:**
D8, and D7 for the audit and `--fresh` sites: the run-exit audit's stuck reason and the pending-findings refusal state each finding once, say why an uncheckable path cannot be checked, and end in one ordered command list that names `lyx webster reset` wherever a reset is needed (issues #362 and #365).

`findingtext.go` holds two pure builders the sites share:
- the findings clause: "N correctness finding(s): 1) <class>: <detail>; 2) ...", each finding once with its path, an uncheckable path carrying its reason in place of the separate "suspect paths:" list, which is dropped;
- the reason for a path that cannot be checked: "under `_lyx`, outside the task worktree's tracked tree", "under `.lyx`", "under webster's scratch directory", "outside the task worktree", "ignored by git", "no plan copy recorded", "the finding names no path".
  The classification mirrors `checkSuspectPaths`' cases; `uncheckableSuspects` and the sites below return the reason with each path, replacing the bare "cannot be checked" and the truncated "since nothing the run recorded can check <paths>" clause.

`pendingPathsWayForward` returns the ordered steps rather than prose, rendered once through `wayForwardSteps`:
restore steps first (`git checkout <last batch head> -- <paths>` for differing tracked paths, `lyx webster restore-plan` for plan paths, `rm <path>` for an uncleared contract path),
then `lyx webster accept-audit`,
then exactly one re-entry step;
when any finding has no path or an uncheckable path that is not a cleared contract path, the whole list is `1) lyx webster reset --to start; 2) lyx webster run --fresh`, the fresh run being the re-entry.
No step restates another, and the reset step precedes the re-entry.

`RunDeps` gains `ReentryStep string`, the plain re-entry step; empty means `lyx webster run`.
`NewWebsterProducer` sets it to "re-step the <row> row" when empty, and `WebsterProducer.Call` stops appending its own ", then re-step the row" to the pending-findings reason, since the list already ends in the row's re-entry; the fresh route's re-entry stays `lyx webster run --fresh`, which the adapter never runs itself (`Fresh` is fixed false).

The texts:
- `runExitAuditCrossCheck`'s stuck reason: "run-exit audit found <findings clause>; way forward: 1) ...; 2) ...".
- `pendingAuditFindingsError`: the same shape, prefixed by `ErrPendingAuditFindings`.
- `AcceptPendingAudit`: each refusal part names its paths once with their reasons and its own step list.
- `freshPendingDrop`'s differing-path and HEAD-not-at-start refusals, `headBeforeEveryStart`'s diverged-starts refusal (no more `git merge-base --octopus`), and `RecoverSpawnOrAttach`'s uncheckable refusal each name `1) lyx webster reset --to start; 2) <their re-run>`.

`refusal-spec.md`'s rows for the run-exit audit, pending findings, accept-audit (finding not checkable, suspect path differs), recover-batch (findings recovery cannot check), the audit classes, and the three `--fresh` refusals quote the new texts.

Tests:
`findingtext_test.go` (untagged) pins, as golden text, the stuck reason for the `board-model` findings: a fork-contract-write on both contract files (one cleared, one not) plus a parent write under `_lyx`, each named once, each uncheckable path with its reason, ending in the numbered list;
the reason builder covers each case.
Grep the tests for every old text these sites emitted ("suspect paths:", "cannot be checked", "nothing the run recorded can check", "with git and run", "merge-base --octopus", ", then re-step the"), and update each assertion to the new text.

**ImpactSummary:** audit and fresh refusal texts are reworded to one numbered-step shape and the shed adapter stops appending its own step; no refusal changes when it fires.
