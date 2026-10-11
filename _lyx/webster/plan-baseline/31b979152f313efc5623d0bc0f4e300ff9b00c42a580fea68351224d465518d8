# Card 3 — tmux-conf-boot-source-guard

**Create:**
- `internal/reedengine#tmuxConfClaimedOption`
- `internal/reedengine#reedSessionOption`
- `internal/reedengine#Engine.serverIdentityLocked`
- `internal/reedengine#bootGuardArgv`
- `internal/reedengine#Engine.guardBootSessionLocked`
- `internal/reedengine#TestTmuxConfBootGuard`
- `internal/reedengine#newHubEngine`
- `internal/reedengine#bootOwnServer`
- `internal/reedengine#TestBoot_TmuxConf`

**Edit:**
- `internal/reedengine#Engine`
- `internal/reedengine#New`
- `internal/reedengine#Engine.serverSpawnArgv`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#TestServerSpawnArgv`
- `internal/reedengine#TestBoot_ReadsNoOperatorConfigAndStartsNonLoginPanes`
- `internal/reedengine/doc.go`
- `pattern/PATTERN-test-isolation.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#tmuxConfEnv`
- `internal/reedengine#exactSessionWindowTarget`
- `internal/reedengine#Engine.pinBootOptionsLocked`
- `internal/reedengine#TmuxCmd.hasSession`
- `internal/testkit/tmuxkit#KillOnCleanup`

