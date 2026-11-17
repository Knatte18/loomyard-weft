---
format: 5
approved: true
language: go
---

# Plan: review rounds converge

Burler's review rounds stop circling in three ways.
The fixer-report carries a parsed disposition per finding, so a rebuttal reaches the next reviewer beside its finding (D1, D2).
A round after the first checks the earlier round's findings, links each still-open one with `carries`, and raises a new finding only at BLOCKING on what the fixes changed (D3, D4), measured against a per-round snapshot of the review target (D5).
The Bouncer escalates with a named `stalemate` cause once the configured number of judged rounds in a row left the target unchanged (D6).
The cards build bottom-up: fabric verbs, the fixer-report format, the snapshot seam and record, the review-file round checks and prompt block, the context builder and its verb, then the escalation and its wiring.

## Card Index

1 — fabric-worktree-tree — Fabric verbs for a working tree's tree id and the paths changed between two ids
2 — fixer-report-dispositions — Fixer-report dispositions frontmatter, its sole parser and the pass-on-cap fix check
3 — target-snapshot — The target snapshot seam: overlay hash and copy, source tree id, changed paths
4 — round-snapshot-record — The per-round snapshot record the review and fix producers write
5 — review-round-checks — Review-file `prior` and `carries`, and the round-check gate entry
6 — prior-findings-block — The prior-findings block in the review prompt and the reviewer's rules on it
7 — round-context-builder — The exported round-context builder and the incremental round's changes file
8 — validate-review-round-checks — `lym burler validate-review` runs the round checks in a run directory
9 — escalation-notice-way-forward — The parent notice names each cause's way forward
10 — stalemate-escalation — The Bouncer escalates a `stalemate` once unchanged rounds reach the count
11 — stalemate-settlers — The arbiter and the circling verbs settle a stalemate
12 — wiring — Wire `review_stalemate_rounds` and the target snapshot into the review rows

## Shared Decisions

### Decision: nil-seams-keep-today

- **Decision:** Every new seam field (`BurlerHalfDeps.Snapshot`, `BouncerConfig.Snapshot`, `BouncerConfig.StalemateRounds`, `Profile.Round`) has a legal zero value that no caller must fill: a nil `BurlerHalfDeps.Snapshot` writes no target id and no changes file, so every round counts as changed and an incremental round holds the BLOCKING rule alone; a nil `BouncerConfig.Snapshot` or a zero `StalemateRounds` never starts a stalemate escalation; a nil `Profile.Round` is a full round with today's prior-rounds text.
- **Rationale:** Hand-built receivers in existing tests and an in-flight run that crosses the binary change keep working with no seam filled, and review as today until they hold a complete new-format round (Open risks).
- **Applies to:** cards 4, 5, 6, 7, 10, 12

### Decision: pass-on-cap-checks

- **Decision:** The fix check and the round-check entry each have 3 attempts and pass on their cap; the strict parse after the gate runs the shape checks alone.
- **Rationale:** D1 and D3: a frontmatter slip never archives a round or spends a bounce, and a review let through on the cap keeps its findings for the fixer.
- **Applies to:** cards 2, 5
