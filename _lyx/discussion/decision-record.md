# board-lock-running — decision record

## Goal

A board entry claimed by a run stays the scope that run is executing.
A write that would change a claimed entry is refused with the reason, naming the run, and the way forward: put the new finding in a note of its own.
The lock starts the moment a batten run claims the task, before the pair exists, so no edit slips in between the claim and the run's first loom transition.

## Scope

In:

- The lock predicate and its enforcement in `internal/boardengine`, inside the board's write critical section, covering every mutating path: `upsert`, `upsert-batch`, `remove`, `merge` (both its removed entries and its upserted entry), `set-deps`, `promote`, and `intake import` with `into`.
- The claim write: `lyx batten run` sets the entry's status before its Worktree-Create row creates the pair.
- Docs: `internal/boardengine/doc.go` (a new section on the run lock), `internal/boardcli/doc.go` (which verbs refuse), the batten package doc that describes the claim write, and `plugins/ly/skills/board/SKILL.md` (a claimed entry takes no scope edits; findings go to a note).

Out:

- Any change to how loom writes status per transition or how landing sets `done`.
- Clearing the status automatically when an unlanded pair is removed (`lyx fabric remove`); the refusal names the manual way out instead.
- Any change to `intake list` and `intake close`, which touch no entry.
- README rendering: the status is already shown on each entry line.

## Decisions

1. **Lock predicate.**
   An entry is locked when its `status` is set, non-empty and not `done`.
   Rationale: the board body states the status already marks a running task, loom writes `<state> · <producer>` on every transition (running, paused, blocked, failed, awaiting all mean a run holds the entry), and landing writes `done`, which releases it.
   A status set by hand also locks; that is the same signal on the board and the same `set-status` clears it.

2. **What is locked: every field but `status`, and the entry's existence.**
   On a locked entry the store refuses a write that changes any field other than `status` (title, kind, labels, issues, recipe, depends_on, isolated, brief, body, short_name), and refuses removing it.
   Rationale: the operator's list (body, brief, issues, labels, merge, intake `into`) is the scope; title, kind, recipe and depends_on describe or drive the same run (Seed-Child reads `recipe` from the board after the claim), so one rule "a claimed entry is read-only but for its status" is simpler to state, test and reason about than a field allowlist.
   Bound: a write that changes nothing on the locked entry (an upsert repeating its current values) is not a change and passes.

3. **`status` stays writable on a locked entry, in its own write only.**
   `set-status`, a `merge`'s `set_status` step, and an upsert whose only difference from the stored entry is `status` pass.
   An upsert that changes `status` together with any other field is judged against the stored status, so it is refused while the stored entry is locked.
   Rationale: loom's per-transition write and landing's `done` go through `SetStatus` and must keep working; clearing the status is the explicit unlock.
   Bound: the unlock is a separate, visible `set-status` write; no single write both unlocks and edits.

4. **Enforcement point: the store, inside `boardCriticalSection`.**
   The check runs in `internal/boardengine`'s store mutators against the entry as loaded under the board file lock, so every caller (`boardcli` verbs, intake, batch, any Go caller) gets it and no race can slip an edit between check and save.
   A refused write mutates nothing: the critical section already discards the in-memory store on error, and `upsert-batch` and `merge` refuse as a whole when any one entry they touch is locked.
   Rationale: one chokepoint, matching how labels, kind rules and dependencies are already validated in the store.

5. **The refusal.**
   A single error type in `boardengine`, matchable with `errors.Is` against an exported sentinel, whose message names the entry's slug, its current status (which carries the run's state and producer), and two ways forward:
   record the new finding as a note of its own (`lyx board upsert` with `"kind":"note"`), and, when no run holds the entry any more, clear the status with `lyx board set-status` first.
   The CLI prints it through the existing JSON error envelope.
   Rationale: the operator asked for the reason and the run to be named; the slug is the run's id and pair name, and the status says where the run stands.
   Board is not webster, shed or loom, so `PATTERN-refusal-way-forward`'s spec-row requirement does not apply; its way-forward rule is followed anyway.
   The named commands are board verbs, none denied, per `PATTERN-no-denied-recovery`.

6. **`intake import --into` a locked entry refuses before any GitHub call.**
   The board write already precedes the comment and close, so a refused fold leaves the issue open and uncommented.
   Import as a new note (`slug`) is never affected.

7. **`merge` checks both sides.**
   It is refused when any `remove_slugs` entry is locked, or when the upserted entry already exists, is locked, and would change (carried issues count as a change).
   Rationale: the 2026-10-06 incident was a fold into a running entry; removing a running entry would drop the run's board record.

