# Plan-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed.
`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.
The overview's `approved:` key is untouched.

## Changes

- **F1** — `02-audit-ledger.md` now also edits `beginbatch.go`, `recoverbatch.go` and `beginbatch_test.go`.
  `BeginBatch` and `recoverSpawn` copy the prior record's `AuditWarnings` onto the new record, beside `StartSHA`, and `TestBeginBatch_ReBeginKeepsAuditWarnings` pins it.
  The ImpactSummary names the extra copied field.
- **F2** — `08-record-batch-audit-dispositions.md` drops the vacuous second-call-on-a-terminal-batch assertion.
  It adds the reachable retry: a no-report call with a parent `fabric-reference`, then a re-begin and an OK report, recording one warning, plus a failing-verify variant.
- **F3** — `15-run-exit-dispositions.md`'s named-spawn end-to-end test asserts that `summary.md`'s `## Audit warnings` holds batch 2's warning exactly once.
- **F4** — card 8's step 3 grounds the no-report and FAILED-report extension in D2 and D8 and explains why a late report re-forms no ring.
  Tests now cover a correctness finding on a no-report call and on a FAILED report.
  `11-recover-failed-batch.md` takes the late-report half: an OK report landing after the failure is archived by `RecoverSpawnOrAttach`, which spawns.
- **F5** — the new `27-loom-gate-refusal-sweep.md` maps `fabric-sync` to `lyx fabric checkout`.
  `worktree-clean` now names git for the warp and `lyx fabric commit` for the weft, and `reconcile` is kept for `fabric-ready` and `junction`.
  The tests exercise the branch-mismatch and the separate warp and weft dirty cases.
- **F6** — `24-webstercli-refusal-sweep.md` says `fabricSync` only commits.
  Its fixture now fails the weft commit through a stale `index.lock`, and the way forward is `lyx fabric commit` or the next bracket verb.
- **F7** — the new `26-loom-cli-refusal-sweep.md` adds `commit-records`' commit and push failures, with ways forward and tests through `commitStatusDeps`.
  It groups approve's lookup failures and commit-records' probe and park-marker failures as raw I/O, and says the `validate-*` verdicts are not refusals.
  `28-refusal-spec.md` states the same for the table.
- **F8** — card 27 groups the `CheckSeed` verdicts that `seed.go` documents as pre-empted by step 1, plus `seed-unreadable` and the other unreachable coherence branches.
  It keeps a way forward and a test only for the reachable out-of-vocabulary-outcome branch, and states that `CheckHalfFinished` is card 22's.
- **F9** — `22-loom-goto-coherence.md` gains a `Prosa` target `contracts/specs/loom-status-spec.md`: `goto` joins the history-outcome rule, and the budget reason is described as a prefix.
  Card 22 now uses `internal/shedengine/run.go`, and its ImpactSummary mentions the spec.
  `17-shed-goto-outcome.md` updates `appendHistory`'s vocabulary comment.
- **F10** — card 8's step 7 also runs the verify re-run when the batch record already carries `AuditWarnings` from an earlier no-report call.
  A failure names those warnings among the reasons.
- **F11** — `01-audit-severity.md` derives the run's `_lyx` directory as `filepath.Dir(geom.WebsterDir)` and says why.
  It states that `CheckParent` emits one `named-spawn` violation per spawn, and its ImpactSummary notes the changed violation count.
- **F12** — `07-batch-fail-primitive.md` moves `state.go` from Uses to Edit to update the `BatchState.Terminal` and `BatchState.Status` comments.
  Its ImpactSummary now states exactly how each reader treats `failed`.
- **F13** — `23-websterengine-refusal-sweep.md` puts the unknown-batch way forward in `findBatch`, naming every verb that reaches it.
- **F14** — `04-batch-card-record.md` edits `recoverbatch_test.go` and adds `TestRecoverSpawn_RecordsCardSet`.
  The deferral to card 11 is dropped, along with card 11's duplicate `Cards` assertion.
- **F15** — card 26 is split into `26-loom-cli-refusal-sweep` (approve, start, commit-records) and `27-loom-gate-refusal-sweep` (seed, preflight, batchifier, commit seams, Webster adapter).
  The documentation cards are renumbered 28–31 (`refusal-spec`, `package-docs`, `ly-drive-skill`, `constraints-invariant`), with their content unchanged apart from the F7 sentence in card 28 and its `cards 5–27` range.
  The overview's prose ranges, Card Index and `way-forward-phrasing` Applies-to (`22–27`) are updated, and so is card 23's pointer to the refusal table (card 28).

## Deferred

None.
