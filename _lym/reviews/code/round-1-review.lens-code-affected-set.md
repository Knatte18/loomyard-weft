```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I looked for the callers, readers, tests, docs and rule documents it forces, and report whether each is covered.

**Set per change**
- **`open_window_command` default now `--title {title}`.**
  - Set found: `battenshed/template.yaml`, `config_test.go` (wantCommands), `battenshed/doc.go`, `shedadapters/doc.go` step 4, `shedadapters/command.go`, the `refusedPlaceholderWayForward` default branch, `contracts/recipes/batten-recipe.yaml`, `battencli/cli.go` (it names no placeholders), `docs/overview.md:702`, and `lifecycle_integration_test.go:1246`.
  - Covered: the template, test and docs are updated, and the percent-refusal branch handles `{title}` through its default case.
  - Two items need no change: `battencli/cli.go` and `batten-recipe.yaml` do not name the placeholders, and the lifecycle test uses its own `printf` command.
  - The reed status-bar `term` button (`template_posix.yaml:32,39`, `template_windows.yaml`, `reedengine/config_test.go:99,104`) still uses `{slug}`. This matches the entry and the summary.
- **New `CommandDeps.Title` field.**
  - Set found: the `battencli/wire.go` fill, the `envkit.FullEnv` fill, `wiring_seams_test.go` `intentionallyNil` (`Command.Title`), and the `command_test.go` fixture.
  - Covered: all four are updated. `TaskDir` is the only other filler in non-test code, and loom does not fill `Title`, which the seam table records.
- **`orchCLI.runsWindowTitle` changed from `string` to `func() string`.**
  - Set found: `cli.go:210`, `runs.go:80`, `start_test.go:168` and `runstitle_integration_test.go`. A repo-wide grep found no other reader of the field.
  - Covered.
- **`statusStrandAddSpec` parameter renamed from `recipe` to `title`.**
  - Set found: `sharedbootstrap.go:301` and `TestStatusStrandAddSpec`.
  - Covered.
- **New `boardengine.TitleWithHandle`.**
  - Set found: three callers (`loomcli`, `orchcli`, `battencli`), `boardengine/doc.go`, the `handles.go` header, and `handles_test.go`.
  - Covered.
- **Window-name docs.** These all agree with the diff:
  - `loomcli/doc.go:62`
  - `orchengine/doc.go:14`
  - `orchcli/runs.go:1`
  - `battencli/cli.go:23`
  - `docs/overview.md:424`, `543`, `686` and `821`
- **PATTERN.md and `pattern/`.**
  - Set found: no declared PATTERN change is in the directive. The diff touches no PATTERN or `pattern/` file.
  - No `Pattern-Departure:` trailer exists, and none is needed here.
  - Covered.
- **Commit message.** It says `<handle>: <slug>` for the task window and describes the status window as "opens its recipe with the run's handle", which is the recipe-title wording, not the entry's `overview`.
  - That is the departure the focus directive flags. It is recorded only in `summary.md`, which is the writer's claim and not a record. The handler should judge it holistically; I raise no separate finding.

**Findings**

1. **LOW / consistency** — `internal/loomcli/sharedbootstrap.go:294` and `internal/orchcli/runs.go:92`.
   - **Summary:** a live status or runs strand that is current (matching GUID and build in the sidecar) is kept as it is, so its window keeps the old title after the board assigns or moves its handle. The docs now say the window is "opened with the run's board handle" and "when the board names one".
   - **Fix:** either state in the docs that the title is fixed when the strand is added or replaced, or fold the title into the keep decision.
   - **Evidence:** `resolveStatusStrandAction` and `StrandCurrent` (`internal/statuspane/build.go:80`) compare only live, GUID and build. The `runsWindowTitle` comment in `cli.go:102` already says "when the strand is added or replaced".

No other member of the affected set is left wrong.

**Tool calls (3 of 6)**
1. Grepped `--title {...}`, placeholder lists and `beam:` and `overview` wording across the tree, because the manifest had no repo-wide view of other consumers.
2. Grepped `open_window_command` and `TaskDir:` across the tree, to find other readers and fillers outside the manifest.
3. Read `battencli/cli.go:215-225` and `shedadapters/command.go:125-195`, to check that `refusedPlaceholderWayForward` and `expandPlaceholders` handle `{title}`.
