```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log for content to cut or move (leanness lens).

- **MEDIUM — Decisions D5 / Scope "In" last bullet.** The docs list appears twice: Scope's "Docs in the same commit" bullet names every doc file, and D5 lists the same files again with what each says. Evidence: Scope names `doc.go` ×3, recipe comments, `docs/overview.md`, the orch role stencil, the rubric, the fixer spec and the PATTERN entry; D5 repeats all of them. Fix: keep the per-file content changes in D5 and cut Scope's bullet to "Docs, per D5".
- **MEDIUM — Open risks.** Neither bullet is a question for the run's parent. The first restates D1's bound on conflict sessions under the parent lock. The second says "the operator accepted this in the board entry's design", so it is settled by its own text. Per the rubric, Open risks holds parent questions; these duplicate D1's Bound paragraph. Fix: drop both, or fold the wall-time note into D1's bound as one clause, and leave the section empty. Its heading is mandatory.
- **LOW — Constraints.** Several bullets only say an untouched entry still holds: `PATTERN-sole-parsers`, `PATTERN-stencil-ownership`, and `PATTERN-test-speed` with `PATTERN-test-isolation`. They give the plan writer nothing to act on, since PATTERN is inlined into every agent's prompt anyway. Keep `PATTERN-shed-stop-chain`, `PATTERN-parent-lock`, `PATTERN-refusal-way-forward`, the older-halt rule and the pawl line, and cut the rest.
- **LOW — D1 Rationale.** The second sentence retells the board entry's "Seen" narrative (perch loops, parent moving again). The citation "operator's design (board entry, 2026-10-25)" is enough. The narrative belongs in the support log, and nothing the plan writer needs is lost.
- **LOW — Scope "Out" last bullet and Auto-mode assumption 2.** Both say the arbiter stencil does not change, and D4 says it a third time ("a resume … names the halted row itself"). Keep it once, in the auto-mode assumption, which carries the premise. Cut the Scope item.
- **LOW — Notes for the plan writer, bullet 1.** It repeats symbols D2 already names (`taskMerger`). `mergeParent` and `haltReason` are new pointers, so keep those and trim the repeat. This is a minor duplicate; the section is optional.
- **LOW — Auto-mode assumption 1.** "checked by grep of the tree on 2026-10-28" pins a snapshot, and "no reader outside the code named in Scope" will go stale silently. Since this is an assumption rather than a design rule, it may stay. A leaner form: "the flags and note are read only where Scope names".
- **LOW — Support log, Interview Turn 2.** It lists Q1 to Q6 with their picks, and the Question ledger lists the same picks again (plus Q7). Turn 2 could just point to the ledger. The support log is not the plan writer's input, so this is low.

Nothing in the decision record looked like it belonged outside Discussion entirely. Each relocation above passes the completeness-before-leanness test: every requirement stays in Decisions or Constraints.

Tool calls: none.
