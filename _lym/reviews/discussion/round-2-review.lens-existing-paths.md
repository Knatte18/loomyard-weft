```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths, tracing the code that already runs against each state the record introduces or changes.

**Paths traced, per decision**

- D1/D2 `heddle.json` in the Bolt checkout:
  - `boardengine.Sync` → `commitDirty` → `Bolt.Commit` commits every dirty file in Bolt, `heddle.json` included — survives.
  - `ensureLockfilesIgnored` patterns (`*.lock`, `*.swaplock`, manifest) don't match `heddle.json` — survives.
  - `Bolt.PullThenCommitWritten` / `pullFirst` dirty skip while a saved `heddle.json` waits for its sync: same window as `board.json` today, way forward `lym board sync` — survives.
  - `IsSeedCommit` drop doesn't touch a heddle commit — survives.
- D3 rendered files at the warp root:
  - `pullWarpArm` / `FastForwardPair` dirty check (`worktreeDirty(scopeTracked)`, tracked only) — survives.
  - `ResetHard` leaves ignored, untracked files — survives.
  - `destroy.go` `git clean -f -d` (no `-x`, weft side) — survives.
  - `impactset`, `commentlint`, webster `ls-files --others --exclude-standard` (honour `info/exclude`) — survives.
  - No `git stash` / `clean -x` anywhere in lym — survives.
  - `unseedGitExclude` strips only junction patterns, so `warp_exclude` lines stay, as D5 states — survives.
  - Claude Code's re-read of root instruction files after compaction — **breaks** D3's "a session reads only when it starts" (F1).
- D4 append-system-prompt file:
  - Launch and resume lines carry the run-dir file; reed relaunch reuses the stored line, so passages stay — survives.
  - Adopted `Prepare` with `ResumeSessionID` rewrites it — matches D4.
  - Claude Code re-renders from the flag file after compaction; the file is unchanged mid-session — survives.
- D5 exclude lines:
  - Unwire's unseed (junction names only) — survives.
  - `seedGitExclude` legacy bare-name drop (junction names only) — survives.
  - Health and drift over `junctionNames` (the list is kept out) — survives.
- D6 take-in event:
  - Pull warp arm, ff and reset: covered.
  - `ReconcileOrigin` code-side `ffWorktree`: covered.
  - `Fabric.FastForwardPair` (`ResetHard` to upstream) and `syncSideBeforeMerge` (`MergeFFOnly(upstreamSHA)` on the code side) also move a checked-out code branch to origin's commits and fire no re-render — **breaks**, against the list as written (F2).
- D7 replay registry:
  - `replayBolt` from `pullFirst` and from `reconcileRecordsBranch` (bolt) — covered.
  - `weftEndedByOrigin`, `readOriginEnded`, `addLocalCreatedStamps` and `readOriginOwners` read `board.json` — covered by the run ledger.
  - `SetAside`, `Lost` and `Refused` consumers are only `Bolt.logReplay` and `logWeftReplay` (log text). No reader parses a slug from them — survives.
  - The `SnapshotTagged` prefix query runs on the weft repo, not Bolt — survives.
  - An older binary on another machine replays only its own board-only local commits — survives.
- D9: `tenderengine` `HubOwner` compare, `pairteardown` parent, `boardengine` spawn register: all read the constant — survives.

**Findings**

**F1 — MEDIUM — design — D3 ("It writes the file in place, which a session reads only when it starts")**
Claude Code re-reads project-root instruction files from disk after compaction.
So a take-in re-render (D6) or a `lym heddle render` rewrites the instructions of a live session that later compacts.
An in-place write read during that reload can be seen half-written, and the per-worktree lock serializes writers only, not Claude's read.
The premise also carries the support log's rejection of an atomic rename, and D6's "a change is never pushed into a live worktree" bound.
Evidence:
- code.claude.com/docs/en/memory, "Instructions seem lost after /compact": "after /compact, Claude re-reads it from disk and re-injects it into the session" (fetched page, line 653).
- The same page: `CLAUDE.local.md` "loads alongside CLAUDE.md and is treated the same way" (line 123).
- `claudeengine` `ContextAfterCompaction` exists, so lym sessions do compact (`internal/shuttleengine/claudeengine/claudeengine.go` `buildSettings` call).

**F2 — LOW — design — D6 (take-in event list)**
D6's rule names the Pull warp arm and `ReconcileOrigin`'s fast-forward. Two more existing paths move a checked-out code branch to origin's commits and fire nothing:
- `Fabric.FastForwardPair` (`ResetHard(rec, upstreamSHA)`)
- `syncSideBeforeMerge` (`repo.MergeFFOnly(upstreamSHA)` on the code side before a merge)

A new `AGENTS.md` arriving through either leaves `AGENTS.override.md` holding the old copy, with nothing pointing at `lym heddle render`.
This may fall under the rubric's call-site-enumeration exclusion. It is reported because the list after the general rule reads as closed, and the acceptance criteria test only Pull.
Evidence: `internal/fabricengine/parenttips.go:237,280`; `internal/fabricengine/merge.go:581,615`.

Tool calls:
1. grep for instruction-file names, `--ignored`/`--others`/`ls-files` and unseed functions — to find existing readers of untracked or ignored warp files, which the manifest lacked.
2. `unseedGitExclude`, `destroy.go` clean, `warpWorktreeDirty` — to see whether unwire, clean or dirtiness checks undo or misread the D3/D5 state.
3. grep for `stash`/`--all`, plus webster `ignoredPath` and `ensureLockfilesIgnored` — for other git paths that sweep ignored files, and the Bolt ignore patterns against `heddle.json`.
4. `logWeftReplay` and consumers of `SetAside`/`ReplayOutcome` — to see whether a non-ledger set-aside slug is misread downstream (D7).
5. `reconcileRecordsBranch` replay reporting — the same question for the reconcile report.
