```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. I checked the record against its board entry, `review-setup-tuning`, and looked for decision notes.

**List**

Goal points of the entry:
- twill-discussion: merge `goal-scope` and `board-fit` into one lens that reports only contradictions with the entry or another entry, and keep the other five lenses. **Met** by D1, with one widening (F2).
- review-plan: replace `way-forward` with a repository-invariant lens covering test seams and tier tags, `doc.go` and PATTERN text a card falsifies, and card order; keep `guard-reach` and `affected-set`. **Met** by D2.
- review-code: fold `code-pattern-fit` into `code-affected-set`. **Met** by D3.
- review-code: narrow `code-tests` to missing coverage of changed behaviour. **Met** by D4.
- Profiles: `light` and `docs` run a solo Code reviewer, and `solid` and `wide` keep the fan. **Met in the template** by D5 and D6. Two gaps:
  - `speed` is left with a fanned loom code review and nothing in the record addresses it (F1).
  - On this hub, the live `profiles.yaml` is not reached; the record hands it to the operator (F3).
- Lens location given as "`contracts/stencils`": **handled**. The record corrects it to `internal/lenses/template.yaml` and pins this under Constraints.
- Out of scope (delta review, round counts per profile, new `validate-*` checks): **respected**. No decision touches them.

Other work the entry does not name:
- `review-discussion` dropping the pair: forced by the deletion and backed by the audit.
- The dormant `discussion` fan: forced by the deletion.
- The csharp layer: forced.
- The D5 reclassification: forced and bounded.
- None of these is unlicensed work.

Decision notes:
- `lym board list` shows no `Decision for review-setup-tuning:` note, so none was read and none was unreachable.
- The record has no `Added after Discussion` entries.

**Findings**

- **F1.** LOW · design · `decision-record.md` D6.
  - D6 makes the `light` loom block solo but leaves `speed`'s loom block on the template fan `["review-code", ""]`.
  - So the lightest profile runs a fanned code review and the next one up runs solo, which inverts the ladder.
  - Neither the entry nor D6 says why `speed` keeps the fan.
  - Fix: state the pick. Either `speed` loom goes solo like `light`, or say in D6 why it keeps the fan.
  - Evidence: `internal/runprofile/template.yaml:16-21` (`speed` loom has `code_max_bounces: 1`, no fan key); entry body, "Profiles" line.

- **F2.** LOW · scope · `decision-record.md` D1, the "Reports only…" bullet.
  - The entry says the merged lens reports "only what contradicts the board entry or another board entry".
  - D1 also reports a design conflict with a run or a landed parent commit, which is not a board entry.
  - This carries over `board-fit`'s parent-commit reach without saying the entry's "only" is being widened.
  - Fix: either narrow the bullet to the entry and other entries, or state in D1 that runs and parent commits count as the board for this lens.
  - Evidence: board entry body ("twill-discussion: merge … reports only what contradicts the board entry or another board entry"); `internal/lenses/template.yaml:144-170` (`board-fit` prose that D1 inherits).

- **F3.** LOW · decision · `decision-record.md` Open risks, first bullet.
  - Question for the run's parent: does the entry's line "the Code review is a solo reviewer in the `light` and `docs` profiles" count as met when only the template changes?
  - This hub's live `profiles.yaml` keeps the fan for `light`, and the record leaves the operator to run `lym config profiles --set` by hand after the deploy.
  - No decision note settles this, and the writer cannot write Bolt config.
  - Evidence: `_board/_lym/config/profiles.yaml` (`light` blocks have no fan key); `internal/runprofile/doc.go:15` (seed-only).

Tool calls:
- `lym board list`, to find `Decision for review-setup-tuning:` notes and touching entries; my context held the entry body but not the board list.
