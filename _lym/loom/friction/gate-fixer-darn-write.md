# Fixing the darn-write comment-wrap failures

`lym gate test --gate darn-write` ran twice and failed tier1 both times on the memory guard, not on a test.
The user slice ran low on its 16 GB and the guard killed the largest test binary:
`cmd/lym` in both runs, and `internal/gatecli` as well in the first.
The guard's advice to find the test that holds memory without bound does not fit a comment-only change.
`cmd/lym` passed tier1 when run alone.
