```yaml
lens: code-affected-set
outcome: reported
```

Task: judge the code-affected-set lens of round 1 against the diff for the pause-gate inversion.

**Per change, the set found:**

1. `InstallNetworkGate` gains a `runCommand` parameter.
   - The callers are `cmd/lym/main.go:157` and the tests in `internal/tendercli/cli_test.go`, which the diff updates.
   - Covered.
2. The gate defaults to `ExemptAll`, so a process outside a run no longer waits at the gate.
   - `internal/adoptcli/doc.go:72-74` says a process the adopt worker starts, "a dobby or an adopt check among them", has a gate that "a pause still holds".
   - Now only the dobby is marked, through `beam dobby`.
   - An adopt check or another unmarked worker child passes unless it is a detached child or an agent strand.
   - The diff does not touch this doc, so it is wrong. See F1.
3. `SetGateExemptAll` callers are `lym adopt` (`internal/adoptcli/worker.go:55` and `caller.go:72`), `internal/fabriccli/fabric.go:1073`, `internal/fabricengine/remoteteardown.go:321` and `:432`, and `internal/battencli/cancelcmd.go:202`.
   - Each keeps a purpose: an agent or run-marked process can still run these verbs.
   - The doc comments in `internal/pauserecord/hold.go` and `internal/pauserecord/doc.go` were updated, so this set is covered.
   - `internal/adoptcli/doc.go` still says "An adopt is the operator's own takeover, so no pause holds it" and gives the exemption as the reason. That stays true but is now only half the reason (see F1).
4. The new env mark `LYM_RUN_PROCESS` and the scrubs that remove it.
   - `stripIdentityEnv` in `internal/reedengine/lifecycle.go` is updated.
   - `tmuxkit.ServerEnviron` is updated, with a string literal in place of the constant.
   - `internal/shuttleengines/shuttleengines_test.go` pins the two copies together.
   - The reed engine's `EnvScrub` type is declared in `internal/reedengine`, and the shuttle engines register their scrub as names and prefixes.
   - I found no other strip or scrub site.
   - Covered.
5. `RunProcessAnnotation` is set on `shed run`, `shed step` and `beam dobby`.
   - The `shedverbs` doc is updated.
   - The test lists `batten run`, `batten step` and `loom run` as true, which they get through `shedverbs`.
   - Other run-loop entry points are for the correctness and holistic reviews.
6. `PATTERN.md` and `pattern/`.
   - No declared PATTERN changes were given in the directive, so there is nothing to land.
   - No `Pattern-Departure:` trailer exists, and none is needed.
   - No pattern background describes the gate's default, so no pattern doc is left wrong.
   - `pattern/PATTERN-reed-panes.md:136-162` covers prompt holds, which this diff does not change.
7. Commit messages: the first commit explains the change. The two `fix:` commits carry no trailers and name no departure.

**Findings:**

- **F1** — MEDIUM, `consistency`, `internal/adoptcli/doc.go:70-74`.
  - The doc says a process the adopt worker starts, "a dobby or an adopt check among them", installs a gate "which a pause still holds".
  - After this diff only a marked, detached or agent process is held, and an adopt check is none of these.
  - Evidence: `internal/tendercli/gate.go:112-123` (`heldByPause`) and `internal/beamcli/cli.go:633` (only `beam dobby` is annotated).
  - Fix: reword the paragraph. The adopt worker is the operator's own process, and it sets the exemption only for the case where it is started from a run-marked or agent process. Name the processes a pause still holds as marked run processes and detached children.

**Tool calls:**
1. A grep for docs and stencils that describe the gate, the pause hold or the explicit-verb exemption. My context lacked a repo-wide doc search.
2. A grep for scrub or strip sites of `LYM_PARENT`, to find any other tmux server or pane env scrub. My context lacked a repo-wide search.
3. A grep of `pattern/` and `PATTERN.md` for pause descriptions, plus `sed` of `internal/adoptcli/doc.go:60-80`. My context lacked the adopt doc lines.
