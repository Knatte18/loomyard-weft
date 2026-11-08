```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge, as an outsider, the F1–F5 proposals the fixer carried out as proposed.

Findings:

1. **MEDIUM** — `pattern/PATTERN-child-processes.md` (the amended `os.Executable` bullet) and `internal/hubgeom/slackgeom.go:46` and `internal/hubgeom/dobbygeom.go:34`.
   - **Summary:** F1's proposal converted only the listed sites and amended the rule for two named exemptions. Two more lym spawns still call `os.Executable` and fit neither exemption.
   - **Evidence:**
     - `hubgeom` fills `Geometry.LymExecutable` from `os.Executable()`.
     - `slackengine/start.go:95` runs `exec.Command(geom.LymExecutable, ...)`, which starts the Slack relay as a lym child.
     - `dobbyengine/start.go:299` does the same for dobby.
     - The amended PATTERN text allows only reads of the binary, plus the `loomcli` drive wiring and the `battencli` wire. The rule is therefore still false for these two spawn paths.
   - **Fix:** convert both `hubgeom` lookups to `selfexec.LymBinary`, or name them in the exemption. Check how the `slackengine` and `dobbyengine` tests build their geometry before choosing.

2. **LOW** — `internal/ideengine/spawn.go`, the new exported var `LymBinary`.
   - **Summary:** the review's proposal to convert the vscode stamp makes every ideengine caller outside this package stamp the bare name `lym` under `go test`.
   - **Evidence:**
     - `selfexec.LymBinary` returns `ErrTestBinary` there, so `lymPath` is `""`.
     - Only the `ideengine` `TestMain` sets the `LymBinary` stand-in.
     - `internal/idecli/cli.go` calls `ideengine.Spawn`, `ideengine.Seed` or `ideengine.OpenTask`. Its tier-1 run was green in the gate, but its tier-0 test and any other caller of `ideengine` were not run after the change. No test of those callers was found to assert an absolute path, but I did not grep for one.

3. **LOW** — `pattern/PATTERN-child-processes.md`, the "resolve the path once at wiring" wording.
   - **Summary:** the stated reason covers only the `battencli` case.
   - **Evidence:** the review gave "resolve once" as the shared reason. The code comment at `battencli/wire.go:1038` says the path is resolved once because a deploy replaces the file under the running batten. `loomcli/drivewiring.go:171` has no such comment, so the amendment's "each resolves the path once" is accurate but gives no reason a reader could check.

No finding against F2, F3, F4 or F5.
- F2 only wraps the lookup, and the doc sentence is now true.
- The F3 seam defaults to `newProcessCap`.
- In F4, `lymtemp.Remove` is a no-op when no run directory exists.
- F5's `argv[2]` indexing is safe, since every `bind-key` argv has at least four elements.

Further tool calls:
1. `grep` for `Executable` in `slackengine`, `dobbyengine` and `tenderengine`, plus the callers of `ideengine.Spawn`, `ideengine.Seed` and `ideengine.OpenTask`. My context had `hubgeom`'s `os.Executable` calls but not whether geometry fields reach `exec.Command`, or who else calls `ideengine`.
