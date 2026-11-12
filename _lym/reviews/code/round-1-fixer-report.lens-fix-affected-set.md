```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: rerun the set searches for F3 (stale refresh wording) and F1 (other pipe-fed detached spawns) and check the fix covers every member.

Unchecked: F2, F4 and F5 are single-site coverage-gap additions, not set premises, so I ran no search for them.

- **F3:** the stale-wording search finds no remaining "test-map refresh the dobby started" or "end the test-map refresh" in `*.go`, `*.md` or `*.yaml`. Two hits remain, neither a miss:
  - `internal/dobbyengine/config.go:36` and `:132` ("test-map refreshes") describe the retired landing count, so they are accurate.
  - `docs/research/speed/testmap-upkeep-2026-11-11/proposal-fable-b.md:244` is a dated research proposal, not live documentation.
- **F1:** `spawnPass` (`internal/beamcli/dobbydeps.go:599`) still sends a detached batten pass's output to a `tailBuffer` pipe. It has the same SIGPIPE-after-dobby-death shape. The review scopes F1 to `startTestMap` and the fixer left `spawnPass` alone. The batten pass is meant to stop with its dobby, so I rate this out of scope and not a defect of the fix. No other `tailBuffer` user exists.

Tool calls:
1. `grep` for the stale refresh wording across go/md/yaml, to rerun F3's own search over the whole repo, not only orchcli.
2. `grep` for `tailBuffer` and `proc.Detach(` in non-test Go, to rerun F1's set of detached spawns whose output is fed through a pipe.
