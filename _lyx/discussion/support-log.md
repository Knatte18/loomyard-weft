# Support log — bugfix-webster

## Interview

Autonomous mode: no operator answers; every decision below is an auto-pick recorded in the question ledger.

1. Read the board entry `bugfix-webster` and the related entries `webster-delete-ordering`, `webster-reset-targets`, `card-batching`, `plan-glyph-precision`, `gate-running-visible` and `test-suite-prune-core`.
   The entry's scope limit keeps Webster's own sequencing and the plan-spec change with the `webster-delete-ordering` note.
2. Read `internal/websterengine/sequence.go`: `deriveEdges` adds an edge for a `Uses`/`Targets` match and for a shared target in card order; `SequenceBatches` is a no-op on a plan whose declared order satisfies every edge.
   Conclusion: a Plan-Gate check mirroring the `Uses` edge rule makes Webster run every gated plan in card order without touching sequencing (D1).
3. Checked quarry v0.2.1's `Repo` API: no reference lookup, only TOC, Glyphs, Resolve, Expand and Delta.
   Conclusion: the delete-before-reference check must search the later card's resolved spans textually (D2).
4. Checked `webster-batch-validation-speed`: quarry#34 closed 2026-10-03 and `beea26e62` bumped quarry to v0.2.1; `card-batching` measures the remaining per-batch overhead at 16–18 s in total, so incremental validation is not built; only the model-assertion drop remains (D10).
5. Traced #402 through the weft history: `skills-in-one-turn` failed batch 04 at 09:46, was re-seeded at 09:48, and batches 01, 02 and 04 each got a `report-archived` record before `done`.
   `webster reset` resets only the warp; reports live in the weft under `_lyx/webster/reports/`, so they survived the hand-archived state.json (D5).
6. Traced #404: Publish pushes only the warp task branch (`Fabric.PushBranch` → `PushWarpRebaseFreeAt`), the parent is merged in before the push, and fabric pushes the warp task branch mid-run (origin's `test-suite-prune-core` equals its local HEAD).
   A reset to start therefore leaves the remote holding abandoned commits; the fix belongs in reset (D7), and Publish only needs a truthful halt (D8).
7. Read `rebaseline.go`: any changed hash of a begun card refuses, whatever the batch's state (D6).
8. Read `batchfail.go` and `recoverbatch.go`: every batch failure names `recover-batch`, whose recovery strand faces the same card (D4).
9. Read `triage.go` and shuttle's wait: after `background_shell_wait_min` the turn end is counted and the shell is left alone; the note says only that the shell "may still be running" (D9).

## Rejected alternatives

- D1, a rule narrower than `deriveEdges` (ignoring **Delete:** targets on the later card): Webster's `SequenceBatches` would still reorder such a plan, so Webster would leave card order; rejected while sequencing stays.
- D1, a separate cycle check: every cycle needs a backward edge, so it would never fire on its own.
- D1, removing `SequenceBatches` now: excluded by the board's scope limit.
- D2, a quarry reference API: a quarry feature, outside a bugfix task.
- D2, informational severity: the `pair-lifecycle` and #401 runs both blocked at Webster; only a blocking finding keeps such a plan out.
- D3, accepting a missing Delete target only when git history shows an earlier batch's commit deleted it: more machinery for the same outcome, since in a run only batches and parent merges change the tree and either way the card's wanted state holds.
- D3, flagging a batch that deletes a file outside its own card: a new strictness on forks that could fail batches that are correct.
- D4, keeping `recover-batch` as the only way forward: it loops (#401).
- D5, having `record-batch` adopt the orphan report: the report has no begin record and no attributable fork, so it cannot be audited.
- D6, accepting an edit to a begun non-terminal batch: its fork may be in flight on the old card.
- D6, resetting only that batch's record: a new partial-reset mechanic; restamping the hash of a failed batch is enough for `recover-batch`.
- D7, Publish refetching and retrying the push: the parent is already merged in, so a retry meets the same diverged task branch.
- D7, Publish force-pushing on rejection: Publish cannot tell the run's abandoned commits from someone else's; reset can, from the pre-reset HEAD.
- D9, waiting until the shell exits: can hang a run forever.
- D9, killing the shell: needs a provider-specific stop mechanism shuttle lacks.
- D9, escalating: turns finished runs into stuck ones over a shell that may be harmless.
- Incremental per-batch validation: the measured cost after the quarry bump does not need it (YAGNI).

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Does the Plan-Gate check need to match `deriveEdges` exactly? — Yes, D1; anything narrower lets Webster reorder a gated plan.
- Q2 (auto-pick): How can the gate see "a symbol still referenced from a member a later card edits" without a quarry reference API? — Whole-word, Go-qualified search inside the later card's resolved `Edit:` spans, blocking, D2.
- Q3 (auto-pick): Option for `webster-delete-already-gone`? — Missing Delete targets are advisory at dispatch only, D3.
- Q4 (auto-pick): What replaces `recover-batch` as the way forward for a delete blocked by a later reference? — Plan edit, `rebaseline --card`, then `recover-batch`; `recover-batch` refuses before spawning while the cause holds, D4.
- Q5 (auto-pick): Adopt or archive an orphan report at begin-batch? — Archive, D5.
- Q6 (auto-pick): Widen `rebaseline --card` or only explain the refusal? — Widen for failed or dead batches only, D6.
- Q7 (auto-pick): Where does #404 get fixed? — At `webster reset` (D7), with a truthful Publish halt (D8).
- Q8 (auto-pick): Intended behavior once `background_shell_wait_min` elapses? — Count the turn end, leave the shell, state the outcome, D9.
- Q9 (auto-pick): Is incremental validation in scope? — No; the quarry bump already landed, D10 is the only remaining change.
- Q10 (auto-pick): Does the plan spec get the new checks despite the scope limit? — Only their rows in the validation-check list; the prose rewrite stays with the note.
- Q11 (open, for the plan): What ends Master's session after Webster finishes, for D9's note text.
