```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find what reads or asserts it and check that some card covers it.

**Per change: the affected set and the cards that cover it**

- **Card 1, PATTERN.md lines and the two background files.** Readers are review only; the `(test)` marker on agent-addressing is kept. Covered by card 1.
- **Card 2, `lymdirs.SlackDirName`.** `dirs.go` header. Covered by card 2.
- **Card 2, `runproc.RoleSlackRelay`.**
  - `runproc/doc.go:4` role list: covered by card 2.
  - PATTERN-child-processes run-process-owner bullet: covered by card 9.
  - Every ender of `HubOwner` records now also ends the relay:
    - fabriccli `EndDobbies` (`anchordeps.go:251`): card 13 restarts the relay after it.
    - orchcli `EndRole` dobby (`cli.go:262-267`): limited to its role, so the relay is untouched.
    - `tenderengine/processes.go:117`: reads `HubOwner` records, ends nothing new.
    - tendercli `runproc.End(..., owner, ...)` (`deps.go:114`): partly covered; no card names it.
- **Card 2, `githubclient.CredentialDir`.** The test assertion is covered by card 2.
- **Card 3, new code `program-claim-in-strand-pane`.**
  - `daemon.fail` (`server.go:240-246`), `Client.exchange` (`client.go:247-251`), `TestResolveSender` row and creelengine `doc.go`: all covered by card 3.
  - The message text is unchanged, so no assertion changes.
- **Card 3, `ReadWait` and `HubStrands`.** Creelcli `doc.go`. Covered by card 3.
- **Card 4, new hubgeom teller.** `hubgeom/doc.go` teller list. Covered by card 4.
- **Card 5, new `slack` config module.**
  - `configreg.Modules`, `TestModules_SeedOnlyAndHubWideFlags` (`configreg_test.go:47,58`), `TestModules_EveryTemplateKeyClassified`: covered by card 5.
  - `configStrictnessDegradingSet` and PATTERN-config-layers degrading list (`PATTERN-config-layers.md:69`): covered by card 5.
- **Card 9, new relay loop file.** `unboundedWaitFiles` (`daemonwakeups_test.go:23`) and the PATTERN-daemon-wakeups "What counts" list. Covered by card 9.
- **Card 10, new mounted module.**
  - `registration_test.go` and `clitree_test.go`: no list there changes.
  - `excludedModules` (`sandbox_coverage_test.go:28`), both `seamsignature_test.go` lists and the `docs/overview.md` package tree: covered by card 10.
- **Card 11, new step in the prime's bring-up.**
  - Tests `TestStart_StartsTheDobbiesFirst` and `TestStart_ResumesTheRecordedSession`: covered by card 11.
  - `orchcli/doc.go:13` (bring-up steps and their warnings): missed.
- **Card 11, presence line in the resume pointer.** Four orchengine tests, covered by card 11. The line is absent by default, so existing pointer assertions are unaffected.
- **Card 12, prime dobby `Deps.Tick` gains the relay ensure.**
  - `TestHubDobbyDeps_PrimeTidyShuttle` (`dobbydeps_integration_test.go:71-80`): covered by card 12.
  - `beamcli/doc.go:209` and `:212`, which say the prime's `Deps.Tick` is the hibernate carrier's tick: missed.
  - `docs/overview.md:229` beamcli line ("the prime's carrying the hibernate request"): missed.
- **Card 13, relay restart and new `relay_start_error` field.**
  - `TestChangeAnchor` and `anchorFake`: covered by card 13.
  - `fabriccli/doc.go:110` (what a move restarts and what it reports): missed.
  - The `lym fabric anchor` Long help (`anchor.go` about 529-531, "reporting a start that failed as dobby_start_error…"): missed.
  - `anchor_integration_test.go:197` asserts only `dobby_start_error`, so it is unaffected.
- **Card 14, new orch-role section.** The marker set is pinned by `contracts/stencils/markers_test.go:36`; no new marker, so unaffected. Also `registry_test.go:20`, which keys the fixed stencil by name only.

**Findings**

1. **MEDIUM, consistency, cards 11, 12 and 13 (method).** Each card puts its package `doc.go` under `Uses` while it changes the behavior that doc describes, so the doc text goes stale. Members missed:
   - `internal/beamcli/doc.go:209` and `:212` describe the prime's `Deps.Tick` as the hibernate carrier's tick alone. Card 12 adds `EnsureRelayWhileAway` to that tick, so the text becomes false.
   - `internal/fabriccli/doc.go:110` and the `lym fabric anchor` Long (`internal/fabriccli/anchor.go:520-531`) list what a move restarts and reports, and name only `dobby_start_error`. Card 13 adds the relay restart and `relay_start_error`.
   - `internal/orchcli/doc.go:13` lists the bring-up steps and their warnings. Card 11 adds the relay step and its `lym slack start` warning.
   - `docs/overview.md:229` beamcli line. Card 12 adds the relay to the prime's dobby.
   - Fix: move each `doc.go` from `Uses` to `Extends` on its card, and name the Long text in card 13's Intent. Card 10 already extends `docs/overview.md`; card 12 should name the beamcli line, or card 10 should cover it.
2. **LOW, scope, cards 2 and 9.** Registering the relay under `runproc.HubOwner` puts it in the set every `HubOwner`-wide `runproc.End` ends.
   - `internal/tendercli/deps.go:114` calls `runproc.End(RunProcDir, owner, os.Getpid())`, which ends every record of the owner.
   - No card names this ender, or says that the relay stays down after it until a start, an away, a dobby tick or a bring-up.
   - I did not confirm that `owner` there can be `HubOwner`. Card 9's Intent should state the enders beyond the anchor change and their restart route, or confirm that the tendercli path never takes `HubOwner`.

**Tool calls**

1. Grep for tests enumerating runproc roles, tests of lymdirs names, the `tierpurity` `allowedSpawners` list and readers of `dobby_start_error`. The manifest held none of these readers.
2. Grep for module-name lists for the config change and tests reading orch-role. The manifest lacked the readers of a new module name and of the role text.
3. Grep for program-name constants. I needed to know whether `SLACK` joins an enumerated program list; none exists.
4. Grep for every caller of `runproc.End`, `EndSparing` and `EndRole`, and for the config module lines in `docs/overview.md`. The manifest held only the anchor's `EndDobbies`.
5. Read `anchor.go:520-540` (the Long text), `beamcli/doc.go` on Tick, and the `orchcli`/`orchengine` docs on the bring-up and the pointer. These docs were only in `Uses` lists and were never read.
6. Grep `registry_test.go` for orch-role size or hash pins, and grep for `HubOwner` users. The manifest lacked the stencil pins and the other `HubOwner` enders.
