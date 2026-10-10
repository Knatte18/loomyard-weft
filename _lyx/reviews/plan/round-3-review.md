---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-tmuxkit-socket-check.md
    summary: the per-user directory rule fails every tier-2 package whose tests reach a tmux client probe through production, such as fabriccli and battencli through pairteardown, and no card adapts them
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/13-gatecli-prebuild.md
    summary: webster's card-verify rerun runs each card's Verify with the inherited environment unstripped, so the card's bound that a verify command never sees the variable is false
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/05-tmuxkit-hermetic-server.md
    summary: the bound that every reed reboot after a test's own down or kill-server reads the operator's tmux config narrows D4's never-read rule, which the decision record does not license
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/31-tierpurity-tmux-constraint.md
    summary: evaluating platform tags against the host's GOOS lets an integration-and-windows file spawn tmuxkit while it compiles in Windows tier 2
---

### [BLOCKING] The per-user directory rule fails tier-2 packages that run a tmux client probe

**Location:** `_lyx/plan/10-tmuxkit-socket-check.md`, the paragraphs on the per-user directory `tmux-<uid>` under `allowed` false.

**Issue:** D6's runtime half fails a tier-1 or tier-2 package "when any socket file appeared under its directory".
Card 10 goes further: with `allowed` false the per-user directory's existence is itself a finding, and the card says itself that tmux creates that directory on any `-L` invocation, including a client probe of a server that never started.
I confirmed that against tmux 3.6: `TMUX_TMPDIR=<d> tmux -L probe has-session -t x` exits 1 and leaves `<d>/tmux-1000`, while `tmux -V` creates nothing.
Production reaches such a probe from tier 2.
`fabriccli`'s `fabric remove` (`internal/fabriccli/fabric.go:1080`) and `battencli` (`internal/battencli/wire.go:636`, `:654`) run `pairteardown`.
Its `EndSession` always calls `endReedSession`: `Engine.Down`, which runs `kill-session` and `list-sessions`, or `EndSessionByName`.
Before that, `waitQuiet` → `probeQuiet` calls `Engine.Status`, whose `requireSessionLocked` runs `has-session` (`internal/reedengine/lifecycle.go:1114`).
`fabriccli`'s `adoptremote`, `cleanhub`, `cleanupremove` and `hubmutation` integration tests and `battencli`'s `lifecycle_integration_test.go` drive those removals.
Both packages run through `tmuxkit.Main`, so under card 10 their `integration` tier fails although no server ever starts.
Card 10's `integration` Verify line leaves out `fabriccli` and `battencli`, so the card gate passes.
The plan-level `go test -tags integration ./...` then fails at the batch gate, and no card has those packages, or `pairteardown`'s production probe, in scope.
The card's claim that "a tier-2 test that trips it runs tmux, which D6 bars from tier 2" goes past D6, which bars a server, not a client probe that finds none.

**Fix:** Drop the directory rule and keep D6's socket-file rule under `allowed` false.
State the gap it leaves as a bound in the card's Intent and in `PATTERN-test-isolation.md`'s list of bounds: under any tag, a server that reed tore down itself removes its socket file and escapes the check.
That matches the bound the card already states for `allowed` true.
`TestCheckSockets` then drops its row for a per-user directory with no socket in it.
This changes no targets.

### [MEDIUM] Card verifies rerun by webster still inherit the prebuilt-binary variable

**Location:** `_lyx/plan/13-gatecli-prebuild.md`, the `verifytree.Verify` paragraph and the bound "A forged value never reaches a gated run"; also the Card Index line for card 13, "a verify command never sees the variable".

**Issue:** The card strips the variable on `lyx gate test`'s paths and in `verifytree.Verify`.
A third path runs verify commands as raw shell: `websterengine.rerunCardVerifies` (`internal/websterengine/cardverify.go:31`), called from `recordbatch.go:470`.
It reruns every card's `Verify:` through `verifyrun.Run`, with `lease.Env(os.Environ())` on the slotted path and a nil environment, so the inherited one, on the other (`runCardVerify`, `cardverify.go:47-69`).
A card's `Verify:` line is a free-form command.
One that runs `go test` directly, rather than through `lyx gate test`, sees whatever value the webster process inherited.
That falsifies the card's stated bound for the verify gates, and the attack-surface rule requires that bound to be true.

**Fix:** Add `internal/websterengine#runCardVerify` to card 13's `Edit:` list and have it pass `gateslot.StripPrebuilt` over its environment on both paths, as `Verify` does: through `lease.Env` when slotted, and as `StripPrebuilt(os.Environ())` when not.
The existing card-verify test gains a row, a command that prints the variable with the variable set through `t.Setenv`, expecting it empty.
`internal/websterengine/doc.go` goes under `Prosa:`.
The card's bound paragraph then names all three paths.

