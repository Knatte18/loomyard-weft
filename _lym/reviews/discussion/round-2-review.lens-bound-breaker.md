```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each bound, guarantee and code claim in `decision-record.md` and tried to break it.

**Claims attacked**
1. D6 bound, "at most `adopt_fix_tries` fix tasks per subject … at most one at a time", read as a promise that tries 2 and later run: **broken** (F1).
2. D6/D8, a failed check with an open front task files none, and the notice says to re-run once it lands: **broken** for a halted fix task (F2).
3. D8 removal order, "cancel --remove … then record drop, then entry removal; a step that fails leaves the steps before it done": **broken** when cancel refuses with "nothing left" (F3).
4. D8, "one `boardengine` write of a released task": **broken** by the draft rule (F4).
5. D8, "the record outlives a dobby restart … dobby records a front entry it sees open with no record": the restart part **holds**; the healing rule is **broken** across machines (F5).
6. D8 bound, "a board edit cannot make dobby run an arbitrary command": **holds** for the command. A board edit can still change which subject's check runs and which worktree is blocked, through a `project` edit or `lym board rename` (F6).
7. D8, `front` and `on_land` refused on upsert, upsert-batch and merge "as `checkBeamPayload` does": **holds**. All three Store paths share one payload check (`store.go:507,842,1021`). Rename is a write path outside this claim (F6).
8. D8, the run-agent task guard skips `fronttask`'s write because it carries no strand name: **holds**. Only `boardcli` sets `StrandName` (`boardcli/cli.go:161`; `taskguard.go:12-16`).
9. D6 slug rule, one fix slug per subject within `MaxSlugLength`=32: **holds**. 10 + 15 + 1 + 6 = 32. A 22-character subject ending in `-<6 hex>` matching a long subject's hash is the only collision, and it is negligible.
10. D4/D8, a front task starts past `max_runs` with held runs listed: **holds** as a stated requirement. `startReady` breaks on `running >= MaxRuns` before reaching any task (`starts.go:80-83`), so the plan must order the bypass ahead of that break. The record states the requirement, so this is no defect.
11. D4 and the declared `PATTERN-adopt-hold`, "only its subject's green adopt check lifts it … `lym tender resume` by hand": **broken** (F7).
12. D6/D9, the machine-failure stop names re-running `lym adopt <subject>`, "never `lym loom resume`": **broken** (F8).
13. D7, `halted` "whose batten run record reads halted": **broken** as a code claim (F9).
14. D7, `running_here` is "any other run this machine owns": **broken** by a run cancelled on the old machine (F10).
15. D4, "dobby's start already waits on a pause covering the task's slug": **holds** (`starts.go:87-101`, `heldByEntry`).
16. D7, "Land's `PruneBoard` … removes every done entry": **holds** (`land.go:103-110`, `board.go:392`).
17. D8, "Cancel alone writes the draft status over the run status … keeps the pair unless `--remove`": **holds** (`cancelcmd.go:491-503`, cancel help text).
18. D5, "a start refused for a held driver lock is no error": **holds** as a stated rule.

**Findings**

**F1 — BLOCKING, design — D6 slug rule with the fix loop**
- Every try reuses `adopt-fix-<subject>`, but a landed run keeps its run record with state `done`.
- For a released task whose run status reads done, `dobbyreads.go` sets `StartRefusal = doneSlugRefusal(...)` (`internal/battencli/dobbyreads.go:288-293`), and dobby never starts it (`starts.go:96-99`, `heldByEntry`).
- The refusal asks for `lym batten cleanup --apply` and deletion of the task branch (`arm.go:865-869`). Nothing in Land removes the run directory: grepping `battenshed/land.go` and `battenshed/teardown.go` finds no RunDir removal.
- Counterexample: try 1 lands, the check stays red, try 2 files the same slug, and dobby holds it with a start refusal forever. The loop's tries 2 to N never run.
- Fix: the slug must vary per try, or the filing must clear the done run record and the leftover branch first.

**F2 — BLOCKING, design — D6 with D8 for a halted fix task**
- A halted fix task's board status is a run status, `blocked · <row>` or `failed · <row>`. The shed states are `blocked` and `failed` (`shedengine/status.go:18-23`), and the board composes `RunStatus(state, producer)` (`runlock.go:16-24`).
- D8 therefore counts the halted task as open, and dobby keeps starting no other task in the worktree.
- Counterexample: the fix halts on a machine failure. The operator settles the machine and re-runs `lym adopt <subject>`, and the check is still red, for example because the failure is now a code failure.
- D6 then files nothing, since an open front task stands, and its notice says to re-run once that task lands. The halted task never lands, because its way forward is that same re-run.
- The subject stays held and blocked with no path the record names.
- Fix: the filing replaces the subject's own halted fix task, as it does a draft one.

**F3 — BLOCKING, design — D8 removal step 1 and D6's draft removal**
- `fronttask`'s removal starts with `lym batten cancel <slug> --remove`.
- Cancel refuses "nothing of <slug> is left to cancel" when no record, claim, pair part, session, process or marker is left (`cancel.go:554-598`). A draft or empty status is no claim (`cancelcmd.go:496-497`).
- Counterexample 1: the operator follows D5's own way forward, `lym batten cancel <fix> --remove`, which leaves a draft entry with no pair and no records. On the next red check, D6 tries to remove that draft fix "before the filing". Cancel refuses, the removal stops at step 1, and the record says neither whether the filing goes on nor how the slug clash with the standing draft entry resolves.
- Counterexample 2: at a green end, a released fix task that never started has nothing to cancel either.
- Fix: removal treats "nothing left" as step 1 done.

**F4 — MEDIUM, design — D8 "one `boardengine` write of a released task"**
- The write critical section gives every new task `draft` (`applyDraftRule`, `draft.go:37-60`). Per `doc.go:245-249`, a create is draft "even when the payload sets status to null"; release takes a second write that sets a null status.
- So filing takes two writes, or a draft-rule exemption the record never states.
- A death between the two writes leaves a draft front task. D8 calls that closed, and dobby's heal only records open entries, so nothing heals it until the next red check.
- The record's crash-heal claim ("a filing that dies between the two writes is healed") covers the record write, not this gap.

**F5 — BLOCKING, design — D8 land action and dobby's heal across machines**
- Dobby "records a front entry it sees open on the board with no record" and runs `on_land` when the entry reads done or gone, with no owner condition.
- The board is shared, and the old machine's dobby keeps ticking for a worktree now owned elsewhere: `owned.go` only pauses such runs.
- Counterexample: machine B adopts beam X, and its check files `adopt-fix-X`. Machine A's dobby for X sees the entry open with no record and records it. The fix lands on B, and Land prunes the entry.
- A's dobby then reads the entry gone and spawns `lym adopt check X` on A, the machine that does not own X.
- If that check is red, A files a new released fix task, which B's dobby starts, against a failure on A's machine. A's try count is separate from B's.
- `PATTERN-owner-machine`'s intent is that only the owning machine starts work.
- Fix: recording and running a front action requires this machine to own the fronted worktree's subject.

**F6 — MEDIUM, design — D8 bound "a board edit cannot make dobby run an arbitrary command"**
- The bound covers the command but not the routing.
- A released front task is not run-held, so `lym board upsert` may change its `project`. The task then fronts and blocks another worktree, and its `adopt-check` runs for that other subject, since the action takes "the subject of the task's parent worktree".
- `lym board rename` keeps every field (`doc.go:263-269`). `fronttask`'s record, keyed by the old slug, then reads gone and runs the action at once. The renamed entry reads open with no record, so dobby records it again and runs the action a second time when it lands.
- Neither edit is refused or bounded in D8.

**F7 — LOW, consistency — D4 against the declared `PATTERN-adopt-hold`**
- The declared text says only the green check lifts a hold, and `lym tender resume` by hand.
- `lym batten cancel <slug>` lifts every pause entry of the slug's own scope through `tenderengine.Lift` (`cancelcmd.go:466-486`), a `HoldCaller` entry included.
- D4 never names this third lifter, so the record contradicts the declared PATTERN text. Either D4 names cancel's lift or the declared entry carves it out.

**F8 — MEDIUM, design — D6/D9 stop way forward**
- On a stop marker, the darn producer always appends `darnStopWayForward` ("record the answer with `lym loom decision add`, then run `lym loom resume`") to the stop report (`darnwrite.go:30`, `darnwrite.go:138-147`).
- The halt reason of a machine-failure stop therefore names `lym loom resume`, which D6 says it must never name. D7's `halted` line, which names "the way forward its stop report gives", inherits the contradiction.
- D9 must state that this stop drops or replaces that suffix.

**F9 — LOW, consistency — D7 `halted`**
- "Batten run record reads halted" names no real record state. The run-record states are `running`, `paused`, `done`, `blocked`, `failed` and `awaiting` (`shedengine/status.go:18-23`).
- The record should name which of them read `halted`.

**F10 — MEDIUM, design — D7 `running_here`**
- Counterexample: a run cancelled on the old machine. Cancel clears the claim to `draft` and removes the run's records (`cancelcmd.go` help text, `clearRunClaim`).
- After the reconcile it is in this machine's pre-reconcile list (`adopt.go:186`) but not in the post-reconcile `movedRuns` (`adopt.go:216`).
- It is not owned elsewhere. It is not `landed`, since its entry is present and draft. It is not held, paused or halted. It falls through to `running_here`, a false reading.
- `runs`' population is also undefined: "one item per run under the named subjects" never says whether it is the pre-reconcile list, the post-reconcile list, or also covers held released tasks and the unstarted fix task, which have no run record and no row.
- So `waiting_for_check` for a held released task, and `running_here` for an unstarted fix task, have no stated source.

**Tool calls**
1. `liftSlugPauses` in `internal/battencli/cancelcmd.go`, plus a grep for run-directory removal in `battenshed`'s Land and teardown files. My context held cancel's help text but not which pause entries cancel lifts, nor whether Land removes the run directory.
