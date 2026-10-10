---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:36
    summary: PATTERN.md is 16430 bytes after the hub-store-gc line grew, 46 bytes over the 16384 cap, so TestOverview_RealFilePassesCheck fails the Publish verify
  - id: F2
    severity: LOW
    class: design
    location: internal/hubreconcile/walk.go:209
    summary: reconcileHubWide pushes the board on every walk, and a board with no upstream can never push, so its stamp stays absent and every start verb re-walks with a Warn
  - id: F3
    severity: LOW
    class: scope
    location: internal/fabricengine/bolt.go:219
    summary: pullFirst's no-upstream and fetch-failed-without-upstream branches, documented as writes proceeding as before, have no test through PullThenCommitWritten
  - id: F4
    severity: LOW
    class: consistency
    location: internal/fabriccli/fabric.go:759
    summary: runReconcile's PullThenCommitWritten write returns pruned legacy config paths, which Bolt records as file_written in the verb's mutations although those files are deleted
  - id: F5
    severity: LOW
    class: consistency
    location: internal/fabricengine/stencilcommit.go:1
    summary: stencilcommit.go's file header and the BoardWriteLockFile and subtree-helper comments still describe CommitSeededStencils as the pre-run's seeding commit and say seeding does not build on Bolt, which the diff made false
  - id: F6
    severity: NIT
    class: consistency
    location: internal/fabricengine/stencilcommit.go:163
    summary: underSubtree has no doc comment although its siblings and the surrounding package document every helper
---

Range: `product.parent` is `main`, no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which is `8003f2d88..HEAD`, plus the merge-in's own diff for the Publish failure.
`go vet`, build and every package but `internal/pattern` are green in `.lyx/verify/publish-failure.log`.
The per-card mechanical checks that apply are the `Delete` group of card 10 (`revisionLabel`, `shortRevisionLength`: no remaining callers in the tree) and the `Create` groups' symbols, all present.
No rubric `Do not flag` item is raised below, and no forbidden literal (`_lyx`, `.lyx`, `-HUB`, `os.Getwd`, raw `git`) appears in the added non-test code.

### [BLOCKING] PATTERN.md exceeds the overview size cap

**Location:** `PATTERN.md:36`, `internal/pattern/check.go:21` (`MaxOverviewBytes = 16 * 1024`)
**Issue:** `main`'s `PATTERN.md` is 16380 bytes, 4 under the cap, after its `PATTERN-gate-slot` line landed.
Card 12 rewrites the `PATTERN-hub-store-gc` line to name the in-flight skip, which adds 50 bytes; the branch then merged `main` in (`6f3760049`), giving 16430 bytes, 46 over.
`TestOverview_RealFilePassesCheck` reports `oversize-overview PATTERN.md (line 0): overview is 16430 bytes, over the 16384 cap`; it is the only failing test in the Publish log.
**Fix:** Bring `PATTERN.md` to 16384 bytes or fewer, 46 or more bytes of saving, while keeping the in-flight-skip clause card 12 requires on the `PATTERN-hub-store-gc` line.
Shorten that line's wording, or tighten another entry's text without dropping its constraint; `wc -c PATTERN.md` and `lyx gate test ./internal/pattern` confirm it.
No other file needs to change, since the background file already carries the detail.

### [LOW] A board with no upstream keeps the build stamp absent forever

**Location:** `internal/hubreconcile/walk.go:209`
**Issue:** `reconcileHubWide` now calls `bolt.Push` on every walk, committed or not, and reports `skipped` on any push error.
`PullThenCommitWritten` deliberately lets a Bolt with no upstream write as before (`pullFirst` returns no skip on `ErrNoUpstream`), but `PushCoalesced` treats no upstream as unpushed, runs `git push`, and that fails with no destination.
For such a board `reconcileHubWide` returns `skipped=true` on every run, so `ensure` never writes the stamp and each start verb repeats the whole walk and the Warn.
Before this diff the push ran only after a landed commit and a failure was logged without touching the stamp.
Card 10 specifies the skip only for a push failure after a landed commit; the push-on-every-walk widening has no carve-out for a board that has nowhere to push.
**Fix:** Decide whether a board without an upstream is a supported state.
If it is, skip the push and the stamp-absent rule there, the way `pullFirst` skips the pull, and add a case to `TestEnsure_*`; if not, state in `doc.go` that a board must have an upstream for the stamp to be written.

### [LOW] No test reaches the no-upstream path of pullFirst

**Location:** `internal/fabricengine/bolt.go:219`
**Issue:** `pullFirst` has two branches for a Bolt with no upstream: a fetch error followed by `ErrNoUpstream`, and `UpstreamSHA` returning `ErrNoUpstream` after a successful fetch.
Card 7 and `doc.go` both state that such a Bolt writes as `CommitWritten` does.
`TestBolt_PullThenCommitWritten` runs every subtest over a `hubforge` hub with an origin, and `TestUpstreamSHA` covers the primitive only, so neither branch's outcome (no skip, write and commit land) is pinned through the public verb.
**Fix:** Add one subtest row to `TestBolt_PullThenCommitWritten` over a board whose branch tracks no upstream, asserting the write commits and `Skipped` is empty.
This is a row in the existing test, not a new top-level test.

### [LOW] Pruned legacy config paths are recorded as file_written

**Location:** `internal/fabriccli/fabric.go:759`
**Issue:** The write handed to `PullThenCommitWritten` appends `ConfigFileRel(legacy)` for each `result.MigratedFrom` entry so the commit stages the deletion.
`PullThenCommitWritten` records `KindFileWritten` for every path the write returns, so `lyx fabric reconcile`'s envelope `mutations` now lists the pruned legacy files as `file_written`; the removed handler recorded only each module's own config file.
`configsync` prunes those legacy files when applied, so the entries name files that no longer exist.
**Fix:** Have the write separate the paths it stages from the paths it wrote, or record `KindFileWritten` only for paths that exist after the write; the existing reconcile tests then assert the legacy path is absent from `mutations`.

### [LOW] Stale descriptions of the seeding commit path

**Location:** `internal/fabricengine/stencilcommit.go:1`, `internal/fabricengine/stencilcommit.go:24`, `internal/fabricengine/junctionnames.go:85`
**Issue:** The `stencilcommit.go` header says the file declares only `CommitSeededStencils` and that it deliberately does not build on Bolt, because seeding under Bolt would not exclude `boardCriticalSection`.
After this diff the root pre-run seeds through `Bolt.PullThenCommitWritten`, which holds the board write lock, and the file also declares `BinaryLabel`, `SeedCommitMessage`, `SyncCommitMessage` and `IsSeedCommit`.
The `StencilsSubtreeRel` and `SpecsSubtreeRel` comments still name `CommitSeededStencils` for a seeding pass, and `BoardWriteLockFile`'s comment lists its holders as the board engine and `CommitSeededStencils`, omitting `PullThenCommitWritten` and `Bolt.PushRecorded`'s drop.
**Fix:** Rewrite the header to state what the file now holds and that only `lyx stencil sync` still commits through `CommitSeededStencils`; reword the three comments to name the holder group, not an enumerated list.

### [NIT] underSubtree lacks a doc comment

**Location:** `internal/fabricengine/stencilcommit.go:163`
**Issue:** The unexported `underSubtree` is the only helper added in the diff without a doc comment.
**Fix:** Add a one-line doc comment stating it reports whether a board-relative path lies beneath a subtree prefix.