### [MEDIUM] Reed's reboots after a test's own teardown read the operator's config, against D4

**Location:** `_lyx/plan/05-tmuxkit-hermetic-server.md`, the bound "A server reed starts itself carries no `-f` … That covers the first boot of a test that registers after the boot, and every boot after a test's own `down` or `kill-server` on its key."

**Issue:** D4 decides that "the operator's `~/.tmux.conf` is never read by a test server".
Its one bound covers a test that never registers its key.
Card 5 adds a second class the record does not name: a registered test that runs reed's `down`, or `kill-server`, and then boots again.
That class gets a reed-started server with no `-f`, which reads the operator's config, and its strand panes start login shells.
`Down` kills the server whenever no session remains (`internal/reedengine/lifecycle.go:813-833`), so every reedcli smoke test that runs `down` and then `up` again falls in that class.
The plan states the gap honestly, but closing or accepting it is a decision.
D4's "Reed is unchanged", as the parent amendment reads it, leaves the plan no reed-side lever, and the plan neither closes the gap by a test-side lever nor cites a record entry that accepts it.

**Fix:** Settle it in one of two ways.
Either close the gap test-side: for example, tmuxkit writes a wrapper that runs `tmux -f <config> "$@"`, and the reed config that fixtures seed names that wrapper as reed's `tmux` binary, so every server reed starts on a test key reads the kit's config.
That puts the seeding helpers and the wrapper into card 5's scope.
Or raise it to the parent, so the decision record carries the bound beside D4's existing one, and card 5 cites that amendment by name.

### [LOW] The tmux-tier test treats platform tags by the host's GOOS

**Location:** `_lyx/plan/31-tierpurity-tmux-constraint.md`, "every other tag satisfied only when it names the host's `runtime.GOOS` or `runtime.GOARCH`".

**Issue:** On a Linux host, an `integration && windows` file evaluates false with no tier tag and false with only `integration` set, so the helper calls it tmux-tier and lets it reference a tmuxkit spawn.
On Windows that file compiles in the `integration` build, which is tier 2.
The existing scan's doc says the opposite: platform-only constraints "count as untagged: they still run in Tier 1 on that platform" (`cmd/lyx/tierpurity_test.go:75-76`).
No such file exists today, but the scan is meant to catch the next one.

**Fix:** Call a file tmux-tier only when both evaluations are false for every platform the module builds on: run the two evaluations once per GOOS in a small fixed set, such as `linux`, `windows` and `darwin`, with that GOOS's tag true and the others false.
`TestIsTierTagged_RecognizesKnownTagsList` then gains the row `integration && windows`, which is not tmux-tier.

## Focus verdicts

- Card 5's pre-start properties: the card says what the pre-start keeps of reed's spawn (`CleanClaudeEnv`, less `LYX_TRACE_ID`, `StrandNameEnv` and `ParentEnv`) and what it drops (`-v`, and the hub logs dir as cwd).
  `TestSmokeDebugLog` is the one test that asserts on `tmux-server-*.log` through a live server, and card 6 registers it after its first `up`.
  `internal/reedengine/lifecycle_test.go` touches those names only in the pure prune test.
  No test exercises the stale-holder reap that the `@lyx_test_server` probe skips.
  I checked that `tmux -f <cfg> -L k start-server` on tmux 3.6 keeps a session-less server answering `@lyx_test_server on`, and that a split pane's `pane_start_command` is the shell.
  `linuxTmuxSocketPids` still matches the pre-started server's argv.
  Not a defect.
- Verify lines that run rather than compile: card 5's line, card 6's first two lines, card 7's line, card 9's two lines and card 10's `tmux` and `integration` lines run tests.
  Every `llm` line, and the `tmux` lines with `-run '^$'` in cards 6, 12 and 22, only compile.
  Card 6's claimed net for the `tmux` tier, card 10's `tmux` line, is a real run over every package that has reed-booting `tmux` files.
  Card 6 states that the `llm` tier has no net but the census and review.
  Not a defect, apart from F1, which card 10's own `integration` net would surface at the batch gate rather than at the card.
- Tier-2 teardown: reed removes only its socket file, so card 10's directory rule does catch a torn-down tier-2 server; it also catches every client probe (F1).
  Under `tmux` or `llm` the card states the bound.
- Card 13: F2.
  The nested-module sentence agrees with `buildRoot`: inside a hub the root is the worktree root whatever `dir` is.
- Card 31: an `integration || tmux` file compiles under the `integration`-only evaluation, so the helper does not call it tmux-tier and the scan fails it.
  Not a defect, apart from F4.
- Reboots without `-f`: card 5 bounds them, and F3 covers it.
  Cards 7 and 8 agree: both keep `loomcli`'s `smoke_helpers_test.go` and `orchcli`'s `smoke_live_test.go` copies, per the amendment.
