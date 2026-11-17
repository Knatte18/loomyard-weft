```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes against a concrete input.

**Per function or branch: the case I ran, and whether it is correct**

- `stencilstore.Pins`
  - With an empty registry, it returns an empty map. Correct.
  - With one stencil whose board copy is missing, that stencil is left out. Correct.
  - With an unreadable board copy (EISDIR), it returns an error naming the stencil, and nothing is pinned. Correct as documented.
- `stencilstore.ChangedPins`
  - A nil or empty `pins` returns nil. Correct.
  - A board copy that was deleted counts as changed. Correct.
  - A banner-only restamp: `BodyHash` strips the banner, so it is not reported. Correct.
  - A stencil installed after the pins were taken is not in `pins`, so it is never reported. Correct.
  - The result is sorted by name. Correct.
- `stencilstore.readBoardCopy`
  - ENOENT gives found false with a nil error; any other error is wrapped. Correct.
- `shedrun.StencilPinsFile` and `StencilPinsRel`
  - Default run: `RunDir` and `StencilPinsRel` both go through `runSegment(l, SelfRunID)`, so they name the same file. Correct.
  - Legacy `self` directory: the fallback picks the same segment for both. Correct.
- `shedrun.WriteStencilPins` and `ReadStencilPins`
  - With no file, the read gives found false. Correct.
  - Both take the lock from `lock.TrackedFileLockPath` under `.lym/locks`. Correct.
- Write and read run ids
  - Write uses `SelfRunID`; read uses `c.runID`, which `resolveRunID` sets to `SelfRunID` for start and for resume (`cobra.NoArgs`).
  - When start is given a positional equal to the slug, `ResolveRunID` turns it into the slug, and `runSegment` picks the same directory. Correct.
- `loomCLI.writeStencilPins`
  - With an empty `StencilsDir`, the paths resolve relative to cwd. The board copies are absent, so it writes `{}` and returns true. That is harmless, and production start and resume are fully wired (`start.go:103` reads the same `StencilsDir`).
  - When the hash or write fails, it logs at Warn and returns false, and the commit leaves the path out. Correct.
- `seedAndCommitBootstrap` step 2d
  - A re-entry with `ErrSeedExists` skips the write, so the pins are kept. Correct.
  - A failed pin write never puts a path into `commitPaths` (it is gated on `wroteStencilPins`). Correct.
  - **Incorrect:** if the pins are written and the seed commit then fails or the process crashes, they are never committed. See F1.
- `loomCLI.warnChangedStencils`
  - With no record, it returns nil and sends no notice. Correct.
  - A malformed record, or a failure from `ChangedPins`, is logged and returns nil. Correct.
  - When something changed, it posts once and returns the names. If the post fails, it logs and still returns the names. Correct.
  - Repeated starts or resumes after the same change post the same notice every time. See F2.
- `stencilsChangedNotice`
  - With one name, it renders "the stencils X changed …". Correct, apart from wording.
- `withChangedStencils`
  - With an empty list, `fields` comes back unchanged. With names, it adds the `stencils_changed` key. Correct.
- `start` RunE step 7
  - The check runs after the bootstrap lock is released. On a first start the pins were just written, so it reports nothing. Correct.
  - The live-strand no-op path also reaches step 7, so it re-warns. See F2.
- `resume`: Go-spawn branch and parked-wake branch
  - `warnChangedStencils` is evaluated as an argument before `succeed` releases the lock. The creel post therefore runs while the bootstrap lock is still held. See F3.
  - The done-run and already-running no-op paths do not warn. That is consistent with the help text.
- `newLoomCLI` `notifyParent`
  - It is wired in production. Receivers built in tests leave it nil, but it is called only when something changed and the test sets it on those paths. Correct.

**Findings**

- **F1 — MEDIUM, design. `internal/loomcli/sharedbootstrap.go:151` and `:185`.** A pins file written by the first seed is never committed if that seed's commit fails or crashes.
  - The pins are committed only when this invocation wrote them (`if wroteStencilPins`). A re-entry gets `ErrSeedExists`, so `wroteStencilPins` is false.
  - `bootstrapCommitPaths` (`sharedbootstrap.go:300`) re-commits the status, seed and origin on every call for self-healing; it does not include the pins.
  - `statusCommitPathspec` (`wiring.go:113-130`) carries the usage directory but not the pins file.
  - So `_lym/shed/<slug>/stencil-pins.json` stays untracked after any failed or crashed first-seed commit.
  - Step 3's own comment (`sharedbootstrap.go` "An uncommitted path fails that check") says the precondition scan fails on an untracked path, so the run is blocked.
  - The usage record survives the same window only because status commits carry its directory.
  - Fix: add `StencilPinsRel` to `commitPaths` whenever the file exists, not only when this invocation wrote it.
- **F2 — LOW, design. `internal/loomcli/stencilpins.go:57`, `internal/loomcli/start.go:628`.** Every successful start or resume posts the same parent notice again, even when nothing new changed.
  - This includes the live-driver no-op start.
  - `orchcli.NotifyParent` tags the post only with `Run`, not a topic (`notify.go:47-50`), so unread duplicates pile up rather than replacing each other.
  - Batten resumes a child run with `lym loom start` repeatedly, so the parent can get one copy per hand-back.
- **F3 — NIT, design. `internal/loomcli/resume.go:135` and `:164`.** The creel post in `warnChangedStencils` runs while the bootstrap lock is still held.
  - Go evaluates `withChangedStencils(..., c.warnChangedStencils())` before `succeed` releases the lock.
  - A slow daemon post therefore lengthens a hold that only the driver spawn or wake needs. Start, by contrast, releases the lock first (`start.go:625-628`).

Outside this lens, for the handler: `lym stencil diff <name>` (`stencilcli/diff.go` `runDiffOne`) compares the fork base with the shipped default. It does not show what changed since the run's pins were taken. The notice's "see each change with lym stencil diff <name>" may therefore show nothing, or a different change.

**Tool calls**

1. Searched for `RunDirRel` and `UsageDirRel` and the lightweight verb list, to find which later commits pick up run-directory files. The context did not have the status-commit pathspec.
2. Read `wiring.go:110-130` and `driftwiring.go:280-293`, to confirm whether status commits carry the pins file. The context did not have `statusCommitPathspec`.
3. Read `bootstrapCommitPaths`, to confirm the per-call self-healing commit set. The context held only its call site.
