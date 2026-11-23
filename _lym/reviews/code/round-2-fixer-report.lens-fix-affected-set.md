```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's own set search (N1's autonomous-switch sites, N2's request-building and run paths) and check the fix covers every member.

**N1 (every site that tied the tmux row to the autonomous or interactive wording)**
- The search for `SwitchAutonomous`, `guardInputs{…autonomous…}`, `in.autonomous` and "refused in an autonomous" / "only for an autonomous" across `internal`, `contracts`, `pattern`, `docs` and `PATTERN.md` finds none left.
- The remaining `"autonomous"` hits are unrelated test labels: `settings_test.go:62`, `:616` and `:1109`, `driftengine/seats_test.go:199`, and `loomengine/discussiontable_test.go:133`.
- Both engines read only the new `AllowRawTmux` spec field.
- No finding.

**N2 (every path that builds a gate test request and runs it)**
- There are three construction sites: `cli.go:102`, `rungate.go:28` and `redfirst.go:93`.
- `cli.go` is the verb entry and the only one that sets `runAgent`. It runs `flagShapeRefusal` through both `cappedCall` (`detachfront.go:258`) and `runTest` (`run.go:43`).
- `rungate.go` and `redfirst.go` run gate-key tiers, which the fix leaves out on purpose. `doc.go` says a `--gate` key is exempt because `gate.yaml` sets its scope.
- The detached runner reads no strand name, but the front refuses a flagged call first, because `cappedCall` returns not capped when `flagShapeRefusal` is non-empty. So the refusal still happens in the agent's own call.
- No finding.

**Unchecked:** I did not rerun the search over stencils and contracts for prose saying a run agent may start a whole tier. The fix did not touch them.

**Tool calls (1 of 6)**
1. One `grep` across `internal`, `contracts`, `pattern` and `docs` for the N1 switch and wording sites and the N2 `testRequest{`, `runTest(` and `flagShapeRefusal(` call sites, because my context lacked a whole-repo view of either set.
