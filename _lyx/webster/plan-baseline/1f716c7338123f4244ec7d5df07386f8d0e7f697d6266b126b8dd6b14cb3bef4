# Card 15 — reedengine-faketmux-rest

**Edit:**
- `reedengine/generation_test.go`
- `reedengine/launchscript_cleanup_test.go`
- `reedengine/lifecycle_test.go`
- `reedengine/namerepair_test.go`
- `reedengine/overlay_test.go`
- `reedengine/probe_test.go`
- `reedengine/reconcile_test.go`
- `reedengine/selvagepane_test.go`
- `reedengine/spawn_test.go`
- `reedengine/strand_test.go`
- `reedengine/watchdog_integration_test.go`
- `reedengine/windowsize_test.go`
- `reedengine/faketmux_test.go`

**Intent:**
Finish #325 §3: every remaining `e.tmux.execHook = ...` closure in reedengine tests becomes a `fakeTmux` script, so reedengine tests use one fake.

Each `switch args[0]` closure becomes per-verb answers on the fake, and each ad hoc slice-append recorder becomes an assertion on `Calls()`, `Sequence()` or `ArgvFor`.
Every scripted answer keeps its exact value.
A closure whose behaviour the fake cannot express (a side effect on disk between calls, a call-count-dependent answer) is served by extending `fakeTmux` with the general mechanism it needs, such as an optional per-verb answer function, not by keeping the closure.
`watchdog_integration_test.go` is integration-tagged and keeps its tag.

After this card, `grep -n 'execHook =' internal/reedengine/*_test.go` matches only `faketmux_test.go`.
Apply D10 to every touched test.

**ImpactSummary:** The remaining reedengine test files switch their tmux closures to the shared fake; fakeTmux may gain a general per-verb hook, assertions unchanged.
