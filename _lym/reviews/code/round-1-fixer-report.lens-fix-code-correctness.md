```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code through the fix-code-correctness lens.

Fork list: `fix-code-correctness`, nothing extra asked for.

### MEDIUM — the Bolt checkout is scanned as a worktree
Changed text: `settingPlaces` in `internal/dobbyengine/reinterpret.go`.
- `settingPlaces` treats every sibling of the prime that holds a `.git` entry as a worktree.
  It skips only the prime's own directory name.
- The Bolt checkout `_bolt` holds a `.git`, so it is scanned as "the worktree _bolt" (`BoltDirName = "_bolt"`, `lymdirs/dirs.go:18`).
- The hub base file is `ConfigFile(boardDir, "beam")`, which is `_bolt/_lym/config/beam.yaml`.
  `ConfigFile(Dirs{Anchor: _bolt}, …)` builds the same path for the worktree layer (`layers.go:135-137`).
- When the hub file holds a retired value, three things happen:
  - `flagRetiredValues` flags the same file a second time, with the wrong advice "run `lym config beam --set …` in the worktree _bolt".
  - For `required` it also adds the `requiredLoss` text again.
  - Any `pull_request` set in the hub file triggers the "a worktree file's pull_request now decides …" line, although the hub file is the default layer.
- `TestReinterpretPerTaskSettings_HubWorktrees` passes only because it never creates a `_bolt` directory.
- The fix is to skip `lymdirs.BoltDirName` in `settingPlaces`, and to add a test row with a `_bolt` directory holding a hub `beam.yaml`.

### LOW — a read failure in the carry loop is silent
Changed text: `carryLegacyBoardSettings`, the line `if _, _, set, err := ownTaskSetting(dirs, key); err != nil || set { continue }`.
- A worktree whose own beam files do not read is skipped with no flag.
- The legacy value is lost, with no way forward in the mail.
- That departs from the review's F14 and F5 advice that read failures are flagged.

### LOW — `CheckLocalValues` conflicts with the marker expansion added for F23
Changed text: `CheckLocalValues` and `checkTaskSetting` in `internal/dobbyengine/config.go`.
- `CheckLocalValues` runs on local-layer writes and run_settings.
  It refuses a raw `${env:X}` value for `pull_request`.
- The lookup now expands that marker and accepts it.
- Result: a marker value is accepted in a worktree file and refused in run_settings or a local `--set`.
- A marker-carrying value is not an acceptable run_settings value.
- Even so, the two sites disagree on whether a marker is allowed.

### LOW — a never-read run past Land is read through a lookup that may have lost its worktree
Changed text: `readLandNoticeRequests` and `unreadLandNoticeRuns` in `internal/dobbyengine/landnotice.go`.
- A run past Land that was never read is now read through `Deps.LandNotice`.
- By then the worktree may be torn down.
- The chain then drops the worktree and returns the parent's `land_notice`, which may differ from the run's own.
- This only happens for runs whose earlier reads all failed.
- Without it, the notice would be lost.

### Checked and fine
- The F8 row `ARunNeverReadAheadOfLandWaitsPastLandForARead` fails on the old code, because the old code never read past Land and forgot the run.
- The F15 row fails without the `ClearOwed` branch, because the lookup would read `requested` and send the line twice.
- The F10 rows fail without the fold in `ParseBoardTasks`: the commit's `notify_on_land` is lost, and the removal falsely conflicts against the folded origin.
- With a nil `existing` and `RunsWithoutFile`, `rewriteHubFile` returns nil, so `bytes.Equal` holds and nothing is written.
- `legacyBeamSettings` on nil raw JSON returns an `Unmarshal` error and the key is skipped.
- Callers of `readLandNoticeRequests` and `ParseBoardTasks` still hold.
  The renamed `parentMode` and `parentErr` test fields stay consistent, and tiers 0 and 1 passed.

### Tool calls
- One `grep` of `configengine` for `ConfigFile`, `ConfigDir` and `LayerFile`, plus `BoltDirName` in `lymdirs`.
  My context lacked the path form of the hub base file and the Bolt directory name.
