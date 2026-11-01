# Round 2 fixer-report — plan

## Changes

- **F1 (MEDIUM, consistency) — fixed as proposed.**
  The premise holds: the decision record's entry "No _board literal stays in livestate_doc_test.go" rules the quoted error text reworded and the guard unexempted, while card 4 left the file as it was and card 8 exempted it.
  - `_lym/plan/04-fabric-test-bolt-dir-sweep.md`: added a `**Prosa:**` group before `**Uses:**` listing `internal/fabricengine/livestate_doc_test.go`, and replaced the "left as it is" bullet with one saying the quoted error text at line 189 is reworded with the Bolt directory name elided ("no <Bolt directory> entry and no weft sibling"), so card 8's guard needs no exemption for it.
  - `_lym/plan/08-stencil-skill-bolt-dir-sweep.md`: dropped `internal/fabricengine/livestate_doc_test.go` from the guard's exemptions; the exemptions are now `internal/lymcwd/anchor.go` and `internal/lymcwd/enforcement_test.go`.
  - `lym loom validate-plan --overlay` with `{}` after the edits: `ok: true`, informational `create-new-unit` findings only.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings.
- fix-proposal: no findings.
  Its side note outside its lens, a `_board` comment at `internal/fabricengine/livestate_refusal_selftest_test.go:307` that card 4 must list, is disputed with ground `premise false`: the comment sits inside `TestRefusedBefore` (its func line is 238), which card 4 lists as `internal/fabricengine_test#TestRefusedBefore`, and card 4's comment bullet renames it.

## Disputed

None.

## Deferred

None.
