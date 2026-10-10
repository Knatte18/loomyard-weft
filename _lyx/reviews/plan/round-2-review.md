---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-tmuxkit-hermetic-server.md:24
    summary: The pre-started server replaces reed's own server spawn, so reed's server-start contract (cwd at the hub logs dir, -v debug flags, CleanClaudeEnv hygiene) never applies, which breaks TestSmokeDebugLog and leaks the test process's Claude identity into llm-tier panes
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-tmuxkit-socket-check.md:23
    summary: checkSockets lists sockets only at the end of the run, and reed's Down deletes its own socket file, so a tier-2 test that boots reed and tears it down cleanly passes the check that D6 says fails on any socket that appeared
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-tmuxkit-package-server.md:67
    summary: Card 6 names card 10's tmux runs as the net for a missed registration, but those runs only compile reedcli, loomcli, reedengine and the llm packages (-run '^$'), so no test runs that could catch one before Publish
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/plan/05-tmuxkit-hermetic-server.md:23
    summary: A test that runs reed down or kill-server and then boots again gets a server reed starts without -f, so that server reads the operator's tmux.conf and starts login shells, against D4, and the plan states no bound or remedy for it
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/13-gatecli-prebuild.md:27
    summary: The bound that a forged prebuilt value never reaches a gated run is false for the plan-level verify, which verifytree.Verify runs as raw go test with the inherited environment, neither built nor stripped
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/31-tierpurity-tmux-constraint.md:11
    summary: The scan counts an integration || tmux file as tmux-tier, but such a file compiles in the integration build, so a tmuxkit spawn in it starts a server in tier 2, which is the hazard card 6 names for loomcli's smoke_helpers_test.go
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/13-gatecli-prebuild.md:25
    summary: buildRoot returns the worktree root inside a hub, which holds cmd/lyx, so a tagged gate run with -C on a nested module still builds lyx, contrary to the sentence that such a run builds nothing
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/07-tmuxkit-private-tmpdir-leftovers.md:31
    summary: Card 7 says the exported probes mean no test package keeps its own, but the D8 amendment keeps the loomcli and orchcli copies and card 8 follows it
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/plan/04-weight-report-before.md:10
    summary: The before and after reports measure the untagged, integration and tmux tiers but not llm, while D1 and the goal say every tier, and the plan gives no reason for leaving llm out
---

### [BLOCKING] A pre-started server bypasses reed's server-start contract

**Location:** `_lyx/plan/05-tmuxkit-hermetic-server.md:24`, with card 6's registration of every reed-booting tmux and llm file before its first boot (`_lyx/plan/06-tmuxkit-package-server.md`).

**Issue:** Reed's boot (`internal/reedengine/lifecycle.go`, the `spawnSession` closure) is what starts the server.
It runs `tmux` with `cmd.Dir = logsDir` so the server's `tmux-server-<pid>.log` lands in the hub logs dir.
It puts the global `debugArgs` (`-v`/`-vv`) on the spawn, which is both the client and the forked server.
It also sets `cmd.Env = clean`, after `CleanClaudeEnv`, `stripTraceID` and `stripAgentNameEnv`, so "a spawned server must never inherit this process's own Claude Code session identity".
Once card 5 pre-starts the server in `Socket`/`KillOnCleanup` and card 6 registers before every boot, reed's `new-session` is only a client command against tmuxkit's server.
That server's cwd is the test's cwd, it ran without `-v`, and its global environment is the test process's unscrubbed environment, which every session and pane inherits.
Concrete breakages:
- `internal/reedcli/smoke_debuglog_test.go`'s `TestSmokeDebugLog`, which card 6 lists for pre-boot registration, waits for a fresh `tmux-server-*.log` in `fabricengine.HubLogsDir`. No such log is written: the server was never started with `-v` or in that directory.
- The llm-tier panes (`orchcli`, `shuttlecli`, `seatengine`, `treadleengine`, `burlerengine`) inherit whatever Claude session variables the gate's agent shell carried. That is the nested-session identity `CleanClaudeEnv` exists to strip.

None of the plan's runs catch this before Publish: cards 5, 6, 10 and 22 compile reedcli and the llm packages with `-run '^$'`.
The plan never states that the pre-start drops these properties.

