```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens review of the decision record against board entry `review-setup-tuning`.

**List**

- Goal point "twill-discussion: merge `goal-scope` and `board-fit` into one board-and-scope lens that reports only contradictions with the board entry or another entry; keep the other five lenses": met by D1. The fan resolves to the four kept discussion lenses plus `board-scope` and `leanness`, and its prose is report-only.
- Goal point "review-plan: replace `way-forward` with a repository-invariant lens (test seams and tier tags, `doc.go` and PATTERN text a card falsifies, card order); keep `guard-reach` and `affected-set`": met by D2.
- Goal point "review-code: fold `code-pattern-fit` into `code-affected-set`": met by D3.
- Goal point "narrow `code-tests` to missing coverage of changed behaviour": met by D4.
- Goal point "Code review solo in `light` and `docs`; `solid` and `wide` keep the fan": met by D6. D5 is the forced, bounded enabler. The `speed` loom block is an extra pick listed under Auto-mode assumptions.
- Goal point "lens stencils under `contracts/stencils`": met. Support-log Turn 2 and the Constraints record that no stencil names a changed lens.
- Out of scope (delta review, round counts per profile, `validate-*` checks): respected. D6 changes no bounce counts, and Scope's Out list names all three.
- Decision notes: `lym board list` shows no entry titled `Decision for review-setup-tuning:`, so I read none. None are unreachable.

**Findings**

- LOW, consistency, `decision-record.md` D1 and `## Auto-mode assumptions`: D1 also changes the `review-discussion` reviewer fan, dropping `goal-scope` and `board-fit` with no replacement. The entry does not name that fan; it names only twill-discussion. Yet Auto-mode assumptions list only the dormant `discussion` fan pick for D1. Fix: add a pointer line under Auto-mode assumptions for the `review-discussion` pick.
  Evidence: the board entry body names only "twill-discussion" (`lym board get`). `internal/lenses/template.yaml:404-410` shows the pair in `review-discussion`, and `decision-record.md` D1 line 3 drops them.

Tool calls:
1. `lym board list`, grepped for `Decision for review-setup-tuning` titles. My context held only `lym board get` for this task, not the list of notes.
