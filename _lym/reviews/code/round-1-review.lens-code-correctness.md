```yaml
lens: code-correctness
outcome: reported
```

I reviewed the diff through the code-correctness lens: each changed function, branch and error path, run on one concrete input.

**Per function or branch: the case I ran, and whether it is correct**

- `gatefixer.Await` with the reordered classify:
  - Gone strand, run record `stopped` and `interrupted`, past the timeout → `interrupted`. Correct.
  - Gone strand with no readable record → `died`. Correct.
  - Live strand past the timeout → `timeout`. Correct.
  - `StrandLive` keeps failing past the timeout → an error each poll, which `rowWrap.wait` turns into stop plus `died`. Before the diff that case was `timeout`, and before the timeout it already errored. The change is harmless.
- `goneStrandReason`:
  - `FindRun` fails → `died`. Correct, as its doc states.
  - The interrupted run dir was removed by the orphan sweep before the read → `died`. **Finding 2.**
- `rowStore.budget`:
  - Records `[done, dead/interrupted, done]` → 2 spent and 1 interrupted. Correct.
  - A live record counts as spent. Correct.
- `rowWrap.spawning`:
  - Records `[interrupted r1]` → spent 0, n=1, so `report-1` and `reply-1` are reused. Those files belong to a terminal record only. Correct.
- `waitThenRun(replace=true)`:
  - The chain ends on the replacement, and `markTerminal` lands on the replacement's guid. Correct.
- `waitThenRun(replace=false)`, the case a restarted driver hits:
  - A left-live interrupted record met through expiry and then `Final` is never replaced. **Finding 1.**
- `waitReplacingInterrupted`:
  - attempts=0 with two interruptions → the first is replaced, the second stands (`2 &gt; 1`). The bound matches its doc.
  - At the cap, the note it leads to is wrong. **Finding 3.**
  - `rf.Spec` fails → the record was already marked terminal and the error is returned. A later evaluation re-runs inner fresh. Consistent.
  - `SpawnOrAttach` fails → the error is swallowed. **Finding 4.**
- `fixerOutcome.note`, interrupted at the cap → claims no replacement ran. **Finding 3.**
- `Bouncer.judgeCall`, died branch:
  - `NotStarted` → `ErrNotStarted`, which `shedtransient` classes as `TransientAgentStart`. Correct.
  - Started and then died → `ErrAgentDied`, a producer failure with no bounce spent. Correct.
  - Cancelled ctx → the cancel error comes first. Correct.
  - A verdict on disk → the harvest settles first. Correct.
- `Bouncer.judgeCall`, `runErr` branch:
  - A start error that is not the not-ready kind, such as an EAGAIN spawn refusal, still degrades to `Stuck` and spends bounce budget. **Finding 5.**
- `judgeDiedError`: all three switch arms checked. The `NotStarted` text repeats the sentinel's wording; that is cosmetic only.
- `boxCapture`:
  - Styled reading shows a box → returned. Correct.
  - Styled shows none, plain capture fails → the styled reading is kept. Correct.
  - Styled shows none, plain shows none → styled is kept and held. Correct.
  - Styled shows none, plain shows a box → plain is used and a warning logged. This happens on every capture while the disagreement lasts. **Finding 6.**
  - An engine without `InputBoxReader` → unchanged.
- `holdsOnlyAPIErrorPrompt`:
  - An empty interior → false. Correct.
  - The prompt wrapped over two lines with `│` bars → true. Correct.
  - "…try again later" → false. Correct.
  - The prompt after the caret in the plain capture → true. Correct.
- `IdleSession` and `InputBoxText` read a box holding only the prompt as empty, in both the styled-turned-plain and the plain capture. Correct.
- `planPaneOOMNotices`:
  - Two strands in one pane scope → one notice. Correct.
  - The first strand's read errs but returns a cgroup (`memory.events` unreadable) → the second strand is skipped. That cgroup's file would fail for it too, so nothing is lost. Correct.
  - The read errs with an empty cgroup → no mark, and the next strand is read. Correct.
  - Strands reordered across ticks → `told` stays per cgroup, so no double notice. Correct.

