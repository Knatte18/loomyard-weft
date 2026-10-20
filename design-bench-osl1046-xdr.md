# Sweep go_parallel on OSL-1046 with Cortex XDR

`bench-osl1046-xdr` · Note · audit

## Done (2026-10-20)

The before and after measurement of `-trimpath` is in `docs/benchmarks/gate-test-xdr.md`.

## Next: a `-p` sweep (operator, 2026-10-20)

Cortex XDR holds each unseen test binary 8-10 s with idle CPU, and holds run in parallel, so with `go_parallel: 4` at most four binaries wait at once and a verify after a change to a widely imported package, such as `internal/logger`, takes about 6 min on OSL-1046.
A higher `-p` overlaps more holds, but the integration tests then run more packages at once, and they create and tear down many git repositories (366 s sys against 322 s user in the uncached before-run), so contention or flaky timeouts may cancel the gain.

Once `gate-fewer-new-binaries` has landed and the machine is quiet, clock one uncached integration tier (`lyx gate test --tags integration ./... -- -count=1`, with new binaries for XDR, for example after a change to `internal/logger` or a cleared build cache) at `go_parallel` 4, 8 and 12, recording wall, user and sys time and every failing or flaky test.
Set `go_parallel` to the fastest value with no new failures, and log the numbers in `docs/benchmarks/gate-test-xdr.md`.
Operator view: the worst case is that XDR stops the whole run.
