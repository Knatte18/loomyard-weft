# Round 1 fixer report — discussion

Every finding was validated against the code and fixed.

## Fixed

- **F1 (BLOCKING, prune).** Confirmed: `Store.Prune` rewrites survivors' `depends_on`.
  Added `prune` to Scope, a Bound in Decision 2 (dropping a done dependency from a locked entry passes; prune never removes a locked entry), an Auto-mode assumption, a ledger entry (Q11), a rejected alternative, and an Acceptance criterion.
- **F2 (MEDIUM, comparison base).** Confirmed: `UpsertTasksBatch` applies a repeated slug to the projected snapshot in turn.
  Decision 4 now judges each entry by its net change from the entry as loaded under the lock, across every step of the write, and replaces "touch" with that rule; Decisions 3 and 7 refer to it.
  Added Acceptance criteria for the duplicate-slug batch (refused), a status-only batch item (passes), and a merge that edits and clears in one write (refused); ledger Q12 and a rejected alternative.
- **F3 (MEDIUM, `abandoned`).** Confirmed: `Prune`'s doc and the store, board and render tests use `abandoned`, and `set-status`/`merge` help use `active`.
  Decision 1 narrows the predicate to the run-status form `<state> · <producer>`, declared once in `boardengine` and composed through it by loom and the claim; Scope, Out, the Auto-mode assumption, ledger Q1 and the Acceptance criteria follow; two rejected alternatives record the broad predicate and the `abandoned` carve-out.
- **F4 (MEDIUM, failed Worktree-Create).** Decision 8 gains a Bound: the claim stays, since the batten run still exists and resumes from that row; abandoning it takes the manual unlock.
  Added an Open risk, an Auto-mode assumption, ledger Q13, a rejected alternative, and an Acceptance criterion.
- **F5 (MEDIUM, unlock unbounded).** Decision 3 states `set-status` is unguarded; Decision 5's unlock arm names `lyx batten status <slug>` as the check; Open risks records that an unchecked clear on a live run shows only in the board's history until loom's next status change re-locks it.
  Ledger Q14, a rejected alternative, and the refusal-message Acceptance criterion follow.
- **F6 (LOW, done or absent entry).** Decision 8 gains a Bound: a done or absent entry is not locked by the claim, an entry created later locks from loom's next status change, and a run on a done entry runs unlocked until `git-robustness`'s refusal lands.
- **F7 (LOW, GitHub ordering).** Decision 6's heading now reads "before any GitHub write", and its body states the issue fetch is a read that stays where it is.
- **F8 (NIT, rejected-alternative trail).** Decision 2's Rationale drops the allowlist comparison and keeps the Seed-Child reason; Decision 8's Rationale states the positive constraint (batten's first status commit follows the row) without arguing against the unchosen option, which stays in `support-log.md`.

Also added `promote` to the refused list in the first Acceptance criterion, since Scope lists it as a mutating path.

## Disputed

None.

## Deferred

None.
