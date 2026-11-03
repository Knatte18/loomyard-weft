```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for stale twins and sibling disagreement (lens fix-consistency).

Findings:

1. **LOW** — `docs/overview.md`, the shuttle entry: "a turn end with outstanding work is waiting, not held, unless its only outstanding work is non-awaited shells, which a gated run judges as a plain turn end and an ungated run holds once they pass the shell wait bound."
   - The F5–F7 fixes reworded the same claim in `waitmark.go`, `spec.go` and `runlevel.go`, but this twin was left as it was.
   - The claim holds only for a gated run with an output file missing. With every output file present, a gated run now gets the nudge and Done at the bound.
   - A reader of the overview would notice the disagreement with `wait.go` and the reworded comments.

2. **LOW** — `internal/shuttleengine/completionsignal_enforcement_test.go:189-190`: "its third judges a gated waiting turn end on non-awaited shells as a plain turn end, Done only after consulting the files."
   - This is the same stale premise as F6 and F7.
   - In `wait.go`, the gated shell-only branch is now a hold (files missing) or a nudge plus `expiredTurnEnd` (files present). It also has a new early return for a turn start.
   - Nothing in the sentence is wrong about the files check, but "as a plain turn end" no longer describes the files-present path.

Checked, no disagreement:
- `shuttleengine/doc.go:429` and `claudeengine/doc.go:312` are both qualified "with an output missing".
- The `case.yaml:12` wording was rejected by the review.
- `nudgedShells` and `shellNudge` are used only in `wait.go` and `run.go`.
- The `wait.go` file-header lines 18-22 and the `expiredTurnEnd` doc still match the new behavior. The new clear-on-turn-start is covered by the added `pollEventsTick` doc sentence.

Tool calls:
1. `grep` for "plain turn end" and the nudge symbols across Go and Markdown, to find twins outside the files I had read.
2. `sed` of `completionsignal_enforcement_test.go:185-192` plus a `git diff --stat` on docs, to see the context of the hit in `completionsignal_enforcement_test.go` and check whether `overview.md` was touched on this branch.
