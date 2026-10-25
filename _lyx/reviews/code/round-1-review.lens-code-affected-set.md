```yaml
lens: code-affected-set
outcome: reported
```

Task: lens `code-affected-set`. For each change the diff makes, I looked for everything it forces outside the lines it touches.

**Per change: the set found, covered or not**

| Change | Set found | Covered |
|---|---|---|
| New `drift` config module | `configreg.Modules`, `cmd/lyx/configstrictness_test.go`, `configcli`, `envkit`, `docs/overview.md` | yes |
| `JOB_OBJECT_LIMIT_BREAKAWAY_OK` in `createKillOnCloseJob` | the loop's named job (`namedjob_windows.go:56`) and the `proc.StartTied` job (`tied_windows.go:14`) | yes |
| New `RoleCapture` | `runproc/record.go`, `runproc/doc.go:4`, `spawn.go:100` | yes |
| `drift-verify` gate key | `gateslot/template.yaml`, `tiers.go:87`, `config_test.go`, `envkit.fixedGateLists`, `loomshed/doc.go`, `PATTERN-gate-slot.md`, `PATTERN-test-speed.md` | yes |
| `gateslot/config.go:398` "landing" key list | not in the diff stat. It names `parentAlignGateKey`, `finalize`, `darn-write` and `code-burler-fix`, and Drift-Verify is not a landing site. | n/a, correct to leave |
| `agentActivityFor` also reads the capture lock and compare marker | `looparm.go`, `reviewstatus_test.go`, `checklist_test.go` | yes |
| `pairteardown.EndSession` calls `killCapture` | batten Worktree-Teardown and `lyx fabric remove` (both through `EndSession`), `adoptcli/deps.go:39` (the adopt's end, through `pairteardown.New`), `teardown_test.go`, `pairteardown/doc.go` | yes |
| `pairteardown` cancel path | `cancel.go` (see F1) | no |
| `DecisionHaltPrefix` doc widened | `bouncerdecision.go`; `AtDecisionHalt` in `verbs.go` and `haltReason` in `verify.go` both read the prefix | yes |
| New recipe rows | `Name*` constants, `InterruptPolicies`, the registry, `loomrecipe` tests, `shedrecipe/doc.go`, `loomrecipe/doc.go`, `shedbuild` test | yes |
| `Env.Drift` | `envkit` `nilLegal` and `driftDeps`, `wiring.go`, `battencli` seam tests | yes |
| Docs | `docs/overview.md`, `docs/dotnet-hub.md`, `gatecli/doc.go`, `gateslot/doc.go`, `reportengine/doc.go`, `shedadapters/doc.go` | updated |

**Findings**

- **F1, LOW, class `scope`:** `Cancel` does not end the pair's drift capture job.
  - Location: `internal/pairteardown/cancel.go:~108-120`, against `internal/pairteardown/teardown.go:265`.
  - `killCapture` is called only in `EndSession`.
  - The cancel path calls `stopStrands`, `killLoop`, `awaitRunLock` and `endSession`, and never `killCapture`.
  - The decision record's Constraints say "`pairteardown` ends a pair's processes, the capture included".
  - On Windows the capture child breaks away from the loop's job.
  - The cancel's `endOwnedAndProbe` ends it through its `runproc` record.
  - Its command runs in a `StartTied` job that closes when the child dies.
  - So a leak is unlikely, but the named job `proc.TerminateNamedJob` reaches is never terminated on cancel.
  - Fix: call `t.killCapture(req.Slug)` in the cancel path beside `killLoop`, or state in `pairteardown/doc.go` why cancel does not need it.
  - Confidence: moderate. I did not read `cancel.go` above line 85 or `doc.go` in full.

- **F2, LOW, class `consistency`:** the `Acceptance` and `ScopeRecord` doc comments say rulings apply under the wrong scope.
  - Location: `internal/driftengine/records.go`, the `Ruling` field ("under partial scope") and `RulingUnruled` ("a caller has not yet ruled on").
  - The code sets a ruling only under `ScopeFull` (`verbs.go` `Accept`), and D7 says `unruled` is the agent session's acceptance, not a caller's.
  - The const group comment reads "Whether an acceptance under partial scope was ruled on".
  - Fix: reword to "under scope full", and `RulingUnruled` to "an acceptance the agent session made that the run's parent has not yet ruled on".

- **Unverified:** `PATTERN.md` itself may be unchanged. The diff stat I viewed was truncated at the top, so I only saw the `pattern/*.md` background changes. The decision record says the design departs from no PATTERN entry, so this is not a finding unless the handler's own check shows a declared entry missing.

**Tool calls**
1. `grep` and `git diff --stat` over `proc`, `PATTERN.md`, `docs`, and the `EndSession` and `RoleCapture` callers. I needed this to confirm the `StartTied` job goes through `createKillOnCloseJob`, plus the caller sets.
2. `sed` of `cancel.go` and `grep` of `killCapture`, `killLoop` and the adopt callers. I needed this to check whether the cancel path and the adopt's end reach `killCapture`.
