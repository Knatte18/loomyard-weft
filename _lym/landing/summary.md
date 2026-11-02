# Report why fabricengine tier1 cannot use /dev/shm as its temp root

Adds a section, `fabricengine-shm-bench`, to `docs/benchmarks/test-suite-timing.md` after `test-time-audit-3`.
It answers whether the `internal/fabricengine` `tier1` run can have its test work directory in memory on OSL-1046, and it leaves the Cortex XDR question open.
The change is documentation only: no code, config or test changed.

## What the report finds

- With `TMPDIR=/dev/shm` the package cannot start.
  `tmuxkit.Main`, called by `internal/fabricengine`'s `TestMain`, refuses a socket directory whose longest socket path passes 107 bytes, and `/dev/shm/lym-<uid>/r<hex>` makes it 109 for this machine's 10-digit uid.
- A symlink `/tmp/s` to `/dev/shm` brings the path to 107 bytes, and the package then starts but 75 tests fail.
  Fabric's ownership checks compare the path a test was handed with the path git reports, and the two differ through the link.
- The files did land in memory, 35 MB at the peak, so memory is no obstacle.
- An unprivileged mount namespace for a short tmpfs is refused on this machine.
- Whether in-memory work avoids the Cortex scan is not measured.

## Design decisions

- Two of the six permitted runs were made, pair 1, and the report states they did not run together: the tool that started them ran the two commands one after the other, so the pair's ratio is no measure and the report gives none.
  Pairs 2 and 3 were not run, since the symlink setting fails the same tests under any load and no other short in-memory path is reachable without root.
  The brief's three concurrent pairs and its three ratios are therefore not delivered.
- The `/tmp/s` symlink was created for the run and removed afterwards; no config changed.
- The follow-up the report names, a shorter socket directory in `tmuxkit` before `TMPDIR=/dev/shm` can be tried again, is not filed: the run's board lock leaves the choice to the operator.
