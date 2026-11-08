# Gate Go runs use -count=1, so go test writes no test logs to the cache

## Why

With test caching on, `go test` keeps a log beside each cached result.
The log lists every environment variable the test read and every file it opened or stat'ed.
lym's tests build whole hubs and git repositories under fresh temp directories, so one log reaches hundreds of MB.
Gate runs wrote 60 to 275 GB of them a day into `~/.cache/go-build`.
The temp directories get new names every run, so no cached result is ever reused,
and the gate's own green record already carries a pass over an unchanged tree.

## What changes

- `gateslot.TestCacheOffFlag` (`-count=1`) is the flag that turns the test result cache off.
  `gateslot.WithTestCacheOff` returns an environment whose single GOFLAGS entry opens with it,
  followed by the value the child would otherwise have seen.
- `runTier` in `internal/gatecli/tierrun.go` runs every tier command under `WithTestCacheOff`.
  That covers each tier, the guard step and the vet step.
- `gateslot.HolderEnv` puts the flag first in the GOFLAGS of every non-tier slot holder's child:
  card-verify lines, drift steps, `cmd/testtiming` and planglyph's `go list` loads.
- The flag goes first, so a hub's own `-count` in a tier's GOFLAGS, or `-- -count=N` on the call, still wins.
- No hub config or reconcile is needed, and the build cache is untouched.
  `go build`, `go vet` and `go list` ignore a GOFLAGS flag they do not define.
- The `-trimpath` note keeps naming the GOFLAGS the hub and environment set, without the added flag.
- The package docs of `gateslot` and `gatecli` now say a fresh worktree reuses the build cache, not the test cache.
  `gatecli`'s doc gains a "Test result cache" section.

## Confirming it writes no test log

In the Go 1.26 toolchain source, `cmd/go/internal/test/testflag.go` injects a GOFLAGS `-count` as `-test.count=1`.
`test.go` then finds `-test.count` outside its cacheable flag list and sets `disableCache`,
and it passes `-test.testlogfile` to the test binary only when `disableCache` is false,
so a run under the gate's GOFLAGS writes no test log.

## Tests

- `TestLeaseEnv` in `internal/gateslot/pool_test.go` pins the new `HolderEnv` GOFLAGS values.
  It also gains `WithTestCacheOff` rows: no GOFLAGS, the last of several entries winning, and an empty value.
- The gatecli integration tests that read the stand-in's GOFLAGS now expect `-count=1 -trimpath -p=3`.
- planglyph's `loadEnv` integration test now expects `-count=1 -mod=readonly -p=3 -trimpath`.
