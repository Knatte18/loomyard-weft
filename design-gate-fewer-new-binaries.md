# Gate builds with -trimpath and verify runs the unit tests once

`gate-fewer-new-binaries` · Task · enhancement, config, shed

## Goal

Fewer test binaries that Cortex XDR has never seen, so a verify on OSL-1046 stops paying an 8-10 s hold for nearly every test package.
Operator decision (2026-10-20): both notes below are built together as one small darn task, and measured on OSL-1046 before and after (`bench-osl1046-xdr`).

## Note `gate-trimpath`

### Seen (orch, 2026-10-20, OSL-1046 with Cortex XDR)

- XDR holds each executable whose hash it has not seen for 8-10 s with idle CPU, and once it has judged a hash, any file with that hash runs at once: a copy of a judged binary in a new directory ran in 0.04 s.
- Without `-trimpath`, a Go binary embeds its source directory, so the same ten-line program built in two directories gave two hashes; with `-trimpath` both builds were byte-identical.
- The repo has 162 test packages, and each pair worktree has its own path, so every verify in a new pair links 162 test binaries per tier with hashes XDR has never seen, each paying the hold.

### Idea

- The gate adds `-trimpath` to the `GOFLAGS` it gives every slotted run (`internal/gateslot`, beside the `-p` cap), so a package whose code did not change links to a binary XDR already knows, in every pair and every run; only packages the change touches, and their dependents, pay the hold.
- `-trimpath` breaks the two test helpers that find the repo root from their own source path with `runtime.Caller` (`internal/testkit/scankit`, `internal/testkit/lyxbin`); they find it by walking up from the test's working directory to `go.mod` instead.
- `tools/deploy/main_test.go`, `tools/internal/devbin` and `internal/reedengine/selvagepane_enforcement_test.go` also call `runtime.Caller` and are checked the same way.
- Measure a verify on OSL-1046 before and after (`bench-osl1046-xdr`).


## Note `verify-unit-tier-twice`

### Seen (orch, 2026-10-20)

The verify command of `darn.yaml` (and the loom verify gates) is `lyx gate test ./... && lyx gate test --tags integration ./...`.
`go test --tags integration` builds and runs every untagged test as well as the tagged ones, so the unit tests run twice, and on OSL-1046 with Cortex XDR each tier also links its own set of test binaries that XDR holds 8-10 s each.

### Idea

- Verify runs only `lyx gate test --tags integration ./...`, or the first tier stays only as a fast fail when it is measurably faster to fail there.
- Check first whether any test is excluded by the `integration` tag (a `!integration` constraint), which would make the two runs differ.

## A re-run repeats packages that already passed (orch, 2026-10-20)

In `arbiter-verdict-quoting`, a Publish verify that failed on one tmux test sent the run back to Darn-Write, whose verify ran the whole suite again on an unchanged tree, minutes long on OSL-1046.
Go's test cache should make a re-run of an unchanged, passed package instant, so something defeats it: a different pair path per worktree (which `-trimpath` may cure), a test reading an environment variable whose value changes per gate slot (the gate sets the slot lock path and `GOFLAGS` for every slotted run), files a test opens that change between runs, or a flag that turns the cache off.
Operator view (2026-10-20): a re-run must not run again what already passed.
This task finds why the cache misses and makes a re-run on an unchanged tree reuse the passed packages' results; scoping verify to touched packages stays in note `verify-diff-scope`.


