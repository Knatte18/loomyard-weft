---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: lens:code-affected-set
    location: internal/fabricengine/junctionnames.go:237
    summary: The board entry asks to take the reserved `_launchers` name out of fabricengine, but the diff keeps it reserved with no recorded decision.
    fix: Leave the code in place and defer this finding to the run's parent, who records or refuses the correction with `lym loom decision add`. The fixer changes nothing here.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/hubmutation_integration_test.go:412
    summary: The deleted row `Remove_RefusesDriftedLauncherDirLinkWithRefusalObject` also pinned the `refusal` object of the `fabric remove` failure envelope, and no test asserts that shape now.
    fix: Add a row to `TestRunCLI_HubMutationScenario` that provokes a real containment refusal through `lym fabric remove --force` without launchers, for example a pair whose warp worktree path is a link out of the hub. Assert exit 1, ok false, a non-empty `error`, a `refusal` object with non-empty `check`, `what`, `target` and `reason`, `check` equal to `fabricengine.CheckContainment`, and a `mutations` key. If a surviving test already asserts that positive shape, name it in the commit message instead.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/retiredlaunchers_test.go:101
    summary: No row seeds a symlink at `_launchers` itself, so replacing `root.Lstat` with `Stat` in `SweepRetiredLaunchers` would fail no test.
    fix: "Add a row to `TestSweepRetiredLaunchers` with `linkRel` set to `_launchers`, pointing at a directory outside the hub that holds a script named `ide.sh`. The row needs no seeded files inside the hub. Expect nothing removed, `Kept` equal to the single string `kept _launchers: not a directory`, the link still present, and the outside script and sentinel untouched."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/doc.go:1564
    summary: Two comments still cite `refuseUncontainedPath`, which the diff deletes.
    fix: In `internal/fabricengine/doc.go:1564` replace `refuseUncontainedPath` with `containmentFailure`. In `internal/fabricengine/destroy_test.go:148` change the comment to name `containmentFailure` instead.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/retiredlaunchers.go:22
    summary: The `.cmd` names in `retiredLauncherFileNames` are untested because every row seeds only `.sh` files.
    fix: In `TestSweepRetiredLaunchers` add one `.cmd` script, for example `_launchers/y/beam/run.cmd`, to `fabricWrittenTree` and to `fabricWrittenRemoved` in sorted position, so the existing rows pin it.
  - id: F6
    severity: NIT
    class: design
    origin: handler
    location: internal/fabricengine/retiredlaunchers.go:34
    summary: The general `hubRel` helper, used by the hub workspace, teardown lock and portals code, now lives in `retiredlaunchers.go`, which names a retired container.
    fix: Move `hubRel` from `retiredlaunchers.go` to `hubworkspace.go`, its first caller's file, changing nothing else.
---

### [MEDIUM] Reserved `_launchers` name kept against the board entry

**Location:** `internal/fabricengine/junctionnames.go:237`, `internal/fabricengine/destroy.go:583`, `internal/fabricengine/doc.go:1672`
**Issue:** The amended entry says to take "the reserved _launchers name out of fabricengine". The diff keeps `launchersDirName`, returns it from `HubReservedNames`, keeps it a geometry root in the destructive gate and documents it as staying reserved in `doc.go`, `PATTERN-cwd-resolution.md` and `docs/overview.md`. The first commit message says so, but no `Added after Discussion` entry, `Pattern-Departure:` trailer or recorded parent correction backs it. Keeping the reservation is defensible while hubs may still hold the directory, so the code is not wrong, but the departure from the entry is unrecorded. This is for the parent to decide.

### [MEDIUM] Refusal-envelope shape no longer tested

**Location:** `internal/fabriccli/hubmutation_integration_test.go:412`
**Issue:** The deleted row drove a real gate refusal through `lym fabric remove --force` and asserted the `refusal` object, its `check` value, the flattened `error` and the `mutations` key. That behaviour outlives the launchers. A search of the surviving fabriccli and battencli tests for `"refusal"` finds only absence checks, in `envelope_test.go:143` and `cleanupremove_integration_test.go:408`, so the positive shape through the CLI has no test.

### [MEDIUM] No test for a symlink at `_launchers`

**Location:** `internal/fabricengine/retiredlaunchers_test.go:101`
**Issue:** The focus directive names a symlink at `_launchers` as a case to judge. The code is right, since `root.Lstat` does not follow the link and `IsDir` is false, so the entry is kept. Nothing pins it. `SymlinkIsKeptAndNotFollowed` seeds the link below the container, so a change to `Stat` would make the sweep descend through a link at the container and delete files outside the hub without failing any test.

### [LOW] Dangling comments name a deleted function

**Location:** `internal/fabricengine/doc.go:1564`
**Issue:** The doc comment and `destroy_test.go:148` cite `refuseUncontainedPath`, removed from `ancestors.go` by this diff. The surviving check is `containmentFailure`.

### [LOW] `.cmd` launcher names untested

**Location:** `internal/fabricengine/retiredlaunchers.go:22`
**Issue:** Four of the eight names in `retiredLauncherFileNames` are `.cmd` forms, and no row seeds one. Dropping them from the list fails nothing.

### [NIT] `hubRel` sits in an unrelated file

**Location:** `internal/fabricengine/retiredlaunchers.go:34`
**Issue:** The helper has callers in `hubworkspace.go`, `hubworkspace_write.go`, `teardownlock.go` and `portals.go`, yet is declared in the file for the retired launchers container. It was moved here by the diff.

## Rejected

- Lens code-tests finding 3, the untested failure of a removal in apply mode: faking it needs a directory race or a read-only parent, and the path is a plain error return the cleanup segment surfaces.
- Lens code-tests finding 5, deleted `ancestors_test.go` tests of `containmentFailure`: the gate that calls it is covered by `destroy_test.go` and `destroy_unix_test.go`, and the deleted tests were about the removed launcher teardown.
- Lens code-tests finding 6, no `assertNoHubContainers` call in `TestAnchorChange`: the same absence is asserted in the add tests, and a second assertion would duplicate that coverage.
- Lens code-tests note on a pair whose only remnant was a launcher directory, and on the missing `launchers_restored` test: removed behaviour needs no test.
