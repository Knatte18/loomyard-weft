---
verdict: APPROVED
---

## Alignment at PR-Gate — hub-workspaces

Pull request 753 at `214371241` against `main` (merge base `82cd60938`, which is `main`'s tip) matches the board entry's goal and the decision record's D1 to D8.

### Goal and decisions

- D1: `HubWorkspacePath` returns `<hub>/<RepoName>.code-workspace`, declared in `fabricengine` alone.
- D2: `hubWorkspaceFolders` lists the prime's anchor named after its branch, or after its directory on a detached HEAD, then `_board` as `Bolt`, then every task pair at its anchor, sorted by slug.
  A pair that does not resolve is skipped at Debug, and a missing directory is left out.
- D3: one function, `RegenerateHubWorkspace`, used by the verbs and by `ideengine`'s prime spawn, which discards its recorder and returns its errors; the spawn's double write and its own builder are gone.
- D4: `Clone`, `Add` and `AddLocal` (through the shared `add` body, past the pre-flight), `Remove`, `Prune` with apply, `Checkout` and `Reconcile` regenerate; `regenerateAfterVerb` logs a failure at Warn naming `lyx fabric reconcile` and never returns it.
- D5: the leaf lock `<hub>/_board/.lyx/hub-workspace.lock` with a 5-second wait, taken only when `_board` exists.
- D6: no production code creates, restores or removes a portal any more; Reconcile reports `launchers_restored` and skips a local-only pair.
- D7: `sweepRetiredHubArtifacts` runs in Reconcile before the regeneration, and `HubDetails` carries the kept entries.
- D8: help text, docs, the sandbox suite and the PATTERN background are updated.

### Carried-over risks

- Discussion-Review `portal-restored-launcher-only-restore-unreported`: fixed by D6's `launchers_restored`, which the code implements.
- Plan-Review `card1-row7-folder-change-via-verb`: fixed in the plan's test row, a test-arrangement point with no scope effect.
- Merge-Review `hub-containment-oneliner-missing-declared-clause`: fixed in `214371241`.
  The `PATTERN-fabric-write-containment` line is shorter than the record's declared text so that `PATTERN.md` fits its headroom line, and it keeps the declared meaning: a hub `os.Root`, never raw `os` or `fslink`.
  This is a wording difference and breaks no recorded decision.

The record's open risks (a vanished folder until the next regeneration, an ignored Warn, unlocked regeneration on a hub with no `_board`, UI edits lost) are bounds the decisions accept, and the diff stays inside them.

### Overlap

- `ubuntu-2404-support` (row 9; this run's `own_row` is 21): text-only overlap.
  Its D7 adds a lock-wait test seam to `internal/fabricengine/export_test.go`, and its D2 rewrites the `dropSeedCommits` paragraph of `internal/fabricengine/doc.go`; this run changes both files at other places.
  Its prime-first rule concerns reed's tmux session row, not the workspace file.
  It adds no PATTERN entry, so no PATTERN line conflicts.
  Landing order: this run is further along by its recipe row, so it lands first, and `ubuntu-2404-support` takes these changes at Merge-In.
- No parent commits are listed.
- Shared bookkeeping: `docs/overview.md` and the `PATTERN.md` lines for `PATTERN-hub-containment` and `PATTERN-fabric-write-containment` change; no peer touches them.

### Superseded

None.
The entries that name workspaces (`orch-any-worktree`, `umbrella-daemon`, `pair-from-parent`, `umbrella-anchor`) keep work this run rules out of scope.
