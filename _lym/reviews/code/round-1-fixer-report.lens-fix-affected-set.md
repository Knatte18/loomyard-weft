```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran each set-premise finding's search to check the fix covers every member, not only the sites the review named.

**Findings**

- **LOW** — `internal/reedengine/panebin.go:96` (`composePaneLaunchLine`), reached from `spawn.go:119`.
  - **Summary:** F9's twin was left in strand panes. The env module's exports can still undo the build-server switches there.
  - **Evidence:**
    - The launch line is built as `sh.Chain(prelude, buildServerExports(sh), nameExports(...), launchCmd)`.
    - `spawn.go:119` has already folded `paneEnvExports` into `launchCmd`, so the module's exports come after `buildServerExports`.
    - A `MSBUILDDISABLENODEREUSE=0` in `env.yaml` therefore wins in a pane.
    - F9 fixed this ordering for gate runs in `run.go:136-139` and `gatecli/doc.go:40`, but `reedengine/doc.go:213` and `envconfig/doc.go` say nothing about it.
    - No test pins either order.
  - **Fix direction:** Either put the build-server exports after the module's exports, or state in the docs that the module may override them in panes.

**Checked, no gap**

- **F3 sites:**
  - Non-test callers of `cfg.Step(` are `driftcli/cli.go:322`, `driftengine/baseline.go:185` and `driftengine/verify.go:173`.
  - Each is a declared-or-not check whose step value is discarded.
  - Every site that hands a step to a run uses `StepWithHubEnv`: `cli.go:204`, `cli.go:391` and `loomcli/driftwiring.go:106`.
- **F4 launch sites:**
  - Every `launchStrandLocked` caller (`strand.go:424,459,628`, `lifecycle.go:870`) goes through `spawn.go:119`.
  - `window.go:98` (`composeWindowCommand`) gets no `PaneEnv`.
  - That is a detached `lym` window, not a strand pane, and the review did not ask for it.
- **F7/F8 text:**
  - The reservation text appears in `envconfig/config.go`, `doc.go` and `template.yaml`, and all three were updated.
  - No other source states it.
- **F1 env path:**
  - Every gate run (tiers, build, guard, vet) takes its env from `exe.env`, built once at `run.go:132-144`.
  - `detachfront.go:140` only strips the strand name for the detached runner, and that runner re-enters `runTest`.

**Unchecked:** F2, F5 and F6. Each is a single-site test gap, not a set.

**Tool calls**
- One Bash call ran five greps: `Step`/`StepWithHubEnv` callers (F3), `compose*`/`launchStrandLocked` callers (F4), `withBuildServerOff`/`BuildServerOffEnv` uses (F9), the env-module reservation text (F7/F8), and the env sources in `gatecli` (F1). My context holds only edited spans, not whole-repo caller sets.
