```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. For each change the diff makes, I listed what reads, asserts or documents it and checked each against the diff.

**Per change: set found, covered or not**

1. **Holder record `fabric.push.holder` (new file shape in the weft lock dir).**
   - Writers: all four takers of `fabric.push.lock` are wrapped. They are `coalesceFlagged` (`pendingflags.go:139`, which serves `PushAnchored`, `PushPairAnchored` and the detached child), `lockWeftForReplay` (`pull.go:500`) and `tryRecordsLocks` (`originreconcilemoves.go:96`, weft only). That covers every `weftPushLockFile` taker in non-test code, per my grep. Covered.
   - Bolt's `board.push.lock` is deliberately not wrapped. It never returns `ErrPushLockBusy`, so this is correct.
   - Readers: only `pushLockBusyError`. Covered.
   - Lock-dir hygiene: the `.weft` dir is untracked (`weftgit_exclude_test.go:89` asserts `.weft` is not in status), so the new file cannot dirty a worktree. Covered.
   - Destructive guard: `cmd/lym/destructiveguard_test.go` allowlist entry added for `pushlockholder.go`. The `os.Remove` there targets only the holder path, so the entry's reason matches the code. Covered.
   - PATTERN-tolerant-records: the decode never uses `DisallowUnknownFields`, and the `unknown:1` row in `TestDescribePushLockHolder` shows an unknown key is accepted. The write goes through `fsx.AtomicWriteBytes` rather than `internal/state`, but `pendingflags.go:83` already does the same for its JSON flags. The record is replaced whole by each holder and never read-modified-written, so there is nothing to preserve. Not a finding.
   - Docs: `doc.go:776-779` describes the record and its stale-record handling. No `PATTERN.md` or `pattern/` entry describes the push lock or its directory (`PATTERN-fabric-git`, `PATTERN-push-both-sides` and `PATTERN-remote-refs-lock` do not). No declared PATTERN change exists in this round. Covered.

2. **`ErrPushLockBusy` message text** (`still held after 30s, <holder>; neither side was pushed`).
   - Matchers: callers and tests match by `errors.Is` (`commitrecords_test.go:144`, `pushanchored_integration_test.go:523`, `landingdeps.go:396`), and the `%w` wrap is kept at the one construction site.
   - Text assertions: my grep found no non-test code or test asserting "still held after" or "neither side was pushed" for this error. The other "still held after" hits belong to unrelated locks.
   - Call sites: `pushLockBusyError` is returned from both `PushAnchored` (`pushanchored.go:113`) and `PushPairAnchored` (`pushanchored.go:215`). The holder text therefore reaches every site that returns the error. Covered.
   - Doc comments: `ErrPushLockBusy`'s own comment and `pushLockBusyError`'s are updated. The `PushAnchored` and `PushPairAnchored` doc comments (lines 72-75 and 128-130) stay true as written. Covered.

3. **`runHeld` signature (`*lock.FileLock` replaced by `release func() error`).**
   - Callers: `coalesceFlagged` is the only caller in the inherited context.
   - Doc: the `runHeld` comment and the `coalesceFlagged` comment are updated. Covered.

4. **`pushRidingOutBusyLock` and `landingPushRetryPauses` (landing `PushTask` retry).**
   - Callers: only `pullRequestDepsGuardedBy`. The other `PushPairAnchored` or `PushAnchored` callers keep their 30s behaviour, which the summary states as intended for `commit-records`.
   - The `commitrecords_test.go:144` row that maps a busy lock to the transient way forward is unaffected.
   - Docs: the comments on the new symbols are present. No loom or landingshed doc or stencil describes the push step's retry, per my search, so no doc is left wrong. Covered.

**Findings**

- **LOW, consistency** — `internal/loomcli/landingdeps.go:350`. The retry sleeps through `time.Sleep`, passed in as `sleep`, for up to 3.5 minutes. The same Deps struct wires `Sleep: sleepContext` at line 380 for its other waits. A pause or cancel arriving during a retry pause is not seen until the pause ends. Fix: if `sleepContext` honours pause or stop, route the retry through it. Otherwise leave as is, since the evidence for that behaviour is outside my context.

Nothing else is left wrong by the diff within this lens. The commit messages carry no `Pattern-Departure:` trailer and none is needed.

**Tool calls**
1. One `grep` across `*.go`, `*.md` and `*.yaml` for the busy-error text, `fabric.push.lock`, the pending flags and `.weft`. It found other readers, asserters and documenters of the lock directory and error text that my context lacked.
