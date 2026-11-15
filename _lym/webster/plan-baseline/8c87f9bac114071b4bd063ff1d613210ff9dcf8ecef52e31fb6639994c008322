# Card 3 — runfsm

**Create:**
- `internal/runfsm/doc.go`
- `internal/runfsm#State`
- `internal/runfsm#Landed`
- `internal/runfsm#Cancelled`
- `internal/runfsm#Cancelling`
- `internal/runfsm#Paused`
- `internal/runfsm#Halted`
- `internal/runfsm#Landing`
- `internal/runfsm#Running`
- `internal/runfsm#Starting`
- `internal/runfsm#Queued`
- `internal/runfsm#NoRun`
- `internal/runfsm#Order`
- `internal/runfsm#Records`
- `internal/runfsm#Reading`
- `internal/runfsm#Derive`
- `internal/runfsm#Inputs`
- `internal/runfsm#Answer`
- `internal/runfsm#AnswerAct`
- `internal/runfsm#AnswerWait`
- `internal/runfsm#AnswerHandOver`
- `internal/runfsm#Reason`
- `internal/runfsm#ReasonPushOwed`
- `internal/runfsm#ReasonOwnedElsewhere`
- `internal/runfsm#ReasonCodeTip`
- `internal/runfsm#ReasonHeld`
- `internal/runfsm#ReasonPauseCovers`
- `internal/runfsm#ReasonPaused`
- `internal/runfsm#ReasonCancelling`
- `internal/runfsm#ReasonTerminal`
- `internal/runfsm#Verdict`
- `internal/runfsm#Act`
- `internal/runfsm#Verdict.Refusal`
- `internal/runfsm#Aside`
- `internal/runfsm#Judge`
- `internal/runfsm#TransitionState`
- `internal/runfsm_test#TestDerive`
- `internal/runfsm_test#TestAct`
- `internal/runfsm_test#TestTransitionState`

**Extends:**
- `pattern/PATTERN-leaf-packages.md`

**Intent:** The one run FSM of D1: a leaf package that imports only the standard library and `internal/shedengine`, for the record type, and does no I/O.
Callers read the tracked records and the machine-local facts and hand them in; every reader of a run's state and every verb that acts on a run goes through it.
`pattern/PATTERN-leaf-packages.md` gains an `internal/runfsm` section stating that import set and that it derives and decides, reading nothing.

