---
verdict: APPROVED
---

## Alignment at pr-gate: fork-git-guard-read-only-tag

The diff of pull request 838 at `3ac081763` against `main` changes `internal/burlerengine/cluster.go`, `cluster_test.go` and `doc.go` only.

### Measure

The board entry's operator decision (2026-10-22) wins over the imported issue body: the subcommand-only guard stays, read-only `git tag` forms (`--points-at`, `-l`/`--list`, `--contains`) pass, and a `git tag` that creates or deletes a tag stays refused.
The decision record has no `Added after Discussion` entry, so the operator decision is the amended entry.

- `commandRunsMutatingGit` now lets a `git tag` through only when `gitTagListsOnly` finds it bare or in list mode with no tag-writing option; every other subcommand keeps the subcommand-only rule.
- The named list forms pass, and `--no-contains`, `--merged`, `--no-merged` and `-n` pass too; the issue body names these, and git documents each as implying list mode, so this sits inside the entry.
- Creating and deleting forms (`git tag v1.0`, `-d`, `-a`, `-s`, `-f`, a writing option beside a list option, `--list` after `--` or as a `-m` value, a short-option cluster) stay mutating.
- `TestCommandRunsMutatingGit` gains a row for the issue's exact command line, as the issue's "Add a test" asks, and the read-only and mutating `git tag` rows; no existing row changes.
- The `doc.go` fork-audit paragraph states the `git tag` exception.

The issue body's other points are settled by the operator decision, which scopes the change to `git tag`: `branch`, `stash`, `remote` and `config` stay as they are, and the record's open risk names the `branch`/`stash` flaw as left out by that decision.
The halt message stays unchanged, which the operator decision does not call for changing.
No change falls outside the entry.

### Open risks

The one carried open risk, `git branch --list` and `git stash list` still halting a fork, names no unrecorded parent correction; the operator decision on the board entry excludes them.

### Overlap

- `lyx loom peers` lists no parent commits.
- Peers `orch-strand-cycle`, `loop-exit-halt-names-its-cause`, `retry-archive-tag-push` and `done-check-spaced-create-paths` change no file in `internal/burlerengine`, and none touches the fork git guard; no overlap, so no landing order applies.
- No other board entry covers the fork git guard, so this run supersedes none.
