```yaml
lens: code-scope
outcome: reported
```

**Task:** code-scope lens, mapping the diff's changes to decisions D1–D11 and back. The record has no "Added after Discussion" entries and the plan has 18 cards.

## Per change: decision traced to

| Change | Decision |
|---|---|
| `beamsetting.AutonomyKey`, dobbyengine `ConfigSpec`, `LoadConfig`, `CheckLocalValues`, `checkTaskSetting`, `template.yaml`; boardcli help; configreg row | D1 |
| Finalize `configChangeNotice` skips `beam.autonomy` | D1 |
| `dobbyengine.LookupAutonomy` | D11 and D1 |
| `hubgeom.Autonomy`, `resolvePrimeWithoutGit` | D2 |
| `NotifyParentHalt`, `NotifyRunFromHalt`, `withRunAutonomy`; batten `notifyRunParent`, drivewiring hand-back and prepush `notify` rewired | D2 (partial, see F1) |
| drift `ApplyAutonomy`, `AutonomyForcedSetting`, `Config.Autonomous`; `driftDeps` config wrap | D3 |
| `verify.go` ended-ask halt and re-ask; `judgeparent.go` cleared ask; `burlerengine.ClearParentAsk` and `Cleared` | D3 |
| loom `ApplyAutonomy`, `AutonomyForcedSetting`, `pr_review_parent_timeout_min`; `wire()` and `autonomySettingLookup` | D3 |
| `pullrequest.Record.Autonomy`, `ClearReviewSession` | D4 |
| `prsession.go` record-once, `Interactive`, two stencil blocks, stencil restructure, `stencils.go` registry | D4 |
| `prreviewask.go` (asker, line, standing); `PullRequestDeps` seams; `askPRReview` | D4 |
| `judge`, `judgeParentAnswer`, `mailStayOpen`, `gateRetryHint` | D4 |
| `parentReviewAskVerb`, answer way-forward, group help | D4 |
| `prcontinue.go` | D4 |
| Finalize closed-pull-request halt | D4 |
| `prreviewgoto.go`; shedengine `Admitted`; shedverbs `GotoAdmitted`; loomcli wiring (`arm.go`, `arbiterconsume.go`, `drivewiring.go`) | D4 |
| pawlengine `noticeDrain`, `DrainMarker`, deps; `pawlwiring` seams; hubgeom `PawlPaths`; shedrecipe `requirePawl` | D6 |
| slackcli `operatorMarker`, `orchAutonomous` | D8 |
| `role-slack.md` marker rule | D8 and D9 |
| `role-autonomy.md`, `orch-role.md` exceptions, beamcli Long help texts, `orchrole.go` embed | D5, D7, D9 |
| `WithLaunchFacts`, `Paths.Autonomy`, watcher and `ResumeContext` callers, `orchcli` start and paths | D9 |
| glossary lines | D10 |
| shuttleparity, envkit, `wire_seams_test` `intentionallyNil` entries, doc.go edits, `docs/overview.md` | Forced by the traced changes |
| `PATTERN-config-layers` per-task line | Forced by D1 and D11 |
| `PATTERN-daemon-wakeups` bullet for the judge re-ask wait | Forced by D3 (the wait loop in `reaskParent`) |

No change traces to no decision, so I found no unrequested scope.

## Per decision: changes that build it

D1 through D11 each have at least one building change, per the table.

## Findings

**F1 — MEDIUM, scope.** D2 says every halt notice lym posts for a run carries `autonomy: &lt;true|false&gt;`. Only three sites were converted: batten `notifyRunParent`, the loom hand-back `Notify`, and the `prepush` notify. Other run notices still go through the unstamped `NotifyParent` / `NotifyRunFrom`. They may be halt or run-end notices that an orch handles under its own value:
- `internal/loomcli/cli.go:211` (`c.notifyParent`)
- `internal/loomcli/landingdeps.go:279` (landing `Notify`)
- `internal/loomcli/darnwiring.go:272` (verify-budget `Notify`)
- `internal/loomcli/wiring.go:267` and `wiring.go:995`
- `internal/loomcli/raddlewiring.go:246`
- `internal/loomcli/raddleverbs.go:248`
- `internal/beamcli/dobbydeps.go:186` (dobby run notices via `NotifyRunFrom`)

I did not read each site's text, so which of them carry a halt is unconfirmed. The fixer should check each and switch every halt-carrying one to `NotifyParentHalt` / `NotifyRunFromHalt`. Fix site: those call sites.

## Other checks
- **D5 and D7, `orch-role.md`:** its operator-word and go-ahead rules are at lines 58 and 429, both given the `role-autonomy.md` exception, along with the "Reviewing a pull request" section. Covered.
- **D8:** the direct-post text is in `role-autonomy.md` and the marker rule in `role-slack.md`. Covered.

## Tool calls
1. `grep NotifyParent( / NotifyRunFrom(` callers in `internal`, plus the `operator's word` lines in `orch-role.md`: the diff shows only the changed call sites, not the unchanged ones.
2. Read the context at the unchanged call sites above, to judge which are halt notices.
