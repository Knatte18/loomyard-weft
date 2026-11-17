```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

**Per change, the affected set I found, covered or not**

- **`termwindow.Launcher.arguments` now takes `lymPath` instead of a shell line, plus the native Windows `wt.exe` launcher.**
  - Covered: `OpenAttach` callers (`internal/reedcli/window.go:34` keeps its signature); `termwindow_test.go` rows updated; `termwindow_integration_test.go:89` calls `attachLineWaiting`, which is unchanged; the package doc is updated.
  - Not listed in `docs/overview.md`'s tree before or after the diff, so the diff makes nothing stale there.
- **tmuxkit's `setTempEnv` now sets `XDG_DESKTOP_DIR`.**
  - Covered: `tmuxkit_test.go` `TestSetEnv`.
  - Not covered: `pattern/PATTERN-test-isolation.md:67-69` still lists only the variables `Main` pointed at before. Finding F2.
- **New package `internal/desktoplauncher` (imports `internal/shell`, `internal/logger`, `golang.org/x/sys/windows`).**
  - Covered: `golang.org/x/sys` is in `go.mod:16`; `docs/overview.md` gains the tree entry; `testmain_test.go` calls `tmuxkit.Main` (PATTERN-test-isolation); the spawns in `locate_other.go` are logged and waited on (PATTERN-child-processes).
- **dobbyengine imports `desktoplauncher`, `machinename` and `selfexec`.**
  - Covered: `seam_enforcement_test.go` allowlist updated; `doc.go` gains a launcher section; `loop.go` `Run` doc updated.
- **`dobbyengine.CatchUpDeps.Launcher` and `battencli.HubCatchUp` filling it.**
  - Callers covered: `beamcli/dobbydeps.go:261` (dobby), `battencli/wire.go:1014` (hand pass), `adoptcli/deps.go:93` (adopt catch-up).
  - All three now remove the launcher when this machine owns nothing in the hub. That matches D10, which says every catch-up. The doc in `dobbyengine/doc.go` names the start, sleep and hand-pass catch-ups but not the adopt's; that sentence is not wrong, and I found nothing stale.
  - Hand-built `fakeCatchUpDeps` has a zero `Launcher`, so `TestCatchUp` is unaffected.
- **machinecli `deps` gains the start seams; `liveHubs` is split into `liveHubEntries` and `recordDir`.**
  - Covered: name-verb callers use `d.hubs` unchanged; `cli_test.go` and `set_test.go` build their deps by hand; `doc.go`, the cobra `Short`/`Long` and `docs/overview.md`'s machine module entry are updated.
  - Not covered: `cmd/lym/sandbox_coverage_test.go:62-65` excludes the whole `machine` module with a reason that names only `name <name>`. Finding F3.
- **`lym machine start`'s reliance on `lym beam start <slug>` (the callee's behaviour).**
  - Not covered: `orchcli`'s `Prepare` claims an unowned prime in a beam too. Finding F1.
- **Commit messages.** No `Pattern-Departure:` trailer is in the range. Commit `5bb2fa43` says "the verb claims nothing", which F1 contradicts.
- **Declared PATTERN changes.** The directive declares none, so this raises nothing.

**Findings**

F1 — **BLOCKING**, class `design`, `internal/machinecli/start.go:621-624`.
Under a prime with no recorded owner, the verb starts each owned beam with `lym beam start <slug>`. That child prepares its first target, and `Prepare` claims the prime for this machine whenever the board records no prime owner. So the verb does claim the prime, against D10's bound ("never claims a prime") and the acceptance criterion "claims no prime".
Evidence:
- `internal/beamcli/start.go:22` (`prepare := index == 0`)
- `internal/beamcli/cli.go:528-529` (`BeamUp` with `Prepare: prepare`)
- `internal/orchcli/start.go:495` (`claimPrimeNotes` in the prepare path)
- `internal/orchcli/ownerpause.go:104-120` (`claimPrimeFor` writes the owner when `owners.Prime == ""`)
- `internal/orchcli/doc.go:233-234` ("The claim runs in the prime and in a beam alike")

The test fakes the `beamStart` seam (`start_test.go:925-943`), so the row `PrimeWithNoOwnerIsNeitherClaimedNorStarted` cannot see the claim.

F2 — **LOW**, class `consistency`, `pattern/PATTERN-test-isolation.md:67-69`.
`tmuxkit.Main` now points `XDG_DESKTOP_DIR` at the run directory so no test writes on the operator's desktop. The background that lists which variables `Main` points there was not updated. The list already omitted `XDG_CONFIG_HOME`, so the gap predates the diff, but the diff adds a new isolation guarantee this rule document does not state.
Evidence: `internal/testkit/tmuxkit/tmuxkit.go:1369-1376` (diff), `pattern/PATTERN-test-isolation.md:67`.

F3 — **LOW**, class `consistency`, `cmd/lym/sandbox_coverage_test.go:63-64`.
The exclusion reason for the `machine` module names only `name <name>`. The module now also has `start`, which starts `lym beam start` children, opens windows and writes desktop launchers, and the reason does not cover it. The exclusion still holds, but its written reason is stale for the module.
Evidence: `cmd/lym/sandbox_coverage_test.go:62-65`, `internal/machinecli/cli.go` (diff adds `machine.AddCommand(start)`).

**Tool calls**

1. Searched docs, contracts, plugins and pattern for mentions of the machine verb, termwindow and the XDG variables, plus go.mod for `x/sys`, the shell package's API, and test files naming the touched packages. My context had none of these repo-wide references.
2. Filtered that saved output to drop research docs, because the first output was too large to show.
3. Grepped `termwindow_integration_test.go`, `sandbox_coverage_test.go`, `hermeticenv_test.go` and machinecli's `cli_test.go` for uses of the changed symbols. Not yet read.
4. Read the head and exclusion list of `sandbox_coverage_test.go` to see whether it enumerates modules. Not in context.
5. Grepped `docs/overview.md` for termwindow and dobbyengine entries. Call 1 had excluded the overview's tree lines.
