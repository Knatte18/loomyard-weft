```yaml
lens: code-correctness
outcome: reported
```

The code-correctness lens over the diff's functions and branches: one finding, at LOW.

**Functions and branches, with the case run**

- `newShed`, empty `NotifyParentOn`: returns the recipe shed unchanged. Correct.
- `newShed`, when `loomrecipe.New` fails: returns the error before any wrap. Correct.
- `newShed`, when `watchRows` refuses: returns `nil` and the refusal. Correct.
- `newShed`, call paths: the step and run `BuildShed` and `prReviewSendBackEdge` all go through `newShed`, so every shed of a run's steps is wrapped. Correct.
- `watchRows`, unknown name `["Drift-Judge"]`: the validation loop runs before any wrap, so the shed is not partly mutated, and the refusal lists every row. Correct.
- `watchRows`, a duplicate name such as `["X","X"]`: wraps once. Correct.
- `watchRows`, wrap target: it replaces `shed.Producers[i].Producer` in place. `run.go:346` calls `def.Producer.Call` from `s.Producers` at call time, so the wrap takes effect. Correct.
- `Call`: `tell` swallows its own failures, then the inner `(outcome, ptr, err)` is returned untouched. Correct.
- `tell`, status file absent or unreadable: warns and sends nothing. Correct.
- `tell`, record absent: reads as the zero `RowNotice{Row:""}`, which differs from `{row, n}`, so the first visit notifies. Correct.
- `tell`, record read fails: warns and sends nothing. Correct.
- `tell`, record write fails: warns and sends nothing, as the doc comment says. Correct.
- `tell`, post fails: the visit is already written, so the notice is lost and not retried, as documented. Correct.
- `tell` across a Yield: `run.go:507` appends no history, so the count is equal and nothing is sent. Correct.
- `tell` across a pause or a cancelled call: no history entry is appended, and the resume write (step 3b, `run.go:323`) appends none either. Correct.
- `tell` on a re-entry by a bounce, an `OnDone` loop, a goto or a reroute: each appends an entry first, so the count grows and a notice goes out. Correct.
- `tell` on a re-call after an `Awaiting` outcome, or after a blocked Stuck halt, at the same row: one entry is appended (or one repeat folds in), so the count changes and a second notice goes out. Wrong against the stated contract; see the finding.
- `tell`, scratch record that outlives history: no code path truncates shed history, so a stale `(row, count)` cannot collide. Correct.
- `verdictCount`: a nil history gives 0, and one entry with `Repeats: 2` gives 3, so a fold adds 1. Correct.
- `ReadRowNotice` and `WriteRowNotice`: an absent file gives a zero value with a nil error, and errors are wrapped. Correct.
- `RowNoticeRecord`: the path is under `ScratchDir`, so the `path+".lock"` beside it is not a tracked-file lock. Correct.
- `newStrandRig` notifier: `NotifyAsker` takes the gate's worktree location (`resolveHub(request.Cwd)`), walks from a pair's recorded parent, signs as `GATE`, and returns the error. `newFixerStopper` also builds the rig, which is harmless. Correct.

**Finding**

- **Severity:** LOW
- **Class:** design
- **Location:** `internal/loomcli/rownotice.go:66-78` (`tell` and `verdictCount`)
- **Summary:** The visit key counts verdicts the row records about itself while the run stays at the row. A Bouncer escalation (`Awaiting`) appends an entry and keeps `current_producer` put, as do a budget-exempt Stuck halt at a row with no `OnStuck` and its fold. The next re-call of that row after the parent's answer or a resume therefore sends a second `[row reached]` mail, although the run never left the row. That contradicts "once each time the run reaches it" in `orch-role.md` and the `rowNoticeProducer` and package docs.
- **Evidence:** `internal/shedengine/run.go:470-481`: Awaiting calls `appendHistory`, and "current_producer stays put, so the next step re-calls the same producer". `run.go:323-339`: the resume write appends no entry. `run.go:386`: a blocked Stuck at a row with no `OnStuck` is budget-exempt and folds, incrementing `Repeats`. `internal/shedadapters/bouncer.go:1604`: Bouncer returns `shedengine.Awaiting`.

**Tool calls**

1. grep `run.go` for the step 3b resume write, the producer call site and the `buildLoomShed` callers: I needed to know whether a resume appends history and whether the wrap reaches the call site.
2. grep the loomcli shed-builder wiring and read `run.go:300-346`: I needed the `BuildShed` paths and the resume write's body.
3. grep for any code that truncates shed history: I needed to rule out a stale record colliding with a reset history.
