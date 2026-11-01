```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on the same state, and the acceptance criteria and constraints against the decisions they touch.

**Pairs checked:**
- D4 hold × D8 front exemption (re-run of adopt while the fix task is released and unstarted): held.
- D4 held runs filling `max_runs` × D8 front start past `max_runs`: held.
- D4 hold list in the check record × D5 check record writes (`runCheck` rewrites the whole record): broken (F1).
- D6 fixed fix slug × D8 front start hold × the done-slug start refusal: broken (F2).
- D6 "files none while a front task stands open" × D8 open definition × D9 machine-failure halt: broken (F3).
- D6 draft-fix removal before filing × D8 removal's first step, `lym batten cancel --remove`: broken (F4).
- D5 green-end removal × D8 removal steps, a draft fix left by a hand `cancel --remove`: broken (F4).
- D5 green end × D8 open front task that is released or running at the green, against D5's acceptance criterion: broken (F5).
- D9 stop way forward × D6 "never `lym loom resume`" × D7 `halted` way forward: broken (F6).
- D8 healing record × D1/D3 ownership move (dobby on the old machine): broken (F7).
- D7 `waiting_for_check` × D4 held released tasks that have no run record: broken (F8).
- D7 `halted` × run-record states: broken (F9).
- D5 green removal × D8 "action runs on gone" (record dropped before the entry is removed): held.
- D5 try-count reset × D6 bound "between two hand adopts": held.
- D4 owner-write failure × D5 re-run starting a check on standing holds: held, F1 aside.
- D2 warning on an unmoved branch × D4 holds written: held.
- D7 precedence × D4 for a held run that also reads halted: held.
- `PATTERN-dobby-starts-runs` × D5 green first passes for lifted moved runs: held, since they are moved runs and not started from an empty status.

**Findings**

F1 — BLOCKING, design, D4 × D5 (`decision-record.md` D4 "Adopt records each held slug in its subject's check record", D5 Bound).
`runCheck` starts with a fresh `CheckRecord` written through `state.WriteJSON` and rewrites the whole record at each tier and at its end.
Evidence: `internal/adoptcli/check.go:144-145`, `:163`, `:171`, `:296-297`.
- The held slugs adopt writes before it spawns the check are wiped when the check starts.
- A second adopt's holds, written while a check runs, are overwritten at the next tier write.
- The green end then lifts nothing, so the held runs stay held for good.

The record never says how the hold list survives the check's whole-record writes, or whether it lives elsewhere.

F2 — BLOCKING, design, D6 × D8 (D6 slug rule, D8 "dobby starts no other task there while it is open").
Every try reuses the slug `adopt-fix-<subject>`.
- After try 1 lands, the slug's run status reads done.
- Dobby's TaskView then carries `StartRefusal` from `doneSlugRefusal`, which says run `lym batten cleanup --apply` or use a different slug. Evidence: `internal/battencli/dobbyreads.go:292-293`, `internal/battencli/arm.go:865-869`.
- The re-filed fix task therefore never starts. As an open front task with an empty status, D8 still blocks every other start in the worktree.

The loop dies at try 2, contradicting D3 and D6 ("up to a configured number of tries").

F3 — BLOCKING, design, D6 × D8 × D9 (D6 "While any front task stands open … the check files none"; D6 machine-failure path).
- A fix task halted on a machine failure has a board status of the form `blocked · <row>`. That is a run status, so D8 reads the task open. Evidence: `internal/shedengine/status.go:21`, `internal/boardengine/runlock.go:17-24`.
- The operator re-runs `lym adopt <subject>` as D6 says. If that check is still red, for instance because the machine was only partly settled or a code failure now shows, the check files nothing, and its notice says to re-run once the halted task lands.
- The halted task never lands: its way forward is the re-run of adopt, never `lym loom resume`.

The subject is left with every run held, a front task that blocks all starts, and no fix task.
D6 removes only a draft fix task of the subject before filing, never a halted one.

F4 — MEDIUM, design, D5/D6 × D8 removal (D8 "removes a front task in three steps: `lym batten cancel <slug> --remove` …"; D5 failed-removal notice).
`lym batten cancel` refuses when nothing of the slug is left. Evidence: `internal/battencli/cancel.go:554-598`, `internal/battencli/cancelcmd.go` help.
- A draft fix task left by a hand `cancel --remove` has no claim, records or pair, so fronttask's first step always fails.
- The same holds for a released fix task that never started.
- With the first step failed, the entry is never removed, and D6's "removed before the filing" leaves the next filing meeting an existing entry at the same slug.
- D5's notice names `lym batten cancel <slug> --remove`, which refuses again; only `lym board remove` works.

F5 — MEDIUM, consistency, D5 decision text against D5's acceptance criterion ("after the green end no fix-task entry of the subject remains and dobby starts the subject's released tasks").
- D5 removes the fix task only when it is halted or draft.
- A re-run of adopt while the fix task stands released or running starts a new check. When that check goes green, the fix entry remains, and under D8 dobby starts no released task until the fix lands.
- That green run also needs `cancel`, which refuses on an unstarted fix (F4).

The criterion is false under D5 and D8 together.

F6 — MEDIUM, design, D9 × D6 × D7 (D6 "as its way forward, a re-run of `lym adopt <subject>` … never `lym loom resume`"; D9 "as for any darn stop"; D7 `halted` "naming the way forward its stop report gives").
The darn producer appends `darnStopWayForward` to every stop report's reason: `lym loom decision add`, then `lym loom resume`. Evidence: `internal/loomshed/darnwrite.go:30`, `:146`.
The halt reason of a machine-failure stop therefore names `lym loom resume` beside the adopt re-run, which contradicts D6. D9 does not change that suffix.

F7 — BLOCKING, design, D8 healing record × D1/D3 ownership move (D8 "dobby records a front entry it sees open on the board with no record … runs the action once … when a recorded task's entry reads done or is gone").
- The old machine's dobby for the same worktree keeps ticking after it loses ownership. Its owner pause applies only to runs (`internal/dobbyengine/owned.go`).
- It sees the fix task filed by the new owner open on the shared board and records it.
- Once the fix task lands, it spawns `lym adopt check <subject>` on the machine that no longer owns the subject.
- A red check there files a new fix task on the shared board, which the owner's dobby then starts.

No decision gates recording or running `on_land` on this machine owning the subject.

F8 — MEDIUM, design, D7 × D4 (D7 `runs` "one item per run under the named subjects", `waiting_for_check` "a held run", the row taken from the batten run record; D4's second bullet).
- D4 holds released tasks that have no run record.
- `movedRuns` lists only entries with a run record that is not done (`internal/adoptcli/adopt.go:402-406`, `doc.go`).
- D7 does not define its population, so a held released task's line, state and row are undefined. The D4 criterion "a released task … gets a hold" has no matching D7 report.

The same gap covers the unstarted fix task that D7 counts as `running_here`.

F9 — LOW, consistency, D7 `halted` ("whose batten run record reads halted").
No run-record state is called `halted`: the shed states are `running`, `paused`, `done`, `blocked`, `failed` and `awaiting` (`internal/shedengine/status.go:18-23`).
The rule does not say which of these read as `halted`.

Outside my lens: D8's "one `boardengine` write of a released task" against the draft rule in `internal/boardengine/draft.go` (`applyDraftRule`) is a claim about existing code, for the bound-breaker or existing-paths lens.

Tool calls:
1. Grepped `internal/adoptcli/check.go` for how `runCheck` writes its record: the manifest had `doc.go`'s description only, and F1 needed the write pattern.
