# Memory guard for every test binary lym runs

`gate-tier-memory-cap` · Note · bug, gate

## Seen (2026-11-17)

The diagnosis of `redundant-mutant-memory-cap` found that the gate tiers run `go test` with no memory cap either (`internal/gatecli/tierrun.go`).

What lym has today is no memory guard:

- The gate watchdog (`internal/gatecli/watchdog.go`) samples the process tree of a running `lym gate test` every 5 s, but only for CPU: it sends one SIGQUIT to a test binary that keeps a core busy for its window.
  It never looks at memory, and it watches only `lym gate test`'s own `go test`, not the mutant and testmap runs of `lym gate redundant` and `lym gate testmap`.
- `proc.ReadTaskLimit` (`internal/proc/tasklimit.go`) reads the cgroup's task limit so dobby can hold back starts; it kills nothing.
- On Neve, the user unit `memsample.timer` only logs a memory sample every 15 s to trace freezes; it kills nothing either.

A test that allocates without bound in a normal gate run can freeze the machine the same way a mutant did.

## Idea

- Give the watchdog a memory trigger: kill (SIGKILL, the whole test binary's tree) any test binary whose RSS passes a cap, or when the user slice nears its `memory.high`, and report which test and package was killed.
- Run the same watchdog over every test binary lym starts: the gate tiers, and the `Toolchain.RunAlone` runs of `lym gate redundant` and `lym gate testmap`, beside the per-run kernel cap `redundant-mutant-memory-cap` builds.
  The kernel cap is the hard bound; the watchdog catches what runs outside it, and names the culprit.
- Put the gate tiers' `go test` runs under the same per-run kernel cap, with a per-tier setting.
