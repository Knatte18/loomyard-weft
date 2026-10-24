```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard a card adds, can a real entry point reach its state, and does the guarded site recur unguarded?

**Per guard, reachable or not:**

- Card 2 `MarkBeam` refusal of a missing record: reachable. Promote step 2, or create's mark step, with the records worktree missing.
- Card 4 parent refusals (unknown parent, prime's name, detached, other branch): reachable through `lyx fabric add --parent` (card 6) and the test rows.
- Card 5 warp branch checked out elsewhere: reachable. Records branch checked out in another records worktree: reachable only when that worktree sits off the pair's own records path. I left this unjudged because Add's existing-path preflight was not read.
- Card 6 `--local` with `--parent` or `--adopt-local`: reachable.
- Card 8 refusals: missing record is reachable. Dirt outside the pathspec is reachable at promote step 4. Root pair (empty `ParentBranch`) is in practice reachable only through the test's hand rewrite; that is acceptable as an executor invariant guard.
- Card 9 `normalizeTask` refusals:
  - Forbidden project fields: reachable through upsert.
  - `order` on a task or note: **not reached by its planned row at card 9**. The upsert allowlist refuses the key first, and only card 10 admits it.
  - `beam` on a task or note: **not reachable through any write path once card 11 lands**. `checkBeamPayload` refuses `beam: true` first and drops `beam: false` as a no-op.
- Card 9 `checkProjectRules` (no task may depend on a project): reachable through upsert or `set-deps`.
- Card 9 `Store.Promote` of a project: reachable through `lyx board promote <project>`.
- Card 10 membership refusals: reachable.
  - The run-status project refusal is reachable during a landing.
  - The project-change and rename refusals with a pair are reachable once card 17 wires `PairExists`.
  - The merge refusal is reachable.
- Card 11 `checkBeamPayload`: reachable. `ToProject` without `Beam` is a wiring guard, reached directly by its `SetOwners` test row.
- Card 13 `checkNewSlugs`: reachable for a new task or project, a note promote, a note-to-project kind change by upsert, and a rename.
- Card 19 `refuseNonPrime` for a non-beam pair: reachable.
- Card 19 `refuseBeamStart`:
  - Owned elsewhere: reachable.
  - Run-held landing: reachable after `lyx beam land` claims.
- Card 21 `hibernateengine.refuseCaller` widening: reachable.
- Card 23 `ResolveBattenPrime`:
  - Non-beam pair: reachable.
  - Run-held beam: reachable.
  - Self-pair refusal: reachable only **before** a landing claims (see finding 3).
- Card 24 `windowSite` with the beam's worktree absent, falling back to the prime: reachable. Worktree-Create then stops.
- Card 27 widened `RecordsPresent`: reachable.
- Card 28 `beamForkPrecondition`:
  - Absent beam pair, both branches: reachable.
  - Run-held beam: reachable. The landing's after-claim refusal (card 36) stops it with the entry still claimed, and a member is started next. A fresh member cannot be added after the claim, because card 10 refuses a `project` naming a run-status project.
- Card 30 describe refusal: reachable. Neither brief nor title is a required upsert key (`store.go:443`), and a beam can add no commits.
- Card 34 refusals, including an absent parent beam: reachable.
- Card 35 `Promoted` start-guard refusal:
  - After step 3: reachable, as a fresh start with `found` false.
  - After step 2: **not shown reachable** (see finding 2).
- Card 35 end-only stop: reachable.
- Card 36 `BeamLandRefusal`: reachable at land, at the start guard, and after the claim (a member added between the start guard and the claim).
- Card 36 start-guard refusals for a project entry:
  - Grouping project: reachable.
  - Beam with no local pair, on the owner machine after create stopped at step 1: reachable.
  - Unreadable record naming `lyx fabric reconcile`: reachable, since a fresh start runs no reconcile (`arm.go:637`).
- Card 36 Worktree-Create precondition for a project entry with no pair: narrowly reachable, by a resume after the pair is removed by hand or by the card's fake row.
- Card 37 refusal of a grouping project: reachable.

**Findings:**

1. MEDIUM, consistency. Card 9 (`09-board-project-kind.md`): the `normalizeTask` refusals of `order`/`beam` on a task or note are not reached by the row that is meant to reach them.
   - At card 9, `TestKindDependencyRules` drives through `Store.UpsertTask`, and `validateUpsertFields` refuses `order` and `beam` first as unknown fields. That text also names the field, so the row passes without reaching the branch. Card 9 itself says "No upsert key is added here".
   - From card 11 on, `checkBeamPayload` pre-empts a `beam` on a task on every upsert, batch and merge path, so that branch is reachable only through replay of a hand-edited `board.json`.
   - Fix: move the `order` row to card 10, after the key is admitted. Treat `beam`-on-task as a load and replay invariant guard reached through a replay test, or drop it.
   - Evidence: `internal/boardengine/store.go:442-476` (allowlist and `validateUpsertFields`), `internal/boardengine/store_test.go:108-138` (the test drives `UpsertTask`), `internal/boardengine/replay.go:283,365` (`normalizeTask` on replay).
2. MEDIUM, design. Card 35 (`35-beam-promote.md`): the `Promoted` refusal claims to cover "a promote stopped after step 2 or 3". After step 2, though, the run is still claimed and its status file still exists, because step 3's `RemoveRecords` and `ClearClaim` have not run.
   - So `lyx batten run <slug>` takes the found-status resume path.
   - The existing guard gates its pair-state check on `!found`. The card leaves the new check's position open, and its single test row does not say whether a status file is present.
   - If the check is gated like the left-pair one, a resume after step 2 restarts the ended run inside a pair that already carries the mark.
   - Fix: state that the `Promoted` check runs whether or not a status file is found, and add a `TestStartGuard_Refuse` row with `found` true.
   - Evidence: `internal/battencli/startguard.go:51-89` (`refuse` gates the pair check on `!found`), `internal/battencli/arm.go:637` (a found status resumes after the owner preflight and reconcile).
3. LOW, consistency. Card 23 (`23-batten-guard.md`) says the self-pair refusal is reached by "only a beam ... at its own landing". At its landing the beam's entry is run-held, so `ResolveBattenPrime`'s run-held refusal fires first. The self-pair refusal is reached only before a landing claims, for example `run <beam>` from inside the beam while its entry is open. The planned `TestArmAt_StartGuardRefusesBeforeAnyWrite` row reaches that state, but the Intent misnames it. Evidence: card 23's own ordering of the run-held refusal inside `ResolveBattenPrime` ahead of the verb's self-pair check.

I found no guarded site that recurs unguarded on a path D8 or D11 binds.

Tool calls:
1. Grepped batten, battenshed and adoptcli kind checks, `normalizeTask` callers and the `TestKindDependencyRules` body, to see what upstream checks a project or a payload meets. The manifest had none of these.
2. Read `battencli/owner.go#resumeOwnerClaim`, the `UpsertPayloadKeys` enforcement sites and `task.go`'s parse path, to learn whether payload keys are refused before `normalizeTask`.
3. Read `store.go#validateUpsertFields` and `summaryparser.ValidateDescription`, to confirm the allowlist pre-empts card 9's rows and that card 30's refusal is reachable.
4. Read `battencli/resumereconcile.go#reconcileRun` and the order of `refuseAtStart`, to learn whether a reconcile runs before the start guard on a fresh start.
