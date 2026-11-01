```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the adopt, fronttask, boardengine, dobbyengine and darn changes, from the inherited context only. I made no tool calls and ran each case by reading the code.

**List: function or branch, case run, verdict**

- `fixSlugPrefix` — a subject that just fits, one that doesn't fit, and a cut ending in `-`: correct. The cut keeps the hash and the `-t999` room, and `TrimRight` drops the trailing hyphen.
- `nextFixSlug` — taken is empty: `-t1`. A slug like `prefix-t1-t2`, `prefix-t01` or `prefix-tx` is ignored. The highest N plus one is correct.
- `failedEnd`:
  - Own fix task released, `running` or `paused`: files and removes nothing, correct.
  - Own fix task draft or halted: removed, then filed, correct.
  - Another front task open: files nothing, correct.
  - Tries at the limit with a halted own fix task: the notice names the task and removal is deferred, as D6 says. Correct.
  - Removal fails: files nothing, correct.
  - Gate label: only added when `Known` says so, correct.
  - Try count: `tries+1` is written between create and release, correct.
  - `File` fails at the record step: wrong notice, see F1.
- `judgeOwnFix` — empty status, a closed status, no run record, `paused`, `awaiting` with and without an escalation, and `done` with an entry that isn't done: all correct.
- `holdEntries`:
  - No entry for the slug: a new hold is added.
  - Owner entry: turned into a hold, keeping `PausedAt`.
  - Hold entry: kept.
  - Operator entry: kept and reported `paused`.
  - These cases are correct.
  - When a slug has several live entries, only the first match is looked at, so owner-then-operator ends up held. This is harmless, since green lifts only the hold and the operator entry stays. No finding.
- `holdTargets`, `holdSubject` — stopped or warned runs skipped, a live pair reported `running_here` and not held, a released front task skipped, an empty status held. Correct. I can't see where `adopt.go` orders the holds against the owner write.
- `appendHolds`, `removeHolds`, `readHolds`, `readTries`, `writeTries` — locked updates, an absent file reads as empty, correct.
- `greenEnd`, `liftHolds` — a missing file or an empty list lifts nothing. A hold already lifted by hand is skipped. A failed `Lift` returns a line, but the slug is still dropped from the holds file, which matches D5's "slugs it read". A failed `removeHolds` is reported. Correct.
- `startLiftedRuns` — no run record, a done run, an `ok:false` envelope and an error: all correct.
- `resumeHaltedChildren` — correct results, but it does needless work, see F3.
- `removeFixTask` — no front task, and a `RemoveError` step failing with the right hand step: correct.
- `fronttask.File` — an action outside the table, an open front task, and a failure at create, between, release or record. Correct on every path except the one in F1.
- `fronttask.Remove` — `ErrNothingLeft` counts as done, any other cancel failure stops before the record, the record drop and entry removal run under the lock. Correct. If the record write fails after `RemoveEntry` succeeded, the hand removal `lym board remove` fails because the entry is gone. The record stays, so `Settle` runs the action once. That is an edge case and I don't count it as a finding.
- `fronttask.Settle`:
  - Subject not owned: the project's records are dropped.
  - Open front entry with no record: it is recorded.
  - Entry gone or done: the action runs once and the record is dropped.
  - Entry halted or in a run status: left alone, so no action.
  - Action outside the table: dropped with a warning.
  - A failed action: the record is kept for retry.
  - All correct.
- `checkFrontPayload` — equal values dropped, differing values refused, a non-bool or non-string refused, project or kind change refused on a front entry. Correct. It mutates the caller's `fields` before a later payload in a batch can fail, which is harmless because the batch is atomic.
- `FileFrontTask` — correct for a fresh slug.
- `Store.Rename` — a front entry is refused, correct.
- Dobby `startReady`, `openFront`, `BuildView`, `planPass`, `drivePasses` — an open front task gives `candidates=[front]` and `pastMaxRuns`. A front task already running means nothing else starts. A held run gets no pass and no owner recheck. A front task takes no slot and the others get `FrontFirst`. Correct. I couldn't see where `RunView.Held` and `d.geom.Slug` are populated.
- `settleFronts` — an error is logged and posted once per episode, and a clean tick resets it. Correct.
- `darnWrite.Call` — both markers removed before the session, a stop marker alone gives the report plus `darnStopWayForward`, both markers give the report alone, a condition marker alone is no stop. Correct apart from F2.
- `runReports` — state precedence, an unknown held subject, a front fix task halted, and a `closedStatus` entry whose run record is gone. Correct.
- `startAction` — correct. It starts nothing and returns an error under `go test`.

**Findings**

- **F1 — LOW, design.** `internal/adoptcli/fixloop.go` `failedEnd` (the `fronttask.File` error return) with `internal/fronttask/fronttask.go` `File` (the record-write error). If the entry was created and released but the record write fails, `File` returns an error. `failedEnd` then says "the fix task … was not filed" and tells the operator to re-run adopt, but the task is released and will start. The try count was also already incremented. The notice should say the task was filed and only its record is missing, which `Settle` heals; or `File` should treat a failed record write as done and log it.
- **F2 — LOW, design.** `internal/loomshed/darnwrite.go`, the new `conditionStopped` branch (about lines 149-156). `pointer.Reason = report` can be empty, because the `darnStopWayForward` fallback that kept the reason non-empty is gone. An empty stop report would halt the run with no reason and no way forward. Fix: fall back to a fixed message when `report` is empty.
- **F3 — LOW, design.** `internal/adoptcli/green.go` `resumeHaltedChildren`. `ChildStatus` runs for every non-done board task before the `ParentOf` subject filter. A status read failure of a child under another subject produces a "way forward" line in this subject's notice, and every green end reads every pair. Fix: read `ParentOf` and skip other subjects before `ChildStatus`.

**Tool calls:** none.
