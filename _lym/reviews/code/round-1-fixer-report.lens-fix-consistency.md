```yaml
lens: fix-consistency
outcome: reported
```

No findings. Lens fix-consistency: I checked whether the F1 exception text agrees with its twins elsewhere.

- I grepped non-test `.go` and `.md` files for the never-died statements. The three sites F1 named (`engine.go`, the `wait.go` header, `docs/overview.md:420`) now carry the same exception as `Wait`'s doc (`wait.go:226-227`) and `checkLivenessTick`'s doc (`wait.go:1242`).
- The `errStrandNotTracked` doc (`wait.go:163-176`) and the `errStrandPaneBindingCleared` doc describe the sentinel errors themselves, not the outcome. They need no exception, since the stop-marked path returns `OutcomeDied` before the sentinel is reached.
- `attach.go:565`, `run.go:663`, `orchcli/live.go:57` and `orchroute/live.go:17` don't claim a `died` classification, so they are unaffected.
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md:179` says a blocked run must never end `outcome:"died"` when reed loses the strand. It describes a reed-side reset on a handle nobody stopped, so the exception does not apply and it stays accurate.
- The table-driven test's `OutputMissing` and `OutputPresent` rows match the "unless the files show the run done" wording in the `wait.go` header and `overview.md`.

Tool calls beyond the inherited context:
1. `grep` over `.go` and `.md` files for never-died and tracked-strand phrases: my context did not hold the other sites that make this claim.
2. `Read` of `wait.go:160-195`: my context did not show the `errStrandNotTracked` and `errStrandPaneBindingCleared` docs.