8. **The claim write: batten, before the pair exists.**
   `lyx batten run <slug>` writes the entry's status as `running · Worktree-Create` (the format of loom's `boardStatus`, with batten's row name) inside its Worktree-Create row, before `Topology.Add`, and only on that row.
   It skips an absent entry and a `done` entry, exactly as loom's `SetBoardStatus` does.
   From loom's bootstrap on, loom's own per-transition writes take over; batten writes no status for Seed-Child, Run-Shed or Worktree-Teardown, so a Run-Shed bounce never overwrites loom's status.
   A failed claim write warns and the run continues, the same disposition as loom's board write.
   Rationale: shed's `CommitStatus` fires only after a producer returns, so the first batten transition comes after the pair is created; the claim has to be written inside the row to precede it.
   The board is the hub's overview, never the run's bookkeeping, so the board being unwritable does not stop a run.
   Bound: on a failed claim write, the lock window is open until loom's first transition, as it is today.

9. **Opening the hub board from batten goes through one shared opener.**
   Batten's write needs a configured board (a bare-path `Board` cannot write); `loomcli`'s `openHubBoard` and the new batten writer use one shared function rather than two copies.
   Where it lives is the Plan's call, within `PATTERN-hub-containment` (the board is reached at the hub only) and `PATTERN-cliwire-sole-wiring`.

10. **No new PATTERN entry.**
    The lock is a rule of one module, enforced at one site inside it and documented in its `doc.go`; it adds no cross-cutting invariant.

## Constraints

- `PATTERN-hub-containment`: the claim write reaches `_board` from the hub path, never through a worktree link.
- `PATTERN-no-denied-recovery`: the refusal names only board verbs.
- `PATTERN-shed-producer-seam`: the claim write lives in `battencli`'s wiring closure for Worktree-Create, not in `shedengine` or `battenshed`'s told-geometry code (`PATTERN-told-geometry`); `battenshed` stays free of `lyxcwd`.
- `PATTERN-test-economy`, `PATTERN-test-speed`: the store rules are tier-1 table tests in `boardengine`; the claim write is driven through the existing batten wire seams with a stub writer where possible.
- `PATTERN-documentation-lifecycle` and the CLAUDE.md docs rule: `doc.go` updates and the board skill text land in the same commit.
- Test budget: a new top-level test in a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Board writes stay one-shot and daemonless; the check adds no I/O beyond the store already loaded under the lock.

## Auto-mode assumptions

- Hand-set statuses lock too (Decision 1); nobody sets non-run statuses today, and `set-status` null unlocks.
- Title, kind, recipe, depends_on, isolated and short_name are locked along with the operator's named fields (Decision 2).
- A failed claim write warns rather than refusing the run (Decision 8).
- No automatic unlock when an unlanded pair is removed; the refusal message carries the manual unlock (Scope).
- The claim status value is `running · Worktree-Create` rather than a new vocabulary word (Decision 8).

## Open risks

- A run abandoned without landing leaves its entry locked until someone clears the status; mitigated only by the refusal naming `set-status`.
- An orch or skill flow that today edits a running entry's body mid-run (for example, recording a finding on the task itself) now fails; the board skill text and the refusal redirect it to a note.
- The `git-robustness` task plans "batten refuses a done slug", which touches the same Worktree-Create row; the two changes must compose (the claim write skips a done entry, so they do not conflict in behavior).

## Acceptance criteria

- On an entry whose status is `running · Webster` (and likewise `blocked · …`, `paused · …`, `failed · …`, `awaiting · …`), each of these is refused, mutates nothing, and its message names the slug, the status and both ways forward: upsert changing body, brief, labels, issues, title, kind, recipe, depends_on; `set-deps`; `remove`; `merge` removing it; `merge` upserting into it with a change; `upsert-batch` with one entry touching it (no entry of the batch lands); `intake import` with `into` it (no GitHub comment or close happens).
- On the same entry these pass: `set-status` to another value; `set-status` to null; a `merge` whose `set_status` targets it; an upsert identical to the stored entry; an upsert whose only change is `status`.
- An upsert that changes `status` and another field in one write is refused while the stored entry is locked.
- An entry with no status, an empty status, or status `done` takes every write as today.
- `intake import` as a new note and writes to other entries (including adding a dependency on the locked entry from another task) pass.
- The refusal is matchable with `errors.Is` against the exported sentinel.
- `lyx batten run` on an entry with no status writes `running · Worktree-Create` before the pair is created, writes nothing for an absent or `done` entry, and a failing board write warns and the run proceeds.
- Batten writes no board status after Worktree-Create; loom's transitions remain the only later writer.
- `go test ./...` and `go test -tags integration ./...` are green.

## Notes for the plan writer

- Status per transition: `newCommitStatusSeam`, `boardStatus` and `loomCommitStatusDeps.SetBoardStatus` in `internal/loomcli/wiring.go`; landing's `done` in `internal/loomcli/landingdeps.go`, beside `openHubBoard`.
- Store mutators and `validateWrite`, `isDone` live in `internal/boardengine/store.go`; the critical section in `board.go`; intake fold in `intake.go`.
- Batten's Worktree-Create closure is in `internal/battencli/wire.go`; it already opens a bare-path board for Seed-Child's `ReadBoardType`.
