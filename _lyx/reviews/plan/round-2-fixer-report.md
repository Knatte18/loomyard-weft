# Plan-Review round 2 — fixer report

I checked every finding against the code: `internal/reedengine/lifecycle.go`, `env.go`, `internal/verifytree/verify.go`, the reedcli smoke files and the tagged test files' build constraints.
Every premise holds, and all nine are fixed.
`lyx loom validate-plan` reports `ok: true` after the edits.

## Fixed

- **F1** (BLOCKING, design), `05-tmuxkit-hermetic-server.md`, `06-tmuxkit-package-server.md`.
  Card 5's `startServer` now applies reed's server-spawn environment hygiene: `reedengine.CleanClaudeEnv`, then it drops `LYX_TRACE_ID`, `agentname.StrandNameEnv` and `agentname.ParentEnv`.
  Its cwd is the kit's directory, and those symbols are listed under `Uses:`.
  `TestSocket_StartsHermeticNonLoginServer` gains the environment assertion through `show-environment -g`.
  Card 5 states the bound for what remains: there are no `-v` flags and no server log in the hub logs dir.
  Card 6 classifies `TestSmokeDebugLog` as registering right after its first `up` instead of before it.
  It is the only tagged test that asserts on reed's server-start artefacts; I checked this with a grep for `tmux-server-`, `HubLogsDir` and `StrippedEnv` across the tmux and llm files.
  Card 6 also gains `lyx gate test --tags tmux ./internal/reedcli -- -run 'TestSmokeDebugLog|TestSmokeColdStart'`.
- **F2** (MEDIUM, design), `10-tmuxkit-socket-check.md`.
  With `allowed` false, the per-user directory `tmux-<uid>` is a finding whenever it exists, and `TestCheckSockets` gains a row for it.
  The review's fix assumed only a server start creates that directory.
  I verified on tmux 3.6 that a client-only `tmux -L probe has-session` creates it too.
  The card therefore says the check fails a tier-1 or tier-2 binary that ran any tmux command, and that the integration Verify run and the plan-level integration verify surface such a test.
  For `allowed` true, the card and `PATTERN-test-isolation.md` state the bound that an unregistered server reed tears down itself escapes.
- **F3** (MEDIUM, design), `10-tmuxkit-socket-check.md` and `06-tmuxkit-package-server.md`.
  Card 10's `tmux` line now runs the tier, not just compiles it, for `loomcli`, `reedcli`, `reedengine`, `webstercli`, `pairteardown`, `standalonegeom` and `tmuxkit`.
  That line replaces the old compile-only line and the old partial run line.
  Card 6's net sentence now names that run for the tmux-tier packages.
  It also says the llm-tier packages are only compiled, so a missed registration there is left to the census and review.
- **F4** (MEDIUM, decision), `05-tmuxkit-hermetic-server.md`.
  I took the bound option and wrote it into card 5's Intent and the "Hermetic servers" section card 5 adds to `PATTERN-test-isolation.md`.
  The bound: a server reed starts itself carries no `-f`, so it reads the operator's `~/.tmux.conf` and starts login-shell panes.
  This covers the first boot of a test that registers after the boot, and every boot after a test's own `down` or `kill-server`.
  The bound applies to tests that pin reed's own start, teardown or recovery, which a pre-started server would bypass.
  Their key stays registered.
  I rejected the re-arm option because it means one `KillOnCleanup` per re-boot site.
  A census found such sites in most reedcli smoke files and in several reedengine, loomcli and llm-tier files, and nothing would catch a missed one.
  Note: this bound narrows D4's sentence "The operator's `~/.tmux.conf` is never read by a test server" to pre-started servers.
  D4's pre-start-with-`-f` mechanism cannot reach a server reed spawns itself without a production change to reed's spawn, which the record's amendment rules out ("Reed is unchanged" means its pane start mode, and only the stale-holder probe is licensed).
  The operator may want to confirm the narrowing.
- **F5** (MEDIUM, design), `13-gatecli-prebuild.md`, `00-overview.md` card index.
  The strip helper becomes `gateslot.StripPrebuilt`, declared beside `PrebuiltLyxEnv` because three packages call it.
  `internal/verifytree#Verify` joins card 13's `Edit:`: the environment it hands the verify command is `StripPrebuilt(os.Environ())`, through `lease.Env` when slotted.
  This covers the plan-level verify and Publish's `publish_verify`, which both go through `Verify`.
  `TestVerify_SlotGate` gains the assertion, and `internal/verifytree/doc.go` joins `Prosa:`.
  The forged-value bound now holds as stated.
- **F6** (MEDIUM, design), `31-tierpurity-tmux-constraint.md`.
  A file now counts as tmux-tier when `go/build/constraint` evaluates its `//go:build` line false both with no tier tag set and with only `integration` set.
  Host GOOS and GOARCH tags count as satisfied.
  The `integration || tmux` row now says not tmux-tier, and a `tmux && !windows` row is added.
  The landing sentence now names the constraint shapes found today, all of which are tmux-tier.
- **F7** (LOW, consistency), `13-gatecli-prebuild.md`.
  I corrected the sentence instead of rekeying the skip.
  Only a root without `cmd/lyx`, outside a hub, skips the build.
  A nested module inside a hub still gets the worktree's build, exported and unused.
- **F8** (LOW, consistency), `07-tmuxkit-private-tmpdir-leftovers.md`.
  I replaced "so no test package keeps its own" with: reedcli moves onto the exports in card 8, and the loomcli and orchcli copies stay, per the amendment.
- **F9** (LOW, decision), `04-weight-report-before.md`, `33-weight-report-after.md`.
  Card 4 states that the `llm` tier is left out of both reports, and the report header says so.
  The reasons: it runs live model sessions that spend tokens, its time is the model's rather than the suite's, and a spawned implementer may not run that tier.
  `running-tests.md` names `-resources -tags llm` for the operator.
  Card 33 repeats the exclusion.

## Disputed

None.

## Deferred

None.
