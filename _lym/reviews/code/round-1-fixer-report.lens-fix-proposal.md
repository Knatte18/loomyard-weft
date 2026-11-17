```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F5, F6, F7, F8, F9). Where the change is my reading of the code, I say so.

Findings:

1. **LOW** — `internal/proc/memorycap_linux.go`, `probeTimeout = 5 * time.Second` (F2).
   - `hostMemoryScope` is a `sync.OnceValue`, so a probe that times out once is remembered for the life of the process.
   - A slow but healthy user manager, for example one under the `memory.high` throttle the review names, then reads as "cannot cap" for the whole gate run.
   - `MemoryCapCoversTree` goes false, `clampParallelToMemory` returns 1, and every test falls back to `RLIMIT_DATA`.
   - Nothing logs that the probe timed out, so the cause of the slowdown is hidden. `memoryCappedParallel` logs `cap_covers_tree=false` but not why.
   - This follows the proposal as written. A longer bound or a log line is the cheap mitigation.

2. **LOW** — `runMemoryCapped`, the `scopeStarted` false branch (F1).
   - The probe runs with the process's own environment, but each run uses `cmd.Env`. A hub `env.yaml` or a tier env without the user bus makes `systemd-run` exit 1 on every run.
   - The call now returns an error each time, with no fallback to `runDataLimited`. A gate with a mis-set env therefore fails every capped run instead of silently misreporting kills.
   - That matches the review's "return an error", but a per-run start failure never feeds back into `hostMemoryScope`.
   - I did not check whether `renderTierEnv` or `gateEnv` can drop `XDG_RUNTIME_DIR`. The tier 0 and tier 1 runs passed, so the fixture environments keep it.

3. **LOW** — `cmd/lym/spawnobservability_test.go` exemption reason and `internal/proc/doc.go` (F7).
   - The reword says the probe spawn "goes unlogged, its outcome logged through MemoryCapCoversTree". `MemoryCapCoversTree` only reports the outcome; the logging is in `memoryCappedParallel` in `gatecli`.
   - `internal/testcover` and `cmd/testtiming` call `RunMemoryCapped` and never log the probe's outcome. The reason is therefore true for the gate callers only.
   - PATTERN-child-processes asks that the exemption's reason hold, so this is a stale-reading risk, not a defect.

No other proposal I challenged held up as wrong:
- F1's sh marker: I built these cases and found none that fails.
  - A command killed by the kernel after the marker still reads as started.
  - A descendant holding fd 3 cannot block the read, because `exec 3>&-` runs first and `markerReadBound` bounds the read.
  - A cancel returns before the marker check.
  - Exit 127 from a missing command still counts as the command's own exit.
- F4's clamp reaches every use of `r.parallel` for test runs. The remaining `r.parallel` use is the `lym` build's `-p`.
- F5, F6, F8 and F9 break nothing I could find.

Calls made: none. My context already held every changed file.
