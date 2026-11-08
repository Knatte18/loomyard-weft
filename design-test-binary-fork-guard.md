# A test binary pinned as a tmux job re-runs its own suite and fork-bombs the machine

`test-binary-fork-guard` · Task · bug, reed, gate, testkit · running · Worktree-Create

## Seen (orch, 2026-11-08)

Neve froze twice on 2026-11-08, at 14:37 and 15:40, and needed a hard restart each time.
The journal's memory samples show a fork bomb of `reedengine.test`, `tmux` and `bash` processes: from 125 to 537 test processes in four minutes, load above 800, memory and swap full.
The cause was a reed status job built from the running binary's own path (`composeSlackStateCommand`, `internal/reedengine/panebin.go`) in the Slack toggles change.
Under `go test` that path is the test binary, so each tmux session a test made ran the test binary again, which ran the suite and made more sessions.

## Fixed already (`3aaa98cc`)

- `slackLineExecutable` refuses the running test binary, so the Slack state job and toggle click are pinned only for a real `lym`.
- `reedengine`'s `TestMain` exits 2 on any positional argument (`testmain_guard_test.go`).
- The `claudeengine` hook commands name `lym` through `PATH`.

## Change (operator, 2026-11-08)

Guard the whole class across lym, not only reed's Slack lines:

1. Audit every place that pins a command built from `os.Executable` (or a stand-in such as `executablePath`) into tmux or another process that runs it unattended: status jobs, key and mouse bindings, hooks, watchdogs.
   Packages to read include `reedengine`, `reedcli`, `loomcli`, `battencli` and `adoptcli`; grep for the rest.
   Fix each one that could run a test binary.
2. One shared helper resolves "this lym binary" and never returns a test binary's path; every such pin uses it.
3. Every package whose tests reach such a pin gets the `TestMain` guard that exits at once on non-test arguments, with a test proving it, as `reedengine` has.
4. `lym gate test` caps the processes a test run may spawn, so a runaway loop fails the gate instead of freezing the machine.