**Findings**

1. **MEDIUM, design.** `internal/gatefixer/rowgate.go:126,159` (`Final` and `waitOnly`, both with `replace=false`).
   - **Summary:** in the board's own scenario, a driver restarted onto a left-live fixer record never replaces the interrupted fixer.
   - **How:** the new wrap's `spawned` set lacks the record's strand, so `expired()` reports true (rowgate.go:322–330). The poll loop then runs `probeExpiry` while no boundary has been seen since the attach (shuttleengine/wait.go:410–420). That finalizes with `evaluateGate(true)`, which runs the entry's `Final` closure (wait.go:1887–1888, 2019). `Final` waits with `replace=false`: it marks the record terminal as interrupted, and a failing verdict carries the note that no replacement ran.
   - An earlier non-fixer entry that fails over the record does the same.
   - The `LeftLiveByARestartedDriver` row calls `entry.Gate()` directly, so it never takes this path.
2. **MEDIUM, design.** `internal/gatefixer/await.go:118–126` (`goneStrandReason`) and the crash-resume premise in rowgate.go:264.
   - **Summary:** the interrupted fixer's run dir can be gone before `Await` reads it.
   - **How:** the reconcile stop removes the strand from reed state (`stopRecorded` ends in `removeStrandIfPresent`, stop.go). Any later shuttle `Start` in the worktree runs `sweepOrphansOpportunistic` (run.go:657). Once the dir is older than twice the startup timeout, which a two-day stop guarantees, the sweep deletes every run dir whose guid is not in reed state (rundir.go:413–414).
   - **Effect:** the fixer then reads `died`, not `interrupted`. It spends budget, is not replaced, and the replacement could not resume the session anyway, since its predecessor record is gone.
3. **LOW, design.** `internal/gatefixer/rowgate.go:49` together with rowgate.go:280–281.
   - **Summary:** the interrupted note says "…before it replied, and no replacement ran", but at the cap replacements did run. The `AnInterruptionPastTheCapStandsWithItsNote` row has 2 spawns, and the writer is told the opposite.
4. **LOW, design.** `internal/gatefixer/rowgate.go:287–290`.
   - **Summary:** a failed replacement spawn returns `(outcome, rec, nil)` and drops the error. Nothing logs it, the findings omit its cause, and the transient mark `SpawnOrAttach` adds (spawn.go:116) is lost.
   - The first spawn, by contrast, puts its error in the findings (rowgate.go:215–218).
5. **MEDIUM, design.** `internal/shedadapters/bouncer.go:2226–2227`.
   - **Summary:** a judge whose start failed in a way other than not-ready still degrades to `Stuck` and spends bounce budget.
   - **How:** `RunGated` returns start errors that do not wrap `ErrNotStarted` unchanged (shuttleengine/run.go:1263–1269). That includes a spawn the host refused for EAGAIN, which `shedtransient.Class` would class as transient (shedtransient.go:32). `degrade` swallows the error before that class can apply.
6. **LOW, design.** `internal/shuttleengine/readiness.go`, the `logger.Warn` in `boxCapture`.
   - **Summary:** while the styled and plain captures keep disagreeing, which is the case this fallback targets, the warning fires on every `boxCapture` call. That means each poll tick of the send-ready wait, each idle probe and each interrupt read, each with a 12-line tail. Nothing limits it to once per strand.

**Tool calls**

1. `wait.go:1845–1895` and the `finalize` grep: to confirm that the expiry-driven finalize runs `Final`, which the manifest did not show.
2. `rundir.go:400–450`, `run.go:1290–1320` and the sweep/resume call sites: to see whether the orphan sweep spares a stopped, interrupted run dir. Only part of `sweepOrphans` was in context.
3. The `paneKills` wiring grep and `ProcessOOMKills`: to learn the kill reader's return shape on error.
4. The `ProcessOOMKills` definition: it forwards to the platform function, which was not in context.
5. The `processOOMKillsPlatform` return lines: to see whether a cgroup comes back alongside an error.
