# Sweep go_parallel on OSL-1046 with Cortex XDR

`bench-osl1046-xdr` · Note · audit

## Done (2026-10-20)

The before and after measurement of `-trimpath` is in `docs/benchmarks/gate-test-xdr.md`.
The `go_parallel` sweep at 4, 8 and 12 is in the same file (`03af4c75`): no value gave a real gain, 8 was 6 % under 4 while two runs at 4 differed by 36 s, and no run failed, so `go_parallel` stays 4.

## Seen in the sweep

With `-trimpath` the build is reproducible: a fresh `GOCACHE` links byte-identical test binaries whose hashes XDR already knows, so it holds none of them.
The sweep forced new hashes with a random `-ldflags=-B` build ID, and even then few packages were held; `internal/fabricengine` (117-175 s) bounds the tier's wall time, and the rise at 8 and 12 is contention.

## Still open

The case the sweep set out to cover is unmeasured: a refill after a real change to a widely imported package, such as `internal/logger`, which showed about 50 held packages and a verify of about 6 min.
Whether a higher `-p` overlaps those holds needs one such run at 4 and one at 8.
