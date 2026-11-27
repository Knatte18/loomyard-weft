# Cut the fixed steps of a tier1 gate call from about 6 s to about 2 s

`gate-fixed-cost-cuts` · Note · enhancement, gate

## Seen

The study `docs/research/speed/gate-fixed-cost-2026-11-27/report.md` timed a tier1 call on one small package at about 6 s, of which:

- `go build ./...` takes 3.0 to 3.9 s on an unchanged tree, since it links every `package main` and discards the binaries;
- the `lym` build in `prebuildLym` (`internal/gatecli/prebuild.go`) takes 2.4 to 2.8 s on every call, whether or not a test uses the binary;
- `go vet`, the test link and the gate's own work cost well under a second together.

## Idea

1. In hub config, replace `go build ./... && go vet {targets}` in `gate.tiers.tier1.command` with `go vet ./...`.
   Cost: a link error in a command without tests is no longer caught by tier1.
2. In `prebuildLym`, build `lym` only when a test file of a targeted package imports `internal/testkit/lymbin`,
   after checking that no test reaches the prebuilt binary another way.

Together, a small call's fixed steps go from about 6 s to about 2 s, by the report's estimate.