**Fix:** In card 5, give `startServer` the environment hygiene reed's boot applies: it drops the variables `CleanClaudeEnv`, `stripTraceID` and `stripAgentNameEnv` drop, spelled from reed's exported source where the kit's import rules allow, else stated as the kit's own list with a test row.
Card 5 should also state the remaining loss, that server cwd and `-v` come from tmuxkit, as a bound.
In card 6, classify tests that assert on server-start artefacts, `TestSmokeDebugLog` at least, as registering after the boot rather than before, and state their resulting bound.
Add a Verify line that runs `lyx gate test --tags tmux ./internal/reedcli -- -run 'TestSmokeDebugLog|TestSmokeColdStart'` instead of compiling it.

### [MEDIUM] The socket check misses a server whose socket file was removed

**Location:** `_lyx/plan/10-tmuxkit-socket-check.md:23`

**Issue:** `checkSockets` lists the socket files present after `m.Run`.
I verified on tmux 3.6 that tmux leaves its socket file behind after `kill-server` and after `exit-empty`, so a raw-tmux leftover is seen.
Reed's own teardown deletes the file, though: `ensureServerGoneLocked` calls `removeSocketFile` → `removeStaleSocket` (`internal/reedengine/lifecycle.go`, `socketfile.go`).
So a tier-2 test that boots reed and runs `down`, or whose cleanup does, leaves no socket and passes.
D6 says the package fails "when any socket file appeared under its directory", and the acceptance criterion says a package "whose test starts a tmux server fails".
The same gap applies under the tags, for an unregistered reed server that a test tears down.
On the focus question: the ordering is right and is pinned by a test. `afterRun` runs `checkSockets` before `sweep`, and the third `TestCheckSockets` row drives `afterRun` with a removing stand-in sweep. So the gap is not the sweep but reed's own removal during the run.

**Fix:** With `allowed` false, `checkSockets` also names the per-user directory `tmux-<uid>` under `dir` when it exists.
tmux creates that directory on a server's first start and nothing removes it.
`TestCheckSockets` gains a row with the directory present and no socket, named under `allowed` false.
For `allowed` true, state in card 10's Intent and in `PATTERN-test-isolation.md` the bound that an unregistered server reed tears down itself escapes.

### [MEDIUM] Card 6's stated net for a missed registration runs no test

**Location:** `_lyx/plan/06-tmuxkit-package-server.md:67`

**Issue:** Card 6 says "card 10's `tmux` Verify run and its compile lines are the net for a missed one".
Card 10's only `tmux` line that runs tests covers `pairteardown`, `webstercli`, `standalonegeom` and `tmuxkit`.
Its `reedcli`, `loomcli` and `reedengine` line and its llm line carry `-run '^$'`, and so do card 6's own reedcli/loomcli and llm lines.
A compile-only run starts no server, so `checkSockets` cannot fire, and F2 hides an unregistered key that reed tore down.
My census of the tmux- and llm-constrained files found the listed files plus those covered by `newIntegrationEngine`, `newColdScratchEngine`, `skipWithoutMultiplexer` and the revive helper, so I found no missed file today. The claimed safety net is still false.

**Fix:** Either run the tier: add `lyx gate test --tags tmux ./internal/reedcli ./internal/loomcli ./internal/reedengine` without `-run '^$'` to card 6 or 10.
Or rewrite the sentence to name Publish's tmux run as the net, with the llm packages left to review.

### [MEDIUM] A reboot after a teardown runs a config-less login-shell server

**Location:** `_lyx/plan/05-tmuxkit-hermetic-server.md:23`

**Issue:** Reed's `new-session` spawn passes no `-f`.
After a test runs `down` or `kill-server`, the next boot starts a fresh server that reads the operator's `~/.tmux.conf` and starts login-shell panes.
Examples: `smoke_staterecovery_test.go` runs `down` then `up` at lines 259–260, `smoke_resume_test.go` runs `kill-server` at line 142 then `up` at line 157, and `smoke_lifecycle_test.go` runs `down` at line 38 then `up`.
D4 says "The operator's `~/.tmux.conf` is never read by a test server".
D4's bound names only an unregistered start, and the plan neither handles nor bounds the re-boot.

**Fix:** Pick one and write it into card 5's Intent and `PATTERN-test-isolation.md`'s "Hermetic servers" section.
Either the listed tests re-arm the hermetic server before each re-boot by calling `tmuxkit.KillOnCleanup` again, which pre-starts, and card 6 names those files.
Or state the bound that a server reed restarts after a teardown is config-less and login-shell.

### [MEDIUM] The forged-value bound does not hold for the plan-level verify

**Location:** `_lyx/plan/13-gatecli-prebuild.md:27`

