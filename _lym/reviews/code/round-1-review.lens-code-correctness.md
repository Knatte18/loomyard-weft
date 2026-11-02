```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's changed code.

**List (function or branch, case run, verdict)**

- `randomRunName` (`internal/lymtemp/run.go:72`): `rand.IntN(36)` returns 0..35, so the index is always in range for the 36-character alphabet. The result is `r` plus 4 characters, 5 bytes. Correct.
- Byte arithmetic for the longest socket path:
  - `/dev/shm` (8) + `/lym-1856216086` (15) + `/rxxxx` (6) = 29.
  - `socketDir` adds `/tmux-1856216086` (16), giving 45.
  - `filepath.Join` adds `/` and 61 key bytes (62), giving 107.
  - `checkSocketPath` refuses only when `len > 107`, so 107 passes. This matches the sun_path limit of 108 less the NUL byte.
  - The old name gave 109. Correct.
  - `filepath.Join` with the all-NUL key does not clean away the key bytes, because NUL is not a separator.
- `newRun` retry path with the new alphabet (`run.go:49-62`): a taken name returns `fs.ErrExist` and is redrawn up to 16 times. Exhaustion returns an error naming `r.path`. With about 1.68 million names, 16 consecutive collisions against N live dirs have a probability of roughly (N/1.68M)^16, which is negligible. The unchanged failure path stays correct, and `run_test.go:78-87` pins it.
- Sweep and inherited-dir detection: both match on `runPrefix` only (`sweep.go:54`, `root.go:54`). Neither assumes hex or a name length, so both are unaffected. Correct.
- `TestCheckSocketPath` new row (`tmuxkit_test.go:35`):
  - It builds `/dev/shm/lym-1856216086/<Base(lymtemp.Dir())>`.
  - `lymtemp.Dir()` always returns a run directory named `r` plus 4 characters, so the case is deterministic at 107 and expects no error. Correct.
  - It would fail at 109 with the old six-digit name, so it pins the fix.
  - It calls `lymtemp.Dir()` inside a tier0 test. The process run directory already exists from `Main`, so nothing new is created.
- `TestNewRun_HoldsItsLockUntilRemoved` regex `^r[0-9a-z]{4}$` (`run_test.go:32`): matches the generator exactly. Correct.
- Error and state paths in `newRun`: the `run.go` diff touches none of them, so no new inconsistent state on failure.

**Findings**

1. NIT, `consistency`: the existing row "run directory of the widest uid" (`internal/testkit/tmuxkit/tmuxkit_test.go:34`) still uses the retired shape `r0a1b2c` (six hex digits). The row passes, but its name no longer matches what `lymtemp` creates.
2. No BLOCKING, MEDIUM or LOW correctness defects in the changed code.

**Tool calls:** none. The manifest already held every definition I needed.
