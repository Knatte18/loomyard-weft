# Gate builds with -trimpath so unchanged test binaries are byte-identical across worktrees

`gate-trimpath` · Note · enhancement, config

## Seen (orch, 2026-10-20, OSL-1046 with Cortex XDR)

- XDR holds each executable whose hash it has not seen for 8-10 s with idle CPU, and once it has judged a hash, any file with that hash runs at once: a copy of a judged binary in a new directory ran in 0.04 s.
- Without `-trimpath`, a Go binary embeds its source directory, so the same ten-line program built in two directories gave two hashes; with `-trimpath` both builds were byte-identical.
- The repo has 162 test packages, and each pair worktree has its own path, so every verify in a new pair links 162 test binaries per tier with hashes XDR has never seen, each paying the hold.

## Idea

- The gate adds `-trimpath` to the `GOFLAGS` it gives every slotted run (`internal/gateslot`, beside the `-p` cap), so a package whose code did not change links to a binary XDR already knows, in every pair and every run; only packages the change touches, and their dependents, pay the hold.
- `-trimpath` breaks the two test helpers that find the repo root from their own source path with `runtime.Caller` (`internal/testkit/scankit`, `internal/testkit/lyxbin`); they find it by walking up from the test's working directory to `go.mod` instead.
- `tools/deploy/main_test.go`, `tools/internal/devbin` and `internal/reedengine/selvagepane_enforcement_test.go` also call `runtime.Caller` and are checked the same way.
- Measure a verify on OSL-1046 before and after (`bench-osl1046-xdr`).
