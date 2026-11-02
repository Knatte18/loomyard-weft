```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behavior the diff adds or changes, I name the test that pins it, or none.

**Per behavior: pinning test**

1. **Run name is `r` plus four characters from `[0-9a-z]`.**
   - Pinned by `TestNewRun_HoldsItsLockUntilRemoved`, the regexp `^r[0-9a-z]{4}$` at `internal/lymtemp/run_test.go:32`.
   - A change to the length or the alphabet fails it. The check is statistical for the alphabet, but the length is exact.

2. **The longest socket path for a ten-digit uid under a `/dev/shm`-length root is at most 107 bytes.**
   - Pinned by the new row in `TestCheckSocketPath`, `internal/testkit/tmuxkit/tmuxkit_test.go:35`.
   - The row builds `/dev/shm/lym-1856216086/<base of lymtemp.Dir()>` and expects no error.
   - The base is `r` plus four characters, so the path is 107 bytes. It would be 109 bytes with the old six-hex name, and the row would fail.
   - The row is deterministic because the name length is fixed.
   - It couples the test to the real run directory name. That is intended: it fails if `runNameDigits` grows.

3. **`checkSocketPath` is inclusive at 107 bytes.**
   - Pinned by the existing row "directory at the limit" at `tmuxkit_test.go:31`. The new row also sits exactly at 107.

4. **The retry bound and the taken-name error naming the root are unchanged.**
   - Pinned by `TestNewRun_RetriesATakenNameUpToTheBound`, `run_test.go:59`.
   - The test stubs `runName`, so it is independent of the name shape. It is unchanged and still valid.

5. **`randomRunName` draws from the 36-character alphabet.**
   - Only the regexp in behavior 1 covers this, over one draw.
   - No test pins the sweep or the inherited-run-directory logic to the new shape. Both key on `runPrefix` alone (`sweep.go:54`, `root.go:54`), which the diff does not change.
   - `root_test.go:153` uses the `r123456` shape. It still passes because only the prefix matters. That is a stale shape in a fixture, not a coverage gap.

**Findings**

- **NIT, consistency.** `tmuxkit_test.go:34`, the "run directory of the widest uid" row, still uses the six-hex name `r0a1b2c`, a shape that no longer exists.
  - The new row now covers the widest ten-digit uid at the real shape.
  - The old row is not a duplicate: it uses a `/tmp` root and passes either way. It is just stale.
  - Evidence: `tmuxkit_test.go:34` against the new row at line 35.

- **NIT, consistency.** `root_test.go:153-154` builds inherited run directories as `runPrefix+"123456"`, the old six-digit shape.
  - It is functionally fine, since only the prefix is matched.
  - Evidence: `internal/lymtemp/root_test.go:153`.

- **No coverage gap.** Every behavior the diff changes has a failing row if broken.
  - No new duplicate test was added: the new row is a row in an existing table, which fits the test-economy rule.
  - The docs change is a measurement report with no code behavior.

**Tool calls:** none. My inherited context already held every file read, including the `root_test.go` call sites from the grep output.
