```yaml
lens: fix-proposal
outcome: reported
```

Findings:

- **MEDIUM**: changed text in D5's dobby-tick bullet, "a pair is ahead when its local records branch or its local code branch is", and the matching D5 acceptance criterion.
  Summary: the proposal fires on every mid-row code commit, so the tick also pushes work in progress.
  The narrower restart F2 needs is a missing transition code tip.
  Evidence: `PushPairAnchored` pushes all of a pair's unpushed code commits once the records side lands. It does not push only up to the records tip's `Warp-SHA`, and today code commits without a transition "go with the next pair push" (`internal/fabricengine/pushanchored.go:106-121`).
  Concrete case:
  - An agent commits code during a row.
  - Its records branch stays level until the row's transition, and the code branch reads ahead.
  - Each dobby tick then starts that pair's pusher, which first runs D7's reconcile (a fetch) and then pushes the work-in-progress commits.
  - Before this change, those commits reached origin only with a transition, through the seam's pusher (D3) or a resume.
  So the change adds a fetch and a push per pair per tick while an agent works. It also puts on origin code commits that no transition names, which an adopter's re-run from origin's last transition then finds past that transition's `Warp-SHA`. The fix does not account for either effect in D2, D3's rationale or D6.
  A narrower restart closes F2's gap without this: count a pair ahead when its records tip's `Warp-SHA` is not on origin's code branch and its local code branch holds it. That is exactly the owner-machine state D6 now leaves out of `CodeTipMissing`.

- No finding on F1's proposal. The pause `PausePair` writes is the hub pause-record entry that `pauserecord.ReadCoverage` reads, so D3's gate and D7's stop now read the same record. `PushPairAnchored` also passes the process pause gate before it takes a lock (`pushanchored.go:137-138`), so the pusher's own push agrees with the gate.

Further tool calls:
- grep of `internal/fabricengine/pushanchored.go` comments, to check which code commits `PushPairAnchored` pushes. My context held no part of that file.
