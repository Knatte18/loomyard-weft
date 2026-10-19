# Every test package fails on a host with a 10-digit uid: tmuxkit socket path over 107 bytes

`tmux-socket-path-long-uid` · Note · bug, reed

Imported from [issue #739](https://github.com/Knatte18/loomyard/issues/739).

Seen on Linux host with uid 1856216086, in the implementer-seam Merge-In verify and reproduced with `lyx gate test ./internal/durfmt`; main fails the same way.

Every package fails before any test runs:

```
tmuxkit: socket directory "/tmp/lyx-1856216086/r859730815" is too long: its longest socket path would be 108 bytes, over the 107-byte limit; set TMPDIR to a shorter path.
```

The uid appears twice, in lyxtemp's root `/tmp/lyx-<uid>/r<rand>` and in tmux's `tmux-<uid>` socket dir, and with `maxKeyBytes` = 61 the longest socket path reaches 108-109 bytes (`internal/testkit/tmuxkit/tmuxkit.go`).

Workaround: none taken; the run holds until main is fixed.

Expected: the suite runs on any uid length, e.g. by giving tmuxkit a shorter socket directory or shortening lyxtemp's run-dir naming.