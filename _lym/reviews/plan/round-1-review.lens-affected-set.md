```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each card's change, I traced what reads or asserts it and checked which cards cover that set.

**Affected sets**

1. **Card 3: `runWindowCommand` and `timing` gain a context and `groupKillAfter`.**
   - Set: the only caller of `runWindowCommand` is `afterWindow` (`internal/desktophook/window.go:58`). `AfterWindowWithTiming` is used by `TestAfterWindow` (`internal/desktophook/window_test.go:65`), and its signature is kept.
   - Covered by card 3, in full.
2. **Card 12: `reedCLI.openTerminal`'s signature changes and `handPlacement` is added.**
   - Set: the `reedCLI` struct (`internal/reedcli/cli.go:55-56`), `windowCmd`, and the one stub in `TestWindowCmd_OpensATerminalAttachedToTheToldWorktree` (`internal/reedcli/window_test.go:57`).
   - Covered by card 12, in full.
3. **Card 13: `deps.terminal` and `startSession.openTerminal` change shape.**
   - Set: `productionDeps` (`cli.go:115`), `runStart` (`start.go:187-190`), `openWindows` (`start.go:369-370`), and three test sites:
     - `newStartDeps` (`start_test.go:169`);
     - `TestStart_WindowAndHubFailuresLeaveTheRestStarted` (`start_test.go:424`);
     - `TestStart_PrintsAttachCommandsWhereNoTerminalLauncherResolves` (`start_test.go:557`).
   - Card 13 covers all three test sites. `runStart` is not listed as a target, but it only assigns the opener, which compiles under the new shape.
4. **Card 2: three new desktop keys.**
   - Set: `internal/desktophook/template.yaml`, `Config`, `ConfigSpec`, `LoadConfig`, `TestLoadConfig`, `TestModules_EveryTemplateKeyClassified`, and the `internal/desktophook/` line of the package tree in `docs/overview.md` (line 346).
   - Card 2 covers everything but the overview line (finding F1).
5. **Card 1: `beam.window_placement` joins the per-task settings.**
   - Set:
     - `beamsetting.go`'s package comment;
     - in `dobbyengine`, `config.go`'s key lists (`:161`, `:177`), `checkTaskSetting`, the `tasksetting.go` comment, `doc.go` and `template.yaml`;
     - `boardcli/cli.go`'s `storeVerbs`;
     - `pattern/PATTERN-config-layers.md`.
   - The `run_watch` mentions in `docs/overview.md` (`:838`, `:853`, `:1725`) describe dobby's behaviour, not a list of settings, so they are not part of the set.
   - Covered by card 1, in full.
6. **Card 11: the hidden `place-window` verb.**
   - Set: `cli.go`'s file comment, the pre-run skip list and its comment, `doc.go`'s sentence on verbs run by launchers, the `AddCommand` list, and the stencil-seed skip annotation that `cmd/lym/stencilseedgate_test.go:78-92` pins for `switch`, `follow` and `button`.
   - Card 11 covers everything but the stencil-seed question (finding F3).
7. **Card 11: a `RolePlacement` child registered under the worktree's slug.**
   - Set: every reader of `runproc.List` or `runproc.Owners` under a run slug:
     - `runproc.End` and `EndSparing` (the intended reader);
     - `pairteardown/leftover.go:172-178` (`requireStillOrphan`: `len(live) &gt; 0` refuses);
     - `battencli/leftoverpairs.go:66-70` (`ProcessLive: len(live) &gt; 0`);
     - `battencli/cancel.go:782-787` (`otherLeft ... len(records) &gt; 0`);
     - `pairteardown/teardown.go:217` (`ownedProcesses`).
   - These read a role and are unaffected: `battencli/wire.go:777` (`RoleGate` and fixer roles), `fabriccli/anchordeps.go:163,184` (`RoleBatten`, `RoleDobby`), and `liveUnsparedChild` (dobby children only).
   - Card 11 and the Shared Decision run-registration weigh only `End` (finding F2).
8. **Card 10: `parentwait.go` joins the unbounded-wait list.**
   - Set: `unboundedWaitFiles` (`cmd/lym/daemonwakeups_test.go:26`) and `pattern/PATTERN-daemon-wakeups.md`'s list.
   - Covered by card 10, in full.
9. **Card 7: the contract test goes in `tier3`.**
   - Set: the tier table in `pattern/PATTERN-test-speed.md` (`:15`) and its "No gate runs tier3" text (`:63-66`), plus the hub's `gate.yaml` tier description (`../_bolt/_lym/config/gate.yaml:34`, "llm tests run by hand only").
   - Card 7 covers the pattern table; the hub description is outside the worktree (finding F4).
10. **Card 8: `lymdirs.WindowPlacesDirName`.**
    - Set: `internal/lymdirs/dirs.go` and the places file. `PATTERN-lym-files`' list of caches covers run-related caches, so this file is not part of that list.
    - Covered by card 8.
11. **Cards 12 and 13: the window command runs from the placement child.**
    - Set: the `window_command` and `window_wait_ms` comments in `internal/desktophook/template.yaml`, the package comment in `window.go`, `docs/overview.md:1763-1764`, and the windows paragraph in `machinecli/doc.go`.
    - `battenshed/template.yaml:2`'s `open_window_command` comment names no wait and no window command, so it stays true.
    - Covered by cards 12 and 13.

**Findings**

- **F1 — LOW, consistency.** Location: `docs/overview.md` package tree, line 346.
  - The tree gets no `internal/kwinplace/` entry. Its `internal/desktophook/` line still says the package holds only "the machine's window command, run after `lym reed window` and `lym ide` open a window".
  - Cards 12 and 13 edit `docs/overview.md`, but only its window paragraphs (around `:1755-1764`).
  - Evidence: `docs/overview.md:346`. The `grep` for `internal/kwin` in `docs/overview.md` found nothing.
  - Fix: card 12 or 13 adds the `kwinplace` tree line and widens the `desktophook` line to name the placement keys.
- **F2 — MEDIUM, design.** Location: card 11 (`11-place-window-child.md`), Intent on registration, and the Shared Decision run-registration.
  - The card registers the placement child under the task's or beam's slug but weighs only `runproc`'s `End`. Other readers count any live record under the slug as a live process of the run:
    - `requireStillOrphan` refuses an orphan teardown;
    - the dobby's orphan view sets `ProcessLive`;
    - cancel's `otherLeft` counts it as something left.
  - All three see the child for up to 10 s + 180 s + 120 s plus the run's script bounds.
  - The card names none of these readers and gives no verdict on whether each change is meant.
  - Evidence: `internal/pairteardown/leftover.go:172-178`, `internal/battencli/leftoverpairs.go:66-70`, `internal/battencli/cancel.go:782-787`.
  - Fix: card 11 lists these readers and states for each why counting a `RolePlacement` record is right, or makes the reader pass over that role.
- **F3 — LOW, consistency.** Location: card 11, `placeWindowCmd`.
  - The card adds `place-window` to the pre-run skip list. It does not say whether the verb carries `clihelp.SkipStencilSeedAnnotation`, which `switch`, `follow` and `button` carry and `stencilseedgate_test.go` pins.
  - Without the annotation, the root pre-run's stencil-seed pass runs before every child, in the hub directory, where `Spawn` starts it.
  - Evidence: `cmd/lym/stencilseedgate_test.go:78-92`; `internal/reedcli/doc.go` ("`follow` and `button` ... skip the root pre-run's stencil-seed pass").
  - Fix: card 11 states the verb's annotation choice, and adds a row to `TestSkipStencilSeed` if it carries the annotation.
- **F4 — NIT, consistency.** Location: card 7, the tier3 change.
  - The hub's `gate.yaml` describes `tier3` as "llm tests run by hand only". The contract test makes that description incomplete.
  - The file sits in the Bolt checkout, outside the task's worktree, so no card can edit it.
  - Evidence: `../_bolt/_lym/config/gate.yaml:34`.
  - Fix: file a board note for the operator.

**Tool calls**

1. Grepped the seam callers of `openTerminal`, `terminal`, `AfterWindowWithTiming` and `runWindowCommand`, the test names in `start_test.go`, and every non-Go mention of `window_command` and `window_wait_ms`. The manifest held the definitions but not the test call sites.
2. Read the batten template and recipe lines and the window lines in `docs/overview.md`, listed the files mentioning `run_watch`, and grepped `standalonestate`'s doc. This was to find doc readers of the changed keys, which the manifest lacked.
3. Read the `run_watch` lines in `docs/overview.md` and `battencli/cleanup.go`, the `desktophook` and reed-verb lines in the overview, and the enumerations of `runproc` roles. This was to judge whether they are lists the change must extend.
4. Read the package tree in `docs/overview.md` around line 346, the tests that read `overview.md`, and inventories of the reed verbs. This was to find the verb and package lists a new verb or package joins.
5. Read `cmd/lym/stencilseedgate_test.go` and looked for a test that the package tree is complete. This was to confirm the stencil-seed annotation set the new verb would join.
6. Grepped for where the `tier3` description lives (`gate.yaml`). This was to find the hub text that card 7's tier choice makes stale.
