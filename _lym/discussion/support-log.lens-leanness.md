```yaml
lens: leanness
outcome: reported
```

Task: leanness review of the validator-checks decision record and support log, looking for content to move or cut.

- **LOW — Auto-mode assumptions.** This section repeats picks the Decisions already state, and the support log's question ledger already lists every auto-pick.
  Evidence: each bullet restates D2/D4, D5, D8 or D7 almost word for word, for example "Darn runs D1 and D2 but not D3 and D4" against D4's last paragraph.
  Fix: shorten each bullet to the bare assumption plus its id, or keep only the assumptions that carry a reading of the entry (D8's proxy and D10's stencil reading). Today the section only says things twice.

- **LOW — Notes for the plan writer, first bullet.** It repeats D4.
  Evidence: D4 already says "`ValidateLoomDiscussion` gains the worktree root and index, and so the Discussion-Write and Discussion-Burler-Fix gates and the verb on a loom run pass them."
  Fix: keep only the new part ("wired in `internal/loomcli` the way the plan gates get `PlanIndex`"), or fold it into D4.

- **LOW — D1, last sentence.** It overlaps D10.
  Evidence: D1 says "`loom-discussion-spec.md` pins the heading form for loom and darn records alike", and D10 says the discussion spec states D1 to D4.
  Fix: drop it from D1, or keep only the "loom and darn alike" reach, which D2 already implies.

- **LOW — D11.** It restates the repo's docs-in-the-same-commit rule from CLAUDE.md and hedges on which package hosts D6 to D8.
  Evidence: "`internal/planglyph` or `internal/planparser` (whichever hosts D6 to D8)…".
  Fix: D11 needs only the non-obvious items: the help texts of the two verbs, and the `//lym:seam` section in `pattern/PATTERN-test-speed.md`. The generic package-doc list is the plan writer's to derive.

- **LOW — Constraints, `PATTERN-test-speed` bullet.** It restates a PATTERN rule that every test task carries and that the Plan writer gets inlined anyway.
  Evidence: "each new test takes one `tierN` tag, the lowest that fits."
  Fix: cut it.

- **LOW — Rationale lines in D2 and D4 quote audit counts.** "14 gating Discussion findings" and "17 gating Discussion findings" are snapshot tallies.
  Fix: move them to the support log and keep only the theme names. The Goal already points at the report section.

- **LOW (outside leanness) — Acceptance criteria.** They cite `loomshed.NoSuchFunc` and `internal/loomshed#NoSuchFunc` as backticked tokens.
  Evidence: once D4 lands, any re-validation of this record after a goto back to Discussion would refuse its own record with `citation-symbol-missing`.
  Fix: describe the case in words, or use a placeholder with `<…>` so D3/D4 skip it.

No BLOCKING or MEDIUM findings.
The Decisions carry no rejected alternatives (they sit in the support log), and no content needs to move out of the record without first being pulled into a Decision.

Tool calls made: none; my context held both files and the rubric.
