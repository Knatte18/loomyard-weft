# Card 10 — gate-cli

**Create:**
- `plan:internal/gatecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/gatecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/gatecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/gatecli#SlotBusyExit` -> `const SlotBusyExit = 75`
- `internal/gatecli/doc.go`
- `internal/gatecli/child_linux.go`
- `internal/gatecli/child_unix.go`
- `internal/gatecli/child_windows.go`
- `internal/gatecli/cli_test.go`
- `internal/gatecli/test_integration_test.go`
- `internal/gatecli/testmain_test.go`

**Edit:**
- `cmd/lyx#newRoot`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/hubgeom#GateSlots`
- `plan:internal/gateslot#TemplateConfig`
- `plan:internal/gateslot#WaitDir`
- `plan:internal/gateslot#WriteWait`
- `plan:internal/gateslot#Pool.Inherited`
- `plan:internal/gateslot#Pool.Holders`
- `internal/lyxcwd#ResolveWorktree`
- `internal/preflight#BoardLyxPresent`

**Intent:**
Add the cobra module `lyx gate` (`PATTERN-cli-cobra`) with one verb, `test`: `lyx gate test [-C <dir>] [--tags <tags>] <packages...> [-- <go test flags>]`, the one sanctioned route for an agent's slotted `go test` (D7).
It is not a standalone-capable CLI and does not go through `internal/cliwire` (`PATTERN-cliwire-sole-wiring` does not apply): it has no `--target-dir`, state, plan or stencils to wire.

- Its own pre-run resolves hub membership from the `-C` dir, made absolute against the seam cwd, else the seam cwd, never the bare process cwd when `-C` is given: `lyxcwd.ResolveWorktree` on that dir, which finds the containing worktree from any directory inside it, then `preflight.BoardLyxPresent`.
  It does not use `preflight.ResolveMode`, whose anchor gate would refuse a nested module directory.
- Inside a hub it takes the pool from `hubgeom.GateSlots`; with `gateslot.InheritEnv` set to a slot the pool reports `Inherited`, it runs inside that slot without acquiring, so a verify command calling it cannot deadlock.
  Bound: a forged `gateslot.InheritEnv` naming any held slot runs unslotted inside that holder's slot; card 12's deny refuses an agent command naming the variable, and an operator shell or a script hiding the name is outside the bound (D4's and D7's Bounds).
  Otherwise it writes a wait record under `gateslot.WaitDir(<anchor path>)`, acquires a slot under a context bounded by the limits' `CLIWait`, removes the wait record on acquire or exit, and on a timed-out wait exits with `SlotBusyExit` and a JSON error naming each holder's worktree and site and the way forward: re-run the same command.
- Outside every hub (no git worktree, or no hub board) it runs unslotted under `TemplateConfig`'s `-p` cap and logs that no hub bound applies; that path lets through only a target outside every hub, capped by the template's `-p` alone (D7's Bound).
- It needs at least one package argument, refused as a JSON error otherwise, and accepts a module-wide pattern, which is slotted.
- It runs `go test -C <dir> -p <cap>`, then `-tags <tags>` when given, then the packages, then the flags after `--`, with the lease's environment, streams the output, logs the spawn and its end (`PATTERN-spawn-observability`), and exits with `go test`'s code.
  The `go` binary name is an in-package seam so the signal tests can run a stand-in that sleeps.
- `go test` runs in its own process group (`child_unix.go`), so `lyx gate test` can kill the whole tree without signalling the shell that called it; it kills that group when it catches a terminating signal, including one sent to its own group, then releases its slot and exits.
  On Linux (`child_linux.go`) the child also carries a parent-death `SIGKILL` (`Pdeathsig`), started from a goroutine locked to its OS thread, so a `SIGKILL` of `lyx gate test` ends `go` itself; on another Unix the group outlives an uncatchable kill (D7's Bound).
  On Windows (`child_windows.go`) the child is assigned to a job object created with kill-on-job-close right after it starts, so any exit of `lyx gate test`, a hard kill included, ends the tree.
- It exposes `Command`, `RunCLI` and `RunCLIIn` like every module, and `doc.go` documents the verb, hub resolution, the inheritance rule, the exit codes and the bounds above.

`cmd/lyx/main.go`'s `newRoot` mounts `gatecli.Command()` and lists `gate` in the root's module list.
`cli_test.go` (tier 1) pins the missing-package refusal and the flag-to-argument assembly.
`test_integration_test.go` drives it over a `hubforge` hub (`PATTERN-hubforge-fixtures`): a free slot runs `go test -p <cap>` over a tiny committed package and exits with its code; every slot held past a shortened `cli_wait_sec` exits `SlotBusyExit` with the holders named; an inherited held slot runs without acquiring; `-C <worktree>` from a cwd outside the hub is slotted; `-C <nested module dir>` inside a hub worktree is slotted and runs in the nested module; a target outside every hub runs unslotted and logs it; a catchable signal mid-run kills the stand-in's group and frees the slot; on Linux a `SIGKILL` of the process mid-run ends its child.
`testmain_test.go` runs the package through `tmuxkit.Main` under `gitkit.HermeticGitEnv()` (`PATTERN-test-isolation`).
`SANDBOX-CORE-SUITE.md` gains a scenario carrying `**Covers:** gate` (`PATTERN-sandbox-coverage`) that runs `lyx gate test` over one sandbox package, then with no package argument, and checks the exit codes and the JSON refusal.
`contracts/specs/refusal-spec.md` gains a `## gate` section with a row per refusal (no package argument, slot busy, a `-C` directory that does not exist, an unreadable or invalid `gate.yaml`) and drops gate from nothing else; its way forwards name `lyx gate test` re-runs or the operator's `lyx config gate` from the prime, never a raw module-wide `go` run (`PATTERN-no-denied-recovery`).
`docs/overview.md` gains the `internal/gatecli/` tree line and a **gate** entry in its module list.
`test-budget.yaml` gains the `internal/gatecli` row.

**ImpactSummary:** One new module mounted on the root; existing modules are untouched apart from the root's module list, the sandbox suite, the refusal spec and the overview.
