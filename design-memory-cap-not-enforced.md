# The per-process memory cap sometimes does not hold, and a runaway process takes the machine down

`memory-cap-not-enforced` · Task · bug, process, gate · running · PR-Review

## What was seen (2026-11-18)

`internal/proc`'s `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap` passed in a tier-1 gate at 13:03 UTC
and failed at 13:10 and 13:12 UTC (gate logs `20261118T131024Z-tier1-808563` and `20261118T131227Z-tier1-1218541` under `_bolt/.lym/gate/runs/`):

`memorycap_linux_test.go:128: RunMemoryCappedVia of a shell whose child keeps 384 MB under 134217728 = tree killed false, <nil>, 1 starts, output "survived\n"`

So the cap did not hold: the child kept 384 MB in a 128 MB scope.
Right after, a fixer probing this by hand ran `systemd-run --user --scope -p MemoryMax=134217728 -p MemorySwapMax=0 -p OOMPolicy=continue -- sh -c '... timeout 20 tail /dev/zero ...' | tail -8`;
a `tail` process grew to 11.4 GB outside any cap, the load reached 265 and Neve hung until a power-off at 14:24 local (journal of boot -1, `memsample` lines).
Earlier the same day the cap did hold: `lym gate redundant`'s `mutants.test` processes were killed at their 2 GB cgroup cap at 14:10 local.
The cap came with `bd707e16`, `c75a2b5d` and `580b27a1` (2026-11-17); the failure is intermittent, which is how it landed.

## Fix

- Find why the cap sometimes does not apply: for example a race between the scope's creation and the child's start,
  a process that ends up outside the scope's cgroup, swap or `MemorySwapMax` handling, or a fallback path that runs the command uncapped without saying so.
- Fix it so a capped command can never run uncapped silently: if the cap cannot be set, the call fails or says so loudly.
- Never reproduce the fault with an unbounded memory hog on the shared machine: every probe and test hog is bounded in size (a fixed number of MB) and in time.

## Done when

- The memory cap tests in `internal/proc` pass on many repeated runs in a row (state the count), also with other gate work running beside them.
- The change description names the root cause and how the fix closes it.
