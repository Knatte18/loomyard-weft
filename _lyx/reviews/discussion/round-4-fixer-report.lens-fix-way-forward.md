```yaml
lens: fix-way-forward
outcome: reported
```

Task: check whether each refusal, error and next step the F2 fix added can be reached and leads to progress (fix-way-forward lens).

- MEDIUM. Changed text: D4, `_lyx/discussion/decision-record.md:119`, "A claim `claimBoardEntry` loses inside the Worktree-Create row stops that row like any row failure, and the run's next full pass reaches `refuseStartOwnedElsewhere` at the found Worktree-Create status and exits with the code."
  Summary: dobby never runs that next full pass, so the code never reaches dobby on this path. The text reads as if the owned-elsewhere handling applies here, but it does not.
  Evidence: a row failure writes `StateFailed` (`internal/shedengine/run.go:479`, `entry.go:85`). Under D4, dobby passes a run "only while batten's own state is `running`" (decision-record.md:158), and "A halted or paused batten run is resumed by a hand pass … dobby never resumes one" (decision-record.md:160). A retry the tackler or arbiter makes inside the failed pass re-runs the row, and so calls `claimBoardEntry` again, without running `ownerPreflight`. So `refuseStartOwnedElsewhere` is reached only by a hand pass, and the next sentence, "Dobby reads that code from a run it drives", does not cover a hand pass. On this path the run gets no owner pause and no owner notice. The way forward that exists runs differently: the failed row's stop carries `ownedElsewhereRefusal`'s text naming `lyx adopt &lt;slug&gt;` and goes through D7's shed chain to the orch, and a hand pass then exits with the code and the same text. The sentence should name that route, the stop through D7's chain and a hand pass exiting with the code, and not imply that dobby reaches it.

No other findings. The fresh-start sentence (a first pass with no status file, refused, "stays a refused start under D5") leads to progress through D5's backoff and its retry on an entry change, which an adopt causes. The acceptance bullet's wording is reachable for the Worktree-Create case only as far as the D4 sentence above allows.

Tool calls:
- One grep of `internal/shedengine` state writes and the decision record's halted-run lines. I needed to know what state a failed row leaves the run in, and whether dobby passes that state. My context showed only `claimBoardEntry`'s call site in `wire.go`.
