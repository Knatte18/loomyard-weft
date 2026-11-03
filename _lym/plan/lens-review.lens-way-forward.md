```yaml
lens: way-forward
outcome: reported
```

Way-forward review of plan cards 1–14: for each refusal, stop and recovery step, I checked whether the next step it names can succeed from the state it leaves.

Findings:

1. **LOW** — Card 5, `DiscardTaskBranch`'s refusal when the teardown-lock wait runs out. It names "re-run the discard once it finishes", but no operator command is called "the discard". The discard is reached from `lym batten cancel <slug> --remove` (`internal/battencli#removePairAndBranch`) and from `lym fabric cleanup`, so the text should name the command that reached it, e.g. a `rerun` the caller passes in, as card 5 already does for `lockPairTeardown`. Evidence: card 5 Intent, third bullet; `internal/battencli/cancel.go:633`.

2. **LOW** — Card 8, the skipped-slug reason. It names "re-run lym fabric cleanup --apply". `Topology.Cleanup` may also be reached from `lym cleanup --apply`'s hub reconcile; if so, the fixed text sends that operator to a different verb. Evidence: card 8 Intent, first bullet; I did not trace the callers of `Topology.Cleanup`.

3. **LOW** — Card 12, the `failReadopt` notice. `battenshed.HandBackNotice` skips the post when the `handback <historyLen>` marker is already set. `WriteFailedStop` does not append to the history. A run that was handed back earlier at the same history length therefore gets no notice for the failed stop, which breaks D2's "posts one notice" and leaves the parent with no way forward. Fix: post with a key of its own, or check the marker first. Evidence: `internal/battenshed/rownotice.go:144-147`; `internal/shedengine/failedstop.go:85-91`.

4. **LOW** — Card 12, the status commit. `c.shedPaths.CommitStatus(producer, "failed")` goes through `newCommitStatusSeam`, which skips the commit and push when its marker already holds the same (producer, state). Example: an earlier failed stop at Land, a resume that never committed a new state, then a permanent readopt answer. The new error and way forward then stay uncommitted and unpushed, so another machine reading origin sees the old error. Evidence: `internal/battencli/commitstatus.go:135-142`.

5. **LOW** — Card 12, the Land way forward on `*ErrRemoteLeftover`. It names `git -C <records repo> tag archive/<slug>/<12hex> <tip>`. That fails with "already exists" when a tag of that name already sits locally at `<tip>`, for example after an earlier archive of the same tip. The steps then cannot be run verbatim. Fix: make the step conditional ("unless the tag exists"), or reuse the existing tag the way `tagAndPushArchiveTip` does. Evidence: card 12 Intent, `NameLand` / `*ErrRemoteLeftover` bullet; `internal/fabricengine/archive.go:114-125`.

6. **LOW** — Card 12, `landRestoreWayForward` when the task branch is gone too. The parent for `git -C <prime> branch <task> <parent>` comes from "the chosen tag's origin record", or from `BranchFile` when the weft is live. When the weft is not live and no `done` candidate exists, neither source exists. The card does not say whether the task-branch step is still named, or with which parent, so a row could name `<parent>` empty. Evidence: card 12 Intent, `landRestoreWayForward` bullets 1–2.

7. **LOW** — Card 12, steps that run raw git. The Land way forwards name raw `git ... push` steps (tag push, leased `--delete`). These go to the run's parent, an agent session. Under `PATTERN-refusal-way-forward` ("no refusal ... names a step agents' settings deny"), the shell guard row `SwitchDenyPublish` refuses `git push` in any session whose launch spec turns it on. The card does not show that the receiving orch session runs without that switch. Evidence: `internal/shellguard/rows.go:22,147-155`.

Checked, and progresses: the run-scope reconcile at Land with the pair absent is not refused as ended, because `withoutFinalizeEntered` keeps a run whose prime batten status names Land. So card 12's Land restore and card 13's Land warning can reach `Add` (`internal/fabricengine/replay.go:262-275`, `:307-320`).

Tool calls:
- Read `withoutFinalizeEntered` (replay.go): needed to know whether a gone pair at Land reads ended before `Add`.
- Read `finalizeEnteredRead` and `HandBackNotice`: needed the absent-pair branch and the notice's dedup marker.
- Grepped `childLanded` / `settleBoardClaim` (pairteardown): needed whether a removal at Land marks the board entry done.
- Grepped contracts and shellguard for git push deny rules: needed to know whether the named raw git steps are denied to agents.
- Listed `.claude/settings.json` and `internal/shellguard`: to locate the deny rules.
- Grepped `.claude/settings.json` and shellguard `rows.go` for push: to read the publish-deny row.
