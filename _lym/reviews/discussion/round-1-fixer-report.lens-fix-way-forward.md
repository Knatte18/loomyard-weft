```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every next step, refusal and way forward the fix added to `_lym/discussion/decision-record.md`.

**Ways forward checked**
- D4 step 2, failed new-parent check: works. The slug pause from step 1 is lifted with `lym tender resume <slug>`, then the orch tells the operator.
- D4 step 3, partial finish leading to a second cancel: works. A slug-scope pause does not block the cancel, which refuses only a hub pause (`internal/battencli/cancel.go:299-304`).
- D4 step 3, refusal leading to D7, which lifts the pause with `lym tender resume <slug>`: works. A refusal comes before `LiftPauses` (`cancel.go:572`), so the slug pause is still live for the resume to lift.
- D4 step 3, remote-teardown failure notice: works. The notice tells the orch to push the tag with `git -C <dir> push origin refs/tags/<tag>` (`remoteteardown.go:184-190`). `<dir>` is the location the cancel command builds for the discard (`cancelcmd.go:287`), so the notice names a directory that still exists.
- D4 step 3, `ls-remote` confirm: two gaps, below.
- D4 step 5, prune then release, or handover to another orch through the operator: works.
- D3, holding a run with no live session: works. `lym tender pause` reports a pair with no reed session as skipped and keeps the pause.

**Findings**

1. MEDIUM. D4 step 3: "Before step 4, the orch confirms that tag on origin with `git -C <prime> ls-remote …`. On a remote-teardown failure notice for the slug, it first takes the way forward the notice names."
   - Summary: the step gives no way forward when the tag is not on origin yet and no notice has come. The cancel returns while the detached teardown child is still pushing, so the first `ls-remote` normally comes back empty. The orch cannot tell whether the push is still running or will never happen, and it has no rule to wait, retry or stop.
   - Evidence: `internal/fabricengine/remoteteardown.go:319-333`. `childHold.finish` hands the job over and "returns at once". The `DiscardTaskBranch` doc says the lock "is held until the remote half ends" (`taskbranchdiscard.go:37`).

2. MEDIUM. D4 step 3: "The code archive tag is `archive/<slug>/<first 12 hex of the tip>`, named from the tip recorded in step 1".
   - Summary: when origin's task branch tip is strictly ahead of the worktree's local tip, the child pushes only the tag for origin's tip. The tag named from the recorded local tip then never reaches origin, and no failure notice is sent. The `ls-remote` confirm can never pass, and the procedure has no exit from that state.
   - Evidence: `internal/fabricengine/taskbranchdiscard.go:105-118`. In `archiveDiscardedTips`, the branch where the local tip is contained in origin's archives only `originTip`. The local tag is created locally (`:79-84`) but nothing pushes it.

Further tool calls:
- grep of `discardRerunWayForward` and the `DiscardTaskBranch` call sites. My context lacked which location the cancel passes to the discard, which decides whether the failure notice names a directory that still exists.
- grep of `cancel.go` for draft and no-run handling. My context lacked whether re-running the cancel after it completes, as the notice's way forward says, is refused. The grep did not settle it, so I raised no finding.
