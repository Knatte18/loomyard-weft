---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/configsync/configsync.go:62
    summary: New and changed Go comments are hard-wrapped mid-sentence, breaking PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/configcli/configcli_test.go
    summary: New tests TestDispatchHubWideBoard and TestBolt_CommitWritten skip t.Parallel without touching process-global state, breaking PATTERN-test-parallel
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/bolt_integration_test.go:170
    summary: CommitWritten behaviors the diff adds have no coverage - deleted path staging, empty path list, SkipGit
  - id: F4
    severity: LOW
    class: consistency
    location: docs/shared-libs/configengine.md:165
    summary: The error-rewrapping note still says boardengine.LoadConfig matches the substring and returns the not initialized here message
  - id: F5
    severity: LOW
    class: consistency
    location: internal/fabriccli/fabric.go:308
    summary: The reconcile verb help text omits that it now heals, commits and reports the hub-wide config
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (status `product.parent` is `main`; no `_lyx/loom/rework` round exists).
`go build ./...`, `go vet ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` all pass on the reviewed head.

Settled focus questions, each a verdict of "not a defect":

- Shared Decision 1: no production site compares a module name to `fabric` or `board` to decide hub-wide handling; `ReconcileAll`, `ReconcileHubWideAt`, `configDirs.baseFor`, `menu`, `printAll` and `buildConfigLong` all branch on `HubWide`.
  The remaining name comparisons are the test pin in `configreg_test.go` and `boardcli`'s `cmd.Name() == "board"`, which is the cobra command name.
- `Bolt.CommitWritten`: the write and the commit both run under the board write lock, the commit is scoped to the returned paths through `StageAndCommit` (empty list returns `("", false, nil)`), and a failed commit leaves the file on disk as an uncommitted edit the next board sync commits.
  A deleted path stages its deletion; verified against real git in a scratch repo.
- `configcli/hubwide.go`: only the staged-to-hub copy, its strict validation and the commit run under the lock; the editor runs on the staging copy before `commit` is called.
  A concurrent edit is refused with the staging file kept, and every other exit removes it.
- `ReconcileHubWideAt` reads the prime at `<hub>/<PrimeName>/<AnchorRel>` (clone passes `res.WeftBase`), never replaces a present hub file, and `retireHubWideCopy` deletes a copy only when every leaf is present in the hub file with an equal value.
- Geometry: `configsync` and `boardengine` receive absolute paths and resolve nothing; `fabriccli`, `configcli`, `boardcli`, `loomcli` and `ideengine.Menu` resolve.
- Card 4 readers: `lyx board`, `openHubBoard` and `ideengine.Menu` all load `fabricengine.BoardDir(hub)`, and a missing hub file refuses naming the file and `lyx fabric reconcile`.
- Card 7: `PATTERN-hub-wide-config` is in `PATTERN.md` in the same change and states the implemented invariant.
- Per-card mechanical check: the one removal (`ReconcileFabricAt`, `legacyFabricConfigModules`) has no remaining caller in production or test code.
- Test economy: the retargeted `TestReconcileHubWideAt_MigratesLegacyFabricConfig` and the table's legacy row assert different things (apply and prune against fold-in seed), so neither duplicates the other; no redundancy finding is raised.

### [BLOCKING] New and changed Go comments are hard-wrapped mid-sentence

**Location:** `internal/configsync/configsync.go` (the `Result.Seed`, `Result.Retired`, `Result.Divergent` field comments, `legacyConfig`, `retireHubWideCopy`, `ReconcileAll` and `ReconcileHubWideAt` doc comments); `internal/fabriccli/fabric.go` (the comment above the `CommitWritten` call in `runReconcile`); `internal/fabriccli/clone.go` (the `ReconcileAll` comment); `internal/fabricengine/bolt.go` (header and `CommitWritten` doc); `internal/configreg/configreg.go` (header); `internal/configcli/configcli.go` (`editOne`, `dispatch` and `runConfig` docs and the `retired`/`divergent` comment).
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line for every new or changed Go comment, and these sentences continue onto a second line without an independent-clause boundary.
**Fix:** Re-break each new or changed comment at sentence ends and independent-clause boundaries, leaving untouched legacy lines alone.

### [BLOCKING] New tests skip t.Parallel without naming global state

**Location:** `internal/configcli/configcli_test.go` (`TestDispatchHubWideBoard`), `internal/fabricengine/bolt_integration_test.go` (`TestBolt_CommitWritten`).
**Issue:** `PATTERN-test-parallel` requires `t.Parallel` unless the test touches process-global state and a comment names it; both tests use only `t.TempDir` fixtures and injected fakes or a local repo.
**Fix:** Call `t.Parallel()` at the top of both tests, and in their subtests where the subtests share no mutable state.

### [MEDIUM] CommitWritten behaviors added by the diff have no coverage

**Location:** `internal/fabricengine/bolt_integration_test.go:170`
**Issue:** `TestBolt_CommitWritten` pins the scoped commit and the unwrapped write error only.
`fabric reconcile` relies on deletion staging to commit the removed legacy `warp.yaml`/`weft.yaml`, and the `SkipGit` and empty-list contracts are stated in the doc comment, yet no test reaches any of the three.
**Fix:** Add subtests to `TestBolt_CommitWritten` for a deleted path (committed as a deletion), an empty path list (no commit, `committed == false`) and `SkipGit` (write runs, nothing committed).

### [LOW] Error-rewrapping note in configengine.md is stale for boardengine

**Location:** `docs/shared-libs/configengine.md:165`
**Issue:** The note says `boardengine.LoadConfig` matches the `"not initialized"` substring and returns `not initialized here; run "lyx fabric reconcile"`.
It now uses `errors.Is(err, configengine.ErrNotInitialized)` and an absent-file check, and returns `board config <path> not initialized; run "lyx fabric reconcile"`.
**Fix:** Narrow the note to the callers that still rewrap by substring and state `boardengine`'s message and sentinel use.

### [LOW] The reconcile verb help omits the hub-wide config repair

**Location:** `internal/fabriccli/fabric.go:308`
**Issue:** `lyx fabric reconcile` now also heals the hub-wide config files, seeds `board.yaml` from the prime, commits them in `_board` and reports `hub_config`, `hub_config_detail` and `warnings`; its `Long` text names none of this, which is the observable CLI behavior `CLAUDE.md` requires documented in the same commit.
**Fix:** Add a paragraph to the `Long` text describing the hub-wide repair, the commit in `_board` and the new envelope keys.
