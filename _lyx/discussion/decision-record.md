# board-lock-running — decision record

## Goal

A board entry claimed by a run stays the scope that run is executing.
A write that would change a claimed entry is refused with the reason, naming the run, and the way forward: put the new finding in a note of its own.
The lock starts the moment a batten run claims the task, before the pair exists, so no edit slips in between the claim and the run's first loom transition.

## Scope

In:

- The lock predicate and its enforcement in `internal/boardengine`, inside the board's write critical section, covering every mutating path: `upsert`, `upsert-batch`, `remove`, `merge` (both its removed entries and its upserted entry), `set-deps`, `promote`, `prune`, and `intake import` with `into`.
- The run-status form, declared once in `boardengine` and composed through it by loom's per-transition write and batten's claim.
- The claim write: `lyx batten run` sets the entry's status before its Worktree-Create row creates the pair.
- Docs: `internal/boardengine/doc.go` (a new section on the run lock), `internal/boardcli/doc.go` (which verbs refuse), the batten package doc that describes the claim write, and `plugins/ly/skills/board/SKILL.md` (a claimed entry takes no scope edits; findings go to a note).

Out:

- Any change to which status loom writes per transition or how landing sets `done`; loom only composes its status through the shared form.
- Clearing the status automatically when an unlanded pair is removed (`lyx fabric remove`); the refusal names the manual way out instead.
- Any change to `intake list` and `intake close`, which touch no entry.
- README rendering: the status is already shown on each entry line.

## Decisions

1. **Lock predicate: a run status locks, nothing else does.**
   An entry is locked when its `status` is a run status: the `<state> · <producer>` form loom writes on every transition and batten's claim writes (Decision 8).
   Any other status leaves the entry unlocked: none, empty, `done`, `abandoned`, or a hand-set word such as `active`.
   Rationale: a run status is the board's existing marker that a run holds the entry, and landing replaces it with `done`, which releases it.
   Bound: `boardengine` declares the run-status form once, as the function that composes it and the predicate that recognizes it; loom's `boardStatus` and batten's claim compose through it, so the writers and the lock cannot disagree on the form.

2. **What is locked: every field but `status`, and the entry's existence.**
   On a locked entry the store refuses a write that changes any field other than `status` (title, kind, labels, issues, recipe, depends_on, isolated, brief, body, short_name), and refuses removing it.
   Rationale: the operator's list (body, brief, issues, labels, merge, intake `into`) is the scope, and title, kind, recipe and depends_on describe or drive the same run: Seed-Child reads `recipe` from the board after the claim.
   Bound: a write that changes nothing on the locked entry (an upsert repeating its current values) is not a change and passes.
   Bound: `prune` dropping a done slug from a locked entry's `depends_on` passes, under Decision 4's prune exception; `prune` never removes a locked entry, since it removes only `done` entries.

3. **`status` stays writable on a locked entry, in a status-only write.**
   A write whose net change on a locked entry (Decision 4) is to `status` alone passes: `set-status`, a `merge`'s `set_status` step, and an upsert whose only difference from the stored entry is `status`.
   A write that changes `status` together with any other field of a locked entry is refused.
   Rationale: loom's per-transition write and landing's `done` go through `SetStatus` and must keep working; clearing the status is the explicit unlock.
   Bound: no single write both unlocks and edits, because Decision 4 judges every write against the entry as loaded before it.
   `set-status` itself is unguarded: anyone who runs it can clear a live run's status.
   The refusal names the check to run before clearing (Decision 5), and the residual risk is in Open risks.

4. **Enforcement point: the store, inside `boardCriticalSection`, on each entry's net change.**
   The check runs in `internal/boardengine`'s store mutators, so every caller (`boardcli` verbs, intake, batch, prune, any Go caller) gets it and no race can slip an edit between check and save.
   Each entry a write touches is judged by its net change from the entry as loaded under the board file lock, across every step of the write: the write is refused when the loaded entry is locked and the net change touches a field other than `status` or removes the entry.
   Prune exception: removing from a locked entry's `depends_on` a slug that the same write removes as a `done` entry is not a scope change and passes; only `prune` makes that change, the dependency is satisfied, and every other `depends_on` change on a locked entry is still refused.
   A batch that lists one slug twice, or a `merge` whose upsert and `set_status` hit the same entry, is judged on the combined change, so a status clear earlier in the same write never unlocks a later edit.
   A refused write mutates nothing: the critical section already discards the in-memory store on error, and `upsert-batch` and `merge` refuse as a whole when any one entry's net change is refused.
   Rationale: one chokepoint, matching how labels, kind rules and dependencies are already validated in the store.

