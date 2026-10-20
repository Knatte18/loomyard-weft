# Verify runs the unit tier once, reuses passed results on a re-run, and runs a failed test first

`gate-fewer-new-binaries` · Task · enhancement, config, shed · running · Merge-In

## Goal

Fewer test binaries that Cortex XDR has never seen and fewer re-runs of passed tests, so a verify on OSL-1046 stops paying an 8-10 s hold for nearly every test package.
Measure on OSL-1046 before and after, in `docs/benchmarks/gate-test-xdr.md`.

## Done in the prime: `-trimpath` (2026-10-20, `2d384150`, `1552ceb4`)

On the operator's call the former note `gate-trimpath` was built directly in the prime: `gateslot.GoFlags` adds `-trimpath` to every slotted run, and the `runtime.Caller` repo-root lookups were made trimpath-safe (`PATTERN-trimpath-safe-root`).
`docs/benchmarks/gate-test-xdr.md` holds the numbers: the cached unit tier fell from 98.6 s to 20.3 s wall, the cached integration tier from 179.5 s to 112.0 s, and a copy of the tree in another directory reused 63 of 162 packages' cached results with no XDR hold.

## Note `verify-unit-tier-twice`

### Seen (orch, 2026-10-20)

The verify command of `darn.yaml` (and the loom verify gates) is `lyx gate test ./... && lyx gate test --tags integration ./...`.
`go test --tags integration` builds and runs every untagged test as well as the tagged ones, so the unit tests run twice, and on OSL-1046 with Cortex XDR each tier also links its own set of test binaries that XDR holds 8-10 s each.

### Idea

- Verify runs only `lyx gate test --tags integration ./...`, or the first tier stays only as a fast fail when it is measurably faster to fail there.
- Check first whether any test is excluded by the `integration` tag (a `!integration` constraint), which would make the two runs differ.

## A re-run repeats packages that already passed (orch, 2026-10-20)

In `arbiter-verdict-quoting`, a Publish verify that failed on one tmux test sent the run back to Darn-Write, whose verify ran the whole suite again on an unchanged tree, minutes long on OSL-1046.
Operator view (2026-10-20): a re-run must not run again what already passed.

The `-trimpath` measurement narrows it down:
- Even in the prime, on an unchanged tree, a second plain run took only 97 of 162 unit packages and 76 of 162 integration packages from the cache; the rest ran again.
- In a copy of the tree in another directory, 34 packages cached in the prime missed, among them the source scans (`internal/testkit`, `internal/pattern`, `contracts/stencils`): Go keys a cached result on the files a test opened, and every file in a fresh worktree is new to it.

This task finds why the uncached packages miss on an unchanged tree (a test reading an environment variable that changes per gate slot, such as the slot lock path, files that change between runs, or a test that turns the cache off) and makes a re-run on an unchanged tree reuse the passed packages' results; scoping verify to touched packages stays in note `verify-diff-scope`.

## A verify after a failure runs the failed test last (orch, 2026-10-20)

After the Publish failure, Darn-Write's verify ran `lyx gate test ./... && lyx gate test --tags integration ./... && go test -tags tmux -run '^TestOpenWindow_RealTmux$' <pkg> && go test -tags tmux ./...`: the test that failed came third, after two full tiers.
The failed test runs first, so a failure that still holds is known in seconds, and the full tiers run only once it passes.