**Issue:** The card says "a forged value never reaches a gated run, so it can mislead only an ungated raw `go test`".
The plan-level `## verify:` gate runs the plan's raw `go test -tags integration ./...` through `verifytree.Verify`.
`verifytree.Verify` hands the command `lease.Env(os.Environ())` (`internal/verifytree/verify.go`), so an inherited `LYX_PREBUILT_LYX` reaches every package of that run unbuilt and unstripped.
That run is also the plan's largest tagged run, and it does not get D7's one build per gate run.
The card repeats D7's premise ("the verify gates ... run through it") without checking it.

**Fix:** Add `internal/verifytree#Verify` to card 13's `Edit:` so the command's environment drops `gateslot.PrebuiltLyxEnv`, which makes D7's "always sets or strips" hold at the verify gates too, with a row in `verifytree`'s existing env test.
Or restate the bound to name the plan-level verify and Publish's `publish_verify` as runs that inherit the variable.

### [MEDIUM] `integration || tmux` passes the tier-purity scan while compiling in tier 2

**Location:** `_lyx/plan/31-tierpurity-tmux-constraint.md:11`

**Issue:** The table row "`integration || tmux` is [tmux-tier]" lets a file that the `integration` build compiles reference a tmuxkit spawn.
Card 6 names exactly that shape as dangerous: loomcli's `smoke_helpers_test.go` is `//go:build integration || tmux`, and card 6 says it "never calls tmuxkit, since that would start a server in tier 2".
Under card 31's rule nothing enforces that sentence statically, and F2 means the runtime check can miss it too.

**Fix:** Decide tmux-tier by evaluating the constraint with `go/build/constraint`: a file is tmux-tier when its expression is false with only `integration` set.
Change the `integration || tmux` row to not tmux-tier, and keep the `tmux`, `llm` and `tmux || llm` rows.
Keep the check that every tmuxkit-spawning file today passes on landing.

### [LOW] The nested-module claim contradicts `buildRoot`

**Location:** `_lyx/plan/13-gatecli-prebuild.md:25`

**Issue:** `buildRoot` is "the worktree root inside a hub", which holds `cmd/lyx`, so `lyx gate test -C <nested module> --tags …` inside a hub builds `lyx`.
The same sentence says "a gate run inside a nested module … builds nothing".

**Fix:** Either key the skip on the `-C` module root, so a nested module builds nothing, and add that case to step three of `TestGateTest_Scenario`.
Or correct the sentence to say only a root without `cmd/lyx` skips the build.

### [LOW] Card 7 restates the struck "only copy" claim

**Location:** `_lyx/plan/07-tmuxkit-private-tmpdir-leftovers.md:31`

**Issue:** "so no test package keeps its own" is the claim that the amendment "D8 moves only reedcli's probes; other copies stay" struck.
Card 8 keeps the loomcli and orchcli copies accordingly.

**Fix:** Reword the sentence to say reedcli moves onto the exports in card 8 and the loomcli and orchcli copies stay, per the amendment.

### [LOW] The report leaves the llm tier unmeasured without saying so

**Location:** `_lyx/plan/04-weight-report-before.md:10`, and card 33, which repeats card 4's commands.

**Issue:** D1 and the goal say `cmd/testtiming` measures every tier, and D3 says one table per tier.
Cards 4 and 33 run the untagged, `integration` and `tmux` tiers only, and neither says why `llm` is left out.

**Fix:** Either add `-resources -tags llm` to both cards, or state in card 4 that the llm tier is left out because it spends tokens, and say so in the report's header.

## Focus verdicts

- Card 10 versus `Main`'s sweep: the order is right and is pinned by `TestCheckSockets`'s `afterRun` row; the remaining gap is reed's own socket removal (F2).
- Card 6's registrations: every tmux- and llm-constrained file that boots a hub-keyed or standalone-keyed reed server is either listed or goes through a helper that already registers. The claimed check that would catch a miss never runs (F3), and registering before the boot carries F1.
- `reed-exit-empty-probe`: the parent amendment "D4: reed's stale-holder probe skips the hermetic test server" licenses card 5's reedengine change. The bound holds for an operator's `~/.tmux.conf`: the early return keys on `@lyx_test_server`, not `exit-empty`, so an operator config that sets only `exit-empty off` is reaped as today. Not a defect.
- `PATTERN.md` overview cap: the plan carries the amendment. Card 23 names its four clauses, card 25 the new line wording and the `PATTERN-comment-line-breaks` clause, and card 32 the moved reasons. I checked that each moved clause is already in its background file. Not a defect.
- The prebuilt variable: gatecli and `cmd/testtiming` strip it on every path that exports no fresh build. The plan-level verify path is not covered (F5).