5. **The refusal.**
   A single error type in `boardengine`, matchable with `errors.Is` against an exported sentinel, whose message names the entry's slug, its current status (which carries the run's state and producer), and two ways forward:
   record the new finding as a note of its own (`lyx board upsert` with `"kind":"note"`), and, to unlock, run `lyx batten status <slug>` from the hub's prime worktree and clear the status with `lyx board set-status` only when no run holds the entry.
   No run holds the entry when that check refuses because the slug has no batten seed, reports state `done`, or reports a halted state (`paused`, `blocked`, `failed`, `awaiting`) whose run the operator has decided to abandon; a `running` state means a run holds it.
   Batten verbs refuse outside the prime, so an agent in a task pair cannot run the check; for it the note is the way forward, and the unlock is the prime's or the operator's step.
   The CLI prints it through the existing JSON error envelope.
   Rationale: the operator asked for the reason and the run to be named; the slug is the run's id and pair name, and the status says where the run stands.
   Board is not webster, shed or loom, so `PATTERN-refusal-way-forward`'s spec-row requirement does not apply; its way-forward rule is followed anyway.
   The named commands are board and batten verbs, none denied, per `PATTERN-no-denied-recovery`.

6. **`intake import --into` a locked entry refuses before any GitHub write.**
   The board write already precedes the comment and close, so a refused fold leaves the issue open and uncommented; the issue fetch that precedes the board write is a read and stays where it is.
   Import as a new note (`slug`) is never affected.

7. **`merge` checks both sides.**
   It is refused when any `remove_slugs` entry is locked, or when the upserted entry already exists, is locked, and its net change (Decision 4) touches a field other than `status` (carried issues count as a change).
   Rationale: the 2026-10-06 incident was a fold into a running entry; removing a running entry would drop the run's board record.

8. **The claim write: batten, before the pair exists.**
   `lyx batten run <slug>` writes the entry's status as `running · Worktree-Create` (the run-status form of Decision 1, with batten's row name) inside its Worktree-Create row, before `Topology.Add`, and only on that row.
   It skips an absent entry and a `done` entry, exactly as loom's `SetBoardStatus` does.
   From loom's bootstrap on, loom's own per-transition writes take over; batten writes no status for Seed-Child, Run-Shed or Worktree-Teardown, so a Run-Shed bounce never overwrites loom's status.
   A failed claim write warns and the run continues, the same disposition as loom's board write.
   Rationale: batten's first status commit comes only after Worktree-Create returns, so the claim is written inside the row to precede pair creation.
   The board is the hub's overview, never the run's bookkeeping, so the board being unwritable does not stop a run.
   Bound: on a failed claim write, the lock window is open until loom's first transition, as it is today.
   Bound: a `done` or absent entry is not claimed, so it is not locked by the claim.
   An entry created after the claim is locked from loom's next status change.
   A run on a `done` entry runs unlocked, as loom writes nothing there either; the `git-robustness` task's "batten refuses a done slug" removes that case, and until it lands the run proceeds as today.
   Bound: when the Worktree-Create row fails after the claim (`Topology.Add` refuses, or the pair is incomplete), the claim stays.
   The batten run still exists, stopped at Worktree-Create, and resumes with `lyx batten run <slug>`, as a stopped loom run keeps its last status; abandoning it instead takes the manual unlock of Decision 5.

9. **Opening the hub board from batten goes through one shared opener.**
   Batten's write needs a configured board (a bare-path `Board` cannot write); `loomcli`'s `openHubBoard` and the new batten writer use one shared function rather than two copies.
   Where it lives is the Plan's call, within `PATTERN-hub-containment` (the board is reached at the hub only) and `PATTERN-cliwire-sole-wiring`.

10. **No new PATTERN entry.**
    The lock is a rule of one module, enforced at one site inside it and documented in its `doc.go`; it adds no cross-cutting invariant.

### Added after Discussion (parent, 2026-10-08): Prune passes the lock

Decision: Decision 4's rule has one exception: removing a slug from a locked entry's depends_on passes when the same write removes that slug as done, which only prune does. Decision 2 points to this exception.

Rationale: Decision 4 as written refuses the prune that Decision 2 and the Acceptance criteria require.

### Added after Discussion (parent, 2026-10-08): Where and how a refused reader checks for a live run

Decision: The refusal says to run lyx batten status <slug> from the hub's prime worktree. No run holds the entry when the slug has no batten seed, when its batten status is done, or when the run is halted and the operator has decided to abandon it. Any other state counts as live. The Acceptance criterion on the refusal's message uses this wording.

Rationale: Batten verbs refuse outside the prime, and no batten status output reads as no run in progress.

## Constraints