`Records` holds the tracked records alone:
`Released` (a task entry whose board status is empty), `BoardDone` (the entry's status is `done`),
`Batten shedengine.Status` with `BattenFound`, `BattenStarted` (batten's row is `Run-Shed` or later),
`TaskBranchOnOrigin`, `Child shedengine.Status` with `ChildFound`, and `ChildLanding` (the child is at `Finalize` or past it, or batten is at `Land` or later).
The caller fills the row facts from the recipes, so this package names no row.

`Order` returns D1's states highest precedence first, and `Derive` walks `Order()` and takes the first state whose condition holds, so production reaches `Order` through `Derive`:
1. `Landed`: batten found and `done`, or no batten record and `BoardDone`.
2. `Cancelled`: a cancel intent with its finish.
3. `Cancelling`: a cancel intent without a finish.
4. `Paused`: a pause intent, or batten's or the child's shed state `paused`.
5. `Halted`: the child `blocked`, `failed` or `awaiting`.
6. `Landing`: `ChildLanding`.
7. `Running`: `BattenStarted`.
8. `Starting`: a batten record at a row before `Run-Shed`.
9. `Queued`: `Released` with no batten record.
Otherwise `NoRun`, a slug no run holds, such as a draft.
A batten record whose shed state is `cancelled` while `TaskBranchOnOrigin` is false counts as no batten record, the crash leftover D3 names, so a released task with one reads `Queued`;
a kept pair's task branch stays on origin, so its record keeps reading `Cancelled`.
`Reading` carries `State` and `PairKept`, true for `Cancelled` whose finish kept the pair.

`Inputs` holds `State`, `OwnedElsewhere` with `Owner` and `AdoptSubject`, `Held`, `PauseCovers` (this machine's pause record covers the run),
`PushOwedOffline`, `CodeTipMissing` and `PastParentMerge` (origin's fetched parent branch holds the task's work).
`Act` answers in D1's order, and `Verdict` carries `Answer`, `Reason` and `State`, plus `CancelMachine` and `CancelRemove`, which `Judge` copies from the record's cancel intent for the cancelling refusal,
and `PausedByIntent`, which `Judge` sets when the state is `Paused` and the record still holds a pause intent once the aside is applied, for the paused refusal:
1. `PushOwedOffline`: wait, `ReasonPushOwed`.
2. `OwnedElsewhere`: hand over, `ReasonOwnedElsewhere`, a landing past its parent-side merge included.
3. `CodeTipMissing`: wait, `ReasonCodeTip`.
4. `State == Landing && PastParentMerge && !PauseCovers`: act, so a landing past its merge finishes under a hold.
5. wait on `Held` (`ReasonHeld`), `PauseCovers` (`ReasonPauseCovers`), state `Paused` (`ReasonPaused`), `Cancelling` (`ReasonCancelling`), `Landed` or `Cancelled` (`ReasonTerminal`).
6. act otherwise, `NoRun` and `Queued` included.
The owner, hold, owed-push and code-tip inputs are passed in by callers; this part's callers fill the owner, the pause and the hold they read today, and the later parts fill the rest, so `PushOwedOffline` and `CodeTipMissing` stay false until then.

`Aside` holds `Pause`, `Cancel`, `Hold`, `ShedPause` and `PushOwed`.
`Judge` clears what aside names before deriving and deciding: `Pause` drops the pause intent and `PauseCovers`, `Cancel` the cancel intent, `Hold` `Held`, `ShedPause` reads a shed state `paused` as not paused, and `PushOwed` drops `PushOwedOffline`.
It returns `Derive`'s reading and `Act`'s verdict over it.
`lym tender resume <slug>` sets `Pause`, `Cancel`, `ShedPause` and `PushOwed` aside, and `lym batten cancel` sets `Cancel`, `Pause`, `Hold`, `ShedPause` and `PushOwed` aside, as D1 and D3 say.
`lym batten run`, `lym batten step`, `lym loom start` and `lym loom resume` set `ShedPause` aside, since each is the verb that resumes its own record's shed pause and would otherwise refuse its own resume (Shared Decisions).

`Verdict.Refusal` returns nil for act and otherwise one error naming the slug, the reason and its way forward, with retry the caller's re-run:
push owed names the re-run once origin answers;
owned elsewhere names the owner, the same verb on that machine, and `lym adopt <subject>` on this machine first;
code tip names the re-run once the code push lands;
held names the wait for the adopt that holds the hub to finish;
pause covers names `lym tender resume <slug>` then the re-run;
paused, with `PausedByIntent`, names `lym tender resume <slug>` then the re-run, since only that verb clears a pause intent, which a removed `.lym` or a machine that took the run over leaves with no local pause entry;
paused without it names `lym batten run <slug>` from the prime, which resumes the shed pause, or `lym loom resume` in the task worktree for the inner run;
cancelling names the cancel's machine (`CancelMachine`) and `lym batten cancel <slug>` run there to finish it, with `--remove` when `CancelRemove` is set, so the finish keeps the original cancel's choice;
terminal names, for `Landed`, that the slug has landed and, to run it again,
`lym batten cleanup --apply` from the prime, which removes its run directory, `lym fabric remove <slug>` if its pair still stands,
for a task `lym board set-status '{"slug":"<slug>","status":null}'`, which releases the entry, and deleting the leftover task branch, locally and on the remote;
or else giving the new task another slug, the one way forward for a landed beam, whose `done` status the board refuses to release;
for `Cancelled`, `lym batten cancel <slug> --remove`, which removes its record and pair before a fresh run.
No text names `lym adopt --take-hold`.

`TransitionState` is the board display of one transition, which the status writers call (card 6): `paused` reads `Paused`, `blocked`, `failed` and `awaiting` read `Halted`; otherwise `landing` reads `Landing`, `started` reads `Running` and neither reads `Starting`.

Tests, all tier0, table-driven:
- `TestDerive`: one row per state from records alone, and a row per adjacent pair of states whose conditions both hold reading the earlier one;
  a slug with no batten record and board `done` reads `Landed`;
  a cancelled record with the task branch gone and a released entry reads `Queued`, and with the branch on origin reads `Cancelled` with `PairKept`;
  a draft with no record reads `NoRun`.
- `TestAct`: one row per branch of the order above, among them hand over for an owned-elsewhere landing past its merge;
  act for a held landing past its merge, wait for the same with a pause covering it or with a push owed offline;
  wait for held, pause, `Paused`, `Cancelling`, an owed push offline, a missing code tip and each terminal state; act otherwise.
  Rows through `Judge` show each `Aside` field turning the matching wait into act, `PushOwed` included.
  Each wait and hand-over row asserts `Refusal`'s exact text, and the act row asserts nil;
  the terminal `Landed` row's text names the run-directory cleanup, the pair removal, the task release, the branch deletion and the other slug;
  the cancelling rows assert the machine named and `--remove` present for a remove intent and absent for a keep one;
  two `Paused` rows through `Judge` assert each paused wording: a pause intent with no local pause entry names `lym tender resume <slug>`, and a shed state `paused` names `lym batten run <slug>`.
- `TestTransitionState`: one row per mapping.

`doc.go` states the package's place: the run's state from tracked records alone, the act rule, and that the board's `<state> · <row>` is a display of it.

**ImpactSummary:** A new package with no callers yet, plus one section added to the leaf-packages background.