**Intent:** Source the operator file once, into the server the boot started, before reed's pins (D3); warn and continue on a sourcing error (D4); guard the session and server lifetime against the file on every POSIX boot (D5, with the parent's additions on read-back and interleaving); tag every reed session at creation (D8's tag, whose read-side rule is card 4).
Windows runs none of it (D7).

The new members live in `internal/reedengine/tmuxconf.go`.
`Engine` gains an unexported `warnOut io.Writer`, which `New` sets to `os.Stderr`; it is the booting command's stderr for D4's warning, and a test replaces it with a buffer.

`serverSpawnArgv`, off Windows, chains after the `new-session` arguments, with a literal `";"` argv element before each: `set-option -t <target> destroy-unattached off` and `set-option -t <target> @lyx_reed_session on`, where `<target>` is `exactSessionWindowTarget` of the session (D5 step 1).
The window form is load-bearing: `set-option` takes a pane target, and a bare `=<name>` fails with `no such session: =<name>` on tmux 3.6, while the `=<name>:` form every other option pin here uses resolves (probed).
The Windows argv is unchanged, and `-f /dev/null` stays where it is.

`serverIdentityLocked` returns the trimmed `display-message -p "#{pid} #{start_time}"` answer, and the empty string when the round trip fails or prints nothing; a session-less server answers it (probed), so the kit's pre-started server counts as seen.
`ensureServerAndSessionLocked` reads it once, off Windows, directly before the boot loop.
After `touchDiscoverSignal` and before `pinBootOptionsLocked` it calls `guardBootSessionLocked` with that pre-spawn identity and card 2's validated path, failing the boot on its error; on Windows neither the read nor the call runs and the boot is as today.
`guardBootSessionLocked` first reads the identity again, its session being up, and the boot is a candidate when the two reads differ (D3 step 1), which covers a server that ended between the read and the spawn and a zombie reaped inside the loop, and is never true for a boot that joined the server it saw.

`bootGuardArgv` builds one chained invocation.
For a candidate it opens with `if-shell -F "#{@lyx_tmux_conf_claimed}" "" "<claim>"`, the set branch empty, where `<claim>` is `set-option -s @lyx_tmux_conf_claimed sourced ; source-file <path>` when the path is non-empty and `set-option -s @lyx_tmux_conf_claimed empty` otherwise (D3 step 2); for a non-candidate it has no claim.
Both end with `; set-option -g destroy-unattached off ; set-option -s exit-unattached off` (D5 step 2), the pin set closed at these two.
The claim's check-and-set waits on nothing and precedes the source, which is what makes exactly one of two concurrent candidates win; the path sits inside a string tmux parses again, which card 2's character refusals keep single-valued.
`guardBootSessionLocked` runs that argv, built for its candidate decision, through `e.tmux.run`.
A non-zero exit is never fatal: when the invocation ran `source-file` it logs a Warn record carrying the path and tmux's stderr, which `run` folds into the error, writes the same warning to `warnOut`, and continues; without a path it logs the Warn alone (D4, parent addition: the one exit status carries claim, source and pins, so it decides nothing).
Then, in separate round trips, it reads back (D5 step 3): `hasSession` for its own session, failing the boot when the session is gone with an error naming `tmux_conf`, `tmuxConfEnv`, the path and the remedy of removing from the file the command that ends the session, such as an untargeted `set destroy-unattached on` or a `kill-session`; and one chained `output` of `show-options -gv destroy-unattached ; show-options -sv exit-unattached`, failing the boot when either line is not `off`, like a failed `remain-on-exit` pin.
The untargeted case is real: the guard client is detached, tmux takes the newest session as its current one, and the probe saw that session die the moment the option applied.
It logs the claim decision at Info with the candidate flag and the path; the tmux calls themselves already log at Debug.
Nothing re-sources the file into a running server, and a starter whose boot ends between spawn and guard leaves the server unclaimed and file-less until its next start (parent addition, no recovery path).

`TestServerSpawnArgv`'s POSIX rows expect the chained tail after the shell, and its Windows row stays as it is.
`TestTmuxConfBootGuard` drives the fake tmux in `tmuxconf_test.go` and names the process-global logger output it swaps through `logger.SetOutput`, so it does not run parallel; its subtests pin: `serverIdentityLocked` returns the answer trimmed and the empty string on an error; `guardBootSessionLocked`, with `display-message` scripted, issues the claim for a pre-spawn identity the post-spawn read differs from, for no pre-spawn identity, and for none when both reads are the same server; `bootGuardArgv` for candidate with path, candidate without path and non-candidate, each as the exact argv; a scripted `if-shell` error leaves `guardBootSessionLocked` returning nil while `warnOut` and the log carry the path and the stderr; a `has-session` exit 1 returns an error naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path; a read-back of `on` on either line returns an error.
`test-budget.yaml` raises `internal/reedengine` to 250 for the two new tests.

`newHubEngine` and `bootOwnServer` are factored out of `TestBoot_ReadsNoOperatorConfigAndStartsNonLoginPanes` in `serverconfig_integration_test.go`, which then uses them: the first builds the config and geometry of `worktree` under `hubDir`, applies `configure`, and returns the engine without registering a key; the second runs `Up`, registers the key through `KillOnCleanup` only after that, so reed starts the server itself, and returns `Up`'s error.
`TestBoot_TmuxConf`, in the new `tmuxconf_integration_test.go` constrained to `tmux && !windows`, writes each operator file into a `t.TempDir` and sets `TmuxConf` through `configure`; its subtests cover the acceptance criteria: a file setting a user option marker, `prefix` and `escape-time` leaves all three on the server; a file also setting `mouse`, `remain-on-exit`, `default-command`, `set -g exit-unattached on` and `set -g destroy-unattached on` boots, reed's values win on the server and the session, and a second worktree booted later into the server keeps `destroy-unattached off`; an untargeted `set destroy-unattached on` fails the boot naming the three; a file appending to a user option shows one append after a second worktree boots and after two worktrees boot concurrently into a fresh hub; two concurrent boots with the two global killers both succeed with both sessions `destroy-unattached off`; a parse error boots, applies nothing and writes the path to `warnOut`; a run-time error between two valid lines boots, applies both and warns; a run-time error beside the two global killers boots, warns and reads both pins `off`.
The pre-spawn-seen-server-ended criterion is the `guardBootSessionLocked` candidacy row whose two identities differ, since no test seam kills a server between the read and the spawn.
The criterion that Windows sources nothing is pinned by card 2's `TmuxConfUnvalidatedOnWindows` row and `TestServerSpawnArgv`'s unchanged Windows row; the skipped guard call itself is reachable only through a real psmux boot.

`doc.go`'s "Operator config" paragraph is rewritten around the key: the hermetic default, the one boot that sources and how it is identified, the claim, the order against the pins and what the file can and cannot change, the warning rule and the read-back, the session tag, and Windows.
`pattern/PATTERN-test-isolation.md`'s "Hermetic servers" section gains one bullet: a server reed starts sources the file `tmux_conf` names, which `tmuxkit.Main` empties by unsetting `LYX_REED_TMUX_CONF`, so a test that exercises it sets the key to a file it wrote and registers its key after reed's first boot.

Bound: the file changes only what reed does not pin; the two pins and the session check are what it cannot pass, and an operator who flips another option owns it.
Only the guard's exit status is non-fatal; card 2's refusals and the read-back still fail the boot.
Reed does not require `if-shell` or `source-file` from the multiplexer: a server lacking one fails the guard invocation, which is the warning, and both pins read `off` by default.

**ImpactSummary:** Every POSIX boot gains one identity read on each side of the spawn, two chained pins in the spawn and two guard round trips before the existing pins; an empty key leaves every option as it is today, and Windows boots are untouched.

**Verify:** lyx gate test --tags tmux ./internal/reedengine
