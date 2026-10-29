## Goal

Measure one refill after a real change to `internal/logger` at gate parallelism 4 and at 8 on OSL-1046 with Cortex XDR, and record the held-package count and the verify time of each in `docs/benchmarks/gate-test-xdr.md`.

## Scope

- Two measured gate runs on one input, a change to `internal/logger`: one at `tier_parallel` 4 and one at 8.
- No other input, no repeats, no Codex.
- Only `docs/benchmarks/gate-test-xdr.md` is committed; the `internal/logger` edits are measurement inputs and are reverted before any commit.
- No code, PATTERN or `doc.go` change.

## Decisions

### D1 — The parallelism knob is `tier_parallel`

`go_parallel` is retired in the gate config, so the sweep sets `tier_parallel` of every declared tier to 4 and then 8, through the machine override, and removes the override afterwards.
Rationale: the retired key is dropped at load, so setting it would change nothing, and the machine layer keeps the change local to this machine.

### D2 — Verify is `lym gate test --wait --upto 2` under `/usr/bin/time -v`

The verify time is the wall time of that command, the call `CLAUDE.md` names for a push to `main`, and the held-package count is the `ok` lines of its tiers at 7 s or more, the figure the file already uses.
Rationale: it runs every non-manual tier whole-scoped over the changed tree, so it measures the refill the entry asks about.

### D3 — Each run gets its own logger change

Run 1 and run 2 each carry a different functional edit to an unexported value in `internal/logger`, so run 2 cannot hit the test cache run 1 filled and both links produce binary hashes XDR has not seen.
The tree is restored with `git checkout` before and after each run.
Rationale: one shared edit would make the second run a cache hit and measure nothing.

### D4 — Quiet-machine rule

Each run starts once the 1-minute load average is below 4, with no batten run live, and the load before each run is recorded.
Rationale: the file's earlier refill ran at a load of 10.88, which the entry calls out as the confound.

### D5 — A failing test is reported, not retried

A failed test stays in the row as a failure; the cap forbids a repeat.
Rationale: the cap is exactly two measured runs.

## Constraints

- Cap: exactly two measured runs; no repeats, no other input, no Codex.
- Hardware info is the file's `Machine` block, updated only where it has changed.
- Markdown follows semantic line breaks, one sentence per line.

## Auto-mode assumptions

- The dependency `creel-flakes-and-adopt-names` has landed: the board lists it done.
- "Verify time" means the wall time of D2's command.

## Open risks

- Run-to-run noise exceeded the effect in the earlier sweep, so two runs may show no separable difference; the report says so rather than infer one.
- The `gate` config reads the hub default and machine layers; a machine override left behind would affect later runs, so it is removed after the runs.

## Acceptance criteria

- D1: the doc names `tier_parallel` as the setting swept and says it replaces the retired `go_parallel`; no override remains in the machine layer afterwards.
- D2: the doc gives, for each of the two runs, the wall, user, sys and CPU times, the packages run, cached and failed, and the held-package count.
- D3: the doc names the two logger edits and states that each run started from a restored tree.
- D4: the doc gives the 1-minute load average before each run.
- D5: a failed test, if any, shows in the row and no run is repeated.
- `lym loom validate-discussion` and `lym loom validate-description` exit 0, and the commit holds only the doc and no `internal/logger` change.
