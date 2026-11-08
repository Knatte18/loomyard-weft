# Run the module's main packages in the last Webster batch gate

A batch gate tests only the packages its cards name.
So a change that breaks a `cmd/lym` test was first seen at the plan-level verify, after the fork that caused it had ended.
`docs/research/webster/webster-time-2026-11-08.md` found `cmd/lym` among the failing ids in 5 of the 8 runs whose first verify failed;
each such run paid a verify-fix fork and a second verify, about 6.8 min.
The last batch's gate now also runs the module's `package main` packages, so the last fork meets and fixes such a break itself.
The plan-level verify is unchanged.

## What changes

- **`lym gate test --main-packages`.**
  With `--card` under the `go` scoper, the call adds every `package main` package of tier0's module to the card scope.
  It lists them with the new `impactset.ModuleMainPackages` (`go list -e ./...`), so a main package that fails to build stays listed.
  A main package a card already names is listed once.
  The added packages go through the scope's vet step, watch-skip and guard scan like any card package.
  The `none` and `dotnet` scopers add nothing.
  `--main-packages` without `--card` is refused, naming `--card <id>` as the way forward.
  The detached runner's call words carry the flag, so a capped call and its runner agree on the scope.
- **Webster.**
  begin-batch and recover-batch render the batch gate of the last batch in the execution order through `gateslot.Config.AgentGateCallWithMainPackages`.
  That call is the usual `lym gate test --gate batch --card …` with `--main-packages` appended.
  The other batches' calls are unchanged.
- **Implementer body.**
  A batch gate call that ends with `--main-packages` belongs to the plan's last batch.
  A failure in the packages that flag adds is the last fork's to fix, even when an earlier batch caused it.
  This is the one exception to the rule that leaves a failure the batch's cards did not cause to the plan-level gate,
  and it is carried into the self-fix bound and the report's `status` rules.
  `TestStencils_NameGoValues` pins the flag name in the stencil.

## Design decisions

- **Last batch only, from the gate run records.**
  Since 2026-11-01, `.lym/gate/runs/*.json` show `cmd/lym` taking a median of 10.0 s at tier0 and 8.8 s at tier1;
  a `cmd/lym`-only tier0 call took 8.4 s of wall time on average.
  The hub's `batch` key runs tiers 0 and 1, so the package adds about 20 s per batch gate.
  The other main packages (`cmd/testtiming`, `tools/*`) add about 2 s more.
  Over the report's 14 runs (85 batches) that is about 28 min on every batch gate, against about 34 min of verify-fix time saved (5 runs at 6.8 min):
  close to break-even, before the fork's own fix rounds.
  On the last batch alone it is about 5 min (14 gates), so it saves about 24 min, roughly 1.7 min per run.
  The saving is an upper bound: the report does not say whether `cmd/lym` was the only failing id in those 5 runs.
- **Derived, with no config.**
  The packages are derived from the module rather than named in `gate.yaml` or `language.yaml`.
  So loomyard gets `./cmd/lym` without a hub step, and no template carries a Go path.
  The derivation lives in the `go` scoper, which is already Go-specific.
- **Ownership in the stencil, not in Go.**
  The rule that a last-batch failure in a main package is the fork's to fix lives in the implementer body and keys on the rendered flag.
  No prompt text is added in Go, and the body needs no new marker.
  With an older deployed body, the last fork still runs the main packages, but it leaves a failure an earlier batch caused to the plan-level gate.
- **Overlap runs twice.**
  A card target such as `./cmd/lym/...` next to the derived `./cmd/lym` leaves both in the scope, so that package runs twice.
  This costs time only and is left as is.

## To check by hand

- The timing numbers above were not re-derived against the gate run records for this description; confirm them against `.lym/gate/runs/*.json` in the hub.
- The flag's reach into the last batch is not pinned by a test in `internal/websterengine/beginbatch.go`: nothing asserts that begin-batch passes the last-batch flag.
- The detached runner's call words for `--main-packages` (`internal/gatecli/detachfront.go`) are not pinned by a test that covers a capped call re-run by its runner.
- No end-to-end Webster run with a `cmd/lym` break in an earlier batch was done; run one to see the last fork fix it.
