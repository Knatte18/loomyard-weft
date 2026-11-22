---
round: 1
focus:
  - "internal/loomcli/prcontinue.go and internal/landingshed/prreview.go: does PR-Review end only on a satisfied answer at the gated head (D4), checked in both pr-continue and the row's judge, with the recorded autonomy used rather than a fresh read?"
  - "internal/driftengine/verify.go and judgeparent.go: does an ended judge ask (timeout, not-sent, no-parent) halt Drift-Verify only under autonomy, and does the resume clear it and send a new ask (D3)? Check the failure paths."
  - "internal/hubgeom Autonomy and internal/dobbyengine LookupAutonomy: is the lookup start (run's own worktree vs orch's own) right, does a stack that fails to load read false with a warning, and is a set value distinguished from a merged one (D2, D11)?"
  - "internal/loomengine and internal/driftengine ApplyAutonomy plus AutonomyForcedSetting: are all five forced keys covered, including the raddle_auto_approve setting-lookup path, and is pr_review_parent_timeout_min classified and validated (D3, PATTERN-config-layers)?"
  - "internal/pawlengine/beamawait.go: do the drain conditions, the drain marker's single writer and its removal on a reopened member match D6, and does autonomy false post nothing?"
  - "Finalize (finalize.go, finalize_pr.go): does the config-change notice leave autonomy out, and does a closed pull request with recorded autonomy true halt whatever ReviewEnded says (D1, D4)?"
  - "internal/orchcli notify.go and slackcli hook.go: does every halt notice carry the run's autonomy, and does the Slack hook filter only under autonomy and a slack presence (D2, D8)?"
  - "Refusal ways forward (PATTERN-refusal-way-forward): does each new refusal in pr-continue, parent-review ask and the answer verb name its way forward, and does a test reach it?"
  - "Test economy: check the contained candidates TestWithLaunchFacts and TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped against their covering tests' assertions before filing anything."
---

Focus covers the highest-risk decisions of the record: PR-Review ending, the judge ask halt, the autonomy reader and its forcing, Beam-Await's drain, and the notice and hook edges.
