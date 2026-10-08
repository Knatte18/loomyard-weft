# board-lock-running — support log

## Interview

Autonomous mode: no operator answers; every pick below is an auto-pick recorded in the ledger.

1. Read the board entry and the board list.
   No other entry plans board write rules; `git-robustness` touches batten's Worktree-Create row ("batten refuses a done slug").
2. Explored `internal/boardengine` (store mutators, `validateWrite`, critical section, intake fold), `internal/boardcli` docs, loom's board status writes in `internal/loomcli/wiring.go` and `landingdeps.go`, batten's wiring and commit-status seam, and `pairteardown` callers.
   Found: only loom writes status (per transition, skip done) and landing writes `done`; shed's `CommitStatus` runs after a producer returns, so batten's first possible commit-time write already follows pair creation.
3. Settled the lock predicate (status set, non-empty, not `done`), the locked surface (every field but status, plus existence), the status carve-out, the enforcement point (store, under the board lock), the refusal content, and the claim point (inside Worktree-Create, before `Topology.Add`).

## Rejected alternatives

- **Lock only the operator's named fields (body, brief, issues, labels).**
  Leaves title, kind, recipe and depends_on editable on a running task; recipe is read by Seed-Child after the claim, so a change there alters the run; a two-tier field list is harder to state than "read-only but status".
- **Enforce in `boardcli` verbs.**
  Misses Go callers and duplicates the check per verb; the store already owns write validation.
- **A `--force` override on scope writes.**
  An extra bypass the operator did not ask for; clearing the status with `set-status` is already an explicit, visible unlock.
- **Claim from batten's `CommitStatus` seam on every batten transition.**
  Fires only after Worktree-Create returns, so it leaves the creation window open; and Run-Shed bounces would overwrite loom's status with `running · Run-Shed`.
- **Claim in `lyx loom start`.**
  Loom's bootstrap persist already writes the first status; the open window is batten's pair creation, seeding and spawn.
- **Refuse the batten run when the claim write fails.**
  The board is the hub's overview, never the run's bookkeeping; loom's write already warns on failure.
- **Clear the status automatically on `lyx fabric remove` of an unlanded pair.**
  Couples fabric teardown to the board for a rare case; the refusal names the manual unlock.
- **A new status word such as `claimed`.**
  Adds vocabulary for a seconds-to-minutes window; `running · Worktree-Create` follows the existing `<state> · <producer>` format.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Which statuses lock? — Resolved (auto-pick): any set, non-empty status other than `done`; it is the board's existing run marker.
- Q2: Which fields are locked? — Resolved (auto-pick): every field but `status`, plus removal; simpler single rule, covers recipe read after the claim.
- Q3: Can status change on a locked entry? — Resolved (auto-pick): yes, in a status-only write; loom and landing depend on it, and it is the unlock.
- Q4: Where is the check? — Resolved (auto-pick): `boardengine` store, inside the board critical section.
- Q5: What does the refusal say? — Resolved (auto-pick): slug, current status, "record it as a note", and "clear the status with set-status if no run holds it".
- Q6: Where does the claim happen? — Resolved (auto-pick): batten's Worktree-Create closure before the pair is created, writing `running · Worktree-Create`, skipping absent and done entries.
- Q7: Does a failed claim write stop the run? — Resolved (auto-pick): no, it warns, as loom's board write does.
- Q8: Unlock for an abandoned run? — Resolved (auto-pick): manual `set-status` null, named in the refusal; no automatic clear.
- Q9: New PATTERN entry? — Resolved (auto-pick): no; the rule is one module's, enforced at one site and documented in its `doc.go`.
- Q10: Does a no-op upsert on a locked entry refuse? — Resolved (auto-pick): no; it changes nothing.
