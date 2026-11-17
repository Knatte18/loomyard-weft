```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Behaviour → pinning test**
- Prime owned here starts the hub with `--all` and skips beams owned elsewhere, naming the machine → `TestStart_StartsOnlyWhatThisMachineOwns/PrimeOwnedHereStartsTheHubWithAll`
- Prime owned elsewhere: owned beams start by slug, the prime is skipped naming its machine and `lym adopt root` → `…/PrimeOwnedElsewhereStartsTheOwnedBeamsBySlug`
- Prime with no owner, and a beam with no owner, are skipped while owned beams start → `…/PrimeWithNoOwnerIsNeitherClaimedNorStarted`
- The verb claims no prime when it starts a beam under an unowned prime → **none**. The fake `beamStart` stands in for the child, so the prime claim the real `lym beam start <slug>` makes through `orchcli` `Prepare`/`claimPrimeFor` is never exercised.
- Nothing owned → nothing started or opened → `…/NothingOwnedStartsAndOpensNothing`
- A child that fails as a whole fails the verb (exit 1) → `…/AFailedBeamStartFailsTheVerb`
- A child that reports one entry with `error` marks that worktree failed → **none** (the fake never returns an entry carrying `Error`)
- `runBeamStart` decodes the child's envelope (`result.beams[].worktree/error`, `lastJSONLine`, the fallback reasons) → **none**
- Workspace opened and one terminal per started worktree → counted in `TestStart_StartsOnlyWhatThisMachineOwns` (`wantWindows` / `wantWorkspaces`). The workspace path, the terminal directory and the title are not asserted.
- No launcher resolves → each started worktree carries its attach command, plus a warning → `TestStart_PrintsAttachCommandsWhereNoTerminalLauncherResolves`
- A terminal that fails to open falls back to the attach command; a workspace that fails to open becomes a warning → **none**
- Hold printed with machine and subjects, with the re-run only for this machine's own hold → `TestStart_PrintsAStandingHold` (both rows)
- A named hub starts alone; an unknown shortname is refused → `TestStart_NamedHubStartsThatHubAlone`. The refusal text it asserts comes from the fixture's own `lookupHub`, not from `hubregistry.Lookup`.
- Refusals for an unreadable live record and an unresolved lym binary; a pull failure or skip as a warning; no desktop as a warning → **none**
- Launcher kept for an owned hub (a beam owned here under a prime owned elsewhere), removed for an unowned hub and a gone hub, all-hubs launcher goes with the last → `TestStart_KeepsTheDesktopLaunchers` (both rows)
- `removeGoneLaunchers` stops at the first error and returns what it removed so far → **none**
- `Desktop.Add`/`Remove`/`Shortnames` per form, an unrelated file left alone, a repeated remove → `TestAddRemove_KeepTheAllHubsLauncherWhileAnyHubHasOne`
- Launcher content per form: Exec escaping, `%` doubling, the WSL distro, the double-quote refusal, native `.cmd` → `TestAdd_WritesLaunchersRunningMachineStart`
- `locate` platform and form choice, the override, WSL by env and by interop, missing or unresolved directory, other platform → `TestLocate_ChoosesDirectoryAndFormPerPlatform`
- `parseUserDirs` → `TestParseUserDirs_ReadsTheDesktopDirectory`
- `freedesktopDesktopDir`'s `$HOME/Desktop` fallback and its non-NotExist read error → **none**
- Dobby start adds the launcher for a worktree owned here → `TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere/OwnedHere`
- Dobby start adds none for a worktree owned elsewhere or with no owner → nominally the `OwnedElsewhere` and `NoRecordedOwner` rows, but they cannot fail (finding F2)
- Dobby start adds the launcher for a beam owned here under a prime owned elsewhere (acceptance criterion) → **none** at the dobby level; the parts are covered by `TestOwnsAnything_ReadsEveryOwnerThroughTheRegistry/BeamOwnedHere` and the machinecli row
- A re-exec start adds the launcher → **none**
- Catch-up removes a standing launcher once nothing is owned, and keeps it while the prime is owned → `TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing`
- `removeLauncherIfOwnsNothing` leaves the launcher when the owners or the identity fail to read → **none**
- `battencli.HubCatchUp` sets `Launcher{BoardDir, Shortname}` → **none** (`TestHubCatchUp` asserts nothing about `Launcher`)
- `OwnsAnything` → `TestOwnsAnything_ReadsEveryOwnerThroughTheRegistry`
- termwindow resolving `wt.exe` on native Windows → `TestResolve_ChoosesLauncherPerPlatform/NativeWindows` and `…/NativeWindowsWithoutWindowsTerminal`
- Native `wt.exe` arguments and `;` escaping; Konsole and WSL arguments unchanged → `TestLauncher_ArgumentsOpenTitledWindowRunningAttach`
- tmuxkit sets `XDG_DESKTOP_DIR` → `TestSetEnv`

**Findings**

- **F1 — MEDIUM, class design** — `internal/machinecli/start_test.go:925-943` (fake `beamStart`). No test pins D10's "claims no prime" for beams started under a prime with no recorded owner. The production child `lym beam start <slug>` runs `orchcli.BeamUp` with `Prepare` on its first target, which calls `claimPrimeFor` and records this machine as the prime owner. Evidence: `internal/orchcli/start.go:495`, `internal/orchcli/ownerpause.go:104-120`, `internal/beamcli/cli.go:527-529`. The row `PrimeWithNoOwnerIsNeitherClaimedNorStarted` passes only because the fake never claims. The row's name asserts a behaviour the code does not have.

- **F2 — MEDIUM, class design** — `internal/dobbyengine/launcher_test.go:976-978`. The `OwnedElsewhere` and `NoRecordedOwner` rows of `TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere` cannot fail when `addLauncher`'s owner gate breaks. Each row's board has a prime owned by another machine or no owner, so the catch-up `Run` performs afterwards removes any launcher the start wrongly added. The test asserts only "stands after the start and its catch-up". Evidence: `launcher_test.go:988` and `:994`, `catchup.go:127` (`removeLauncherIfOwnsNothing`). Fix: assert after the start alone, or give those rows a board this machine owns, so the catch-up keeps whatever the start wrote.

- **F3 — MEDIUM, class design** — `internal/machinecli/start.go:747-777`. `runBeamStart` (the envelope decode, `lastJSONLine` and the reason fallbacks) has no coverage. The verb's coupling to `beamcli`'s `{"result":{"beams":[{"worktree","error"}]}}` shape is unpinned (`internal/beamcli/cli.go:436-450`, `:629-638`), so a renamed key would turn every start into "printed no result" with no test failing. The parse could be split from the spawn and tested at tier0 on a canned envelope, including the partial-failure envelope that carries both `error` and `result`.

- **F4 — LOW, class design** — `internal/machinecli/start.go:633-637`. A child that reports a per-worktree `error` entry, the partial failure `lym beam start --all` produces, has no row. The fake returns only whole-child errors (`start_test.go:929-931`).

- **F5 — LOW, class design** — `internal/battencli/catchup.go:46-48`. The `Launcher` wiring in `HubCatchUp` (board directory plus `ReadShortname(BoltDir)`) has no assertion. Dropping it would silently stop every production catch-up from removing launchers. Evidence: `internal/battencli/catchup_integration_test.go:34` (`TestHubCatchUp`) never mentions `Launcher`.

- **F6 — LOW, class design** — `internal/machinecli/start.go:673-681` and `:653-655`. No row covers a terminal window that fails to open falling back to the attach command, or a failed workspace open becoming a warning. `TestStart_PrintsAttachCommandsWhereNoTerminalLauncherResolves` covers only the case where no launcher resolves.

- **F7 — LOW, class design** — `internal/dobbyengine/launcher_test.go`. The acceptance criterion's case of a beam owned here under a prime another machine owns has no row at the dobby level: start adds the launcher and the catch-up keeps it. The `OwnedHere` row's board makes this machine the prime owner (`launcher_test.go:988`). The machinecli row `KeepsAnOwnedHubAndRemovesAnUnownedAndAGoneOne` covers only the verb.

- **F8 — LOW, class consistency** — `internal/dobbyengine/launcher_test.go:1001` (`TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing`). The redundancy report lists it as contained by `TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere`. Comparing assertions: only this test plants a launcher first and asserts that the catch-up removes it (`OwnsNothing`) or keeps it (`PrimeOwnedHere`). `TestRun_…` never starts with a launcher standing. It pins a side effect the covering test does not check, so keep it and mark it `//testtiming:keep the catch-up removing a launcher that stands` directly above its `func Test…` line.

- **F9 — NIT, class design** — `internal/machinecli/start.go:483-498`. The refusals for an unreadable live-hub record and an unresolved lym binary have no reaching test, and the shortname refusal is asserted only against the fixture's own text (`start_test.go:915`, `:1088`). PATTERN-refusal-way-forward binds only webster, shed and loom, so this is a coverage gap, not a rule breach.

Tool calls:
1. Grepped `battencli` tests for `HubCatchUp`/`Launcher`, plus `fakeCatchUpDeps` and `loopHarness.run`. I needed to know whether `HubCatchUp`'s `Launcher` wiring is asserted anywhere and whether the harness's `Run` performs the catch-up, which decides F2, F5 and F8; the manifest did not include them.
