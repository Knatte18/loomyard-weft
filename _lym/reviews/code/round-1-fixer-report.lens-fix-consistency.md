```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens, meaning twins and siblings of the changed text that were left stale.

**Findings**

1. **MEDIUM**, `internal/reedengine/spawn.go:119` with `panebin.go:96`, against the F9 change in `internal/gatecli/run.go` and the new `internal/gatecli/doc.go` text.
   - Summary: the F9 fix makes the gate's build-server switches win over the env module. The strand-pane launch line has the opposite order, so a module variable can still undo them in panes.
   - Evidence:
     - `composePaneLaunchLine` chains the prelude, then `buildServerExports`, then `nameExports`, then `launchCmd`.
     - `spawn.go:119` puts the module's exports inside `launchCmd` (`sh.Chain(e.paneEnvExports(...), launchCmd)`), so they come after the build-server exports.
     - A hub `env.yaml` with `MSBUILDDISABLENODEREUSE: "0"` therefore wins in an agent pane.
   - Why it matters: F9's premise was that the module must not undo the switches lym forces for every run. `gatecli/doc.go` now says "the module never undoes a build-server switch", while `reedengine/doc.go:213-214` says the build-server exports come "after the prelude" and the module exports precede the launch command, with no mention of the ordering.
   - Fix: either move the pane env exports ahead of `buildServerExports` in the composition, or state in `reedengine/doc.go` and `envconfig/doc.go` that the module may override the switches in panes.

2. **LOW**, `internal/envconfig/doc.go:10-11` precedence paragraph.
   - Summary: the paragraph still reads as complete precedence, with no mention of lym's build-server switches. The new `gatecli/doc.go` says they outrank the module.
   - Evidence: the paragraph lists only "a gate tier's or build's own `env`, or a Drift step's, then this module, then the environment lym inherits."
   - Fix: add the build-server exception here, or leave it as is if finding 1 is resolved by documenting that panes differ.

Checked with no discrepancy:
- The reservation text (`config.go` doc, the error message, `doc.go`, `template.yaml`) says the same thing everywhere.
- A grep for other mentions of the `LYM_` reservation found none in `.md` or `.yaml` files.
- `reservedPath` matches the `PATH` prelude in `panebin.go`.

**Tool calls**
1. `grep` for the reservation text and `BuildServerOffEnv`: to find sibling mentions my context lacked.
2. `sed -n 211,216p internal/reedengine/doc.go`: to read the reed doc's launch-line ordering text, which I had not read.
