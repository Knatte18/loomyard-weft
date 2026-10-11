# Webster round 2 — fixer-report

## Changes

- **F1** (BLOCKING, design) — fixed differently, in commit `ea4e53819`.
  The premise holds.
  The failing row built its stale socket by closing a listening `net.UnixListener`.
  A child that a parallel test forks before `Close` holds a copy of the listening fd until it reaches `exec`, so `Dial` can succeed and `removeStaleSocket` keeps the file, as its doc comment says it should.
  Neither the branch's range nor merge `916d8ec72` touches `internal/reedengine`, which fits a load-dependent race rather than a regression.
  The new `bindStaleUnixSocket` helper in `internal/reedengine/socketfile_test.go` binds a `unixgram` socket and closes it.
  A datagram socket never listens, so a stream `Dial` to its path fails no matter which process holds the fd.
  Both stale rows, in `TestRemoveStaleSocket` and in `TestRemoveSocketFileOnceGone`, now use it.
  This is the review's bound-but-not-listening option.
  I did not take its proposal of a `syscall` bind helper split by build tag.
  `net.ListenUnixgram` is portable and needs no build-tag split, and the Windows rows are skipped anyway.
  The serial-test option was rejected too: it would give up `t.Parallel` against `PATTERN-test-speed`.
  Every remaining `listenUnixSocket` caller passed `unlinkOnClose` as `true`, so I dropped the parameter and the `listen` alias.
  Verified with `lyx gate test ./internal/reedengine`, `lyx gate test ./...`, `lyx gate test --tags integration ./...` and `lyx loom lint-comments`, all green.

## Disputed

None.

## Deferred

None.