- `PATTERN-hub-containment`: the claim write reaches `_board` from the hub path, never through a worktree link.
- `PATTERN-no-denied-recovery`: the refusal names only board and batten verbs.
- `PATTERN-shed-producer-seam`: the claim write lives in `battencli`'s wiring closure for Worktree-Create, not in `shedengine` or `battenshed`'s told-geometry code (`PATTERN-told-geometry`); `battenshed` stays free of `lyxcwd`.
- `PATTERN-test-economy`, `PATTERN-test-speed`: the store rules are tier-1 table tests in `boardengine`; the claim write is driven through the existing batten wire seams with a stub writer where possible.
- `PATTERN-documentation-lifecycle` and the CLAUDE.md docs rule: `doc.go` updates and the board skill text land in the same commit.
- Test budget: a new top-level test in a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Board writes stay one-shot and daemonless; the check adds no I/O beyond the store already loaded under the lock.

## Auto-mode assumptions

- Only a run status locks; `done`, `abandoned` and hand-set words such as `active` do not (Decision 1).
- Title, kind, recipe, depends_on, isolated and short_name are locked along with the operator's named fields (Decision 2).
- `prune` dropping a done dependency from a locked entry passes (Decision 2).
- A failed claim write warns rather than refusing the run (Decision 8).
- A claim whose Worktree-Create row then fails stays on the entry (Decision 8).
- `set-status` is not guarded; the refusal names the `lyx batten status` check instead (Decisions 3 and 5).
- No automatic unlock when an unlanded pair is removed; the refusal message carries the manual unlock (Scope).
- The claim status value is `running · Worktree-Create` rather than a new vocabulary word (Decision 8).

## Open risks

- A run abandoned without landing leaves its entry locked until someone clears the status; mitigated only by the refusal naming `set-status`.
- A batten run stopped at a failed Worktree-Create reads `running · Worktree-Create` on the board until it is resumed or its status is cleared.
- A batten process that died without persisting a halt still reads `running` in `lyx batten status`, so its entry stays locked until the operator judges the run dead and clears the status.
- An agent that skips the refusal's `lyx batten status` check can clear a live run's status and edit the entry; loom's next status change writes the status back, so the edit shows only in the board's history.
- An orch or skill flow that today edits a running entry's body mid-run (for example, recording a finding on the task itself) now fails; the board skill text and the refusal redirect it to a note.
- The `git-robustness` task plans "batten refuses a done slug", which touches the same Worktree-Create row; the two changes must compose (the claim write skips a done entry, so they do not conflict in behavior).

## Acceptance criteria

- On an entry whose status is `running · Webster` (and likewise `blocked · …`, `paused · …`, `failed · …`, `awaiting · …`), each of these is refused, mutates nothing, and its message names the slug, the status, both ways forward, `lyx batten status <slug>` run from the prime worktree, and the outcomes that mean no run holds the entry (Decision 5): upsert changing body, brief, labels, issues, title, kind, recipe, depends_on; `set-deps`; `promote` of a note; `remove`; `merge` removing it; `merge` upserting into it with a change; `merge` upserting into it with a change while its `set_status` clears it; `upsert-batch` with one entry changing it (no entry of the batch lands); `upsert-batch` listing it twice, first clearing its status and then changing its body; `intake import` with `into` it (no GitHub comment or close happens).
- On the same entry these pass: `set-status` to another value; `set-status` to null; a `merge` whose `set_status` targets it; an upsert identical to the stored entry; an upsert whose only change is `status`; an `upsert-batch` whose only item on it changes `status` alone.
- An upsert that changes `status` and another field in one write is refused while the stored entry is locked.
- `prune` on a board where the locked entry depends on a `done` entry removes the `done` entry and drops it from the locked entry's `depends_on`.
- An entry with no status, an empty status, or status `done`, `abandoned` or `active` takes every write as today.
- `intake import` as a new note and writes to other entries (including adding a dependency on the locked entry from another task) pass.
- The refusal is matchable with `errors.Is` against the exported sentinel.
- Loom's per-transition status and batten's claim are composed by `boardengine`'s run-status function and recognized by its predicate.
- `lyx batten run` on an entry with no status writes `running · Worktree-Create` before the pair is created, writes nothing for an absent or `done` entry, and a failing board write warns and the run proceeds.
- When `Topology.Add` fails after the claim, the entry keeps `running · Worktree-Create`.
- Batten writes no board status after Worktree-Create; loom's transitions remain the only later writer.
- `go test ./...` and `go test -tags integration ./...` are green.

## Notes for the plan writer

- Status per transition: `newCommitStatusSeam`, `boardStatus` and `loomCommitStatusDeps.SetBoardStatus` in `internal/loomcli/wiring.go`; landing's `done` in `internal/loomcli/landingdeps.go`, beside `openHubBoard`.
- Store mutators and `validateWrite`, `isDone`, `Prune` live in `internal/boardengine/store.go`; the critical section in `board.go`; intake fold in `intake.go`.
- Batten's Worktree-Create closure is in `internal/battencli/wire.go`; it already opens a bare-path board for Seed-Child's `ReadBoardType`.
