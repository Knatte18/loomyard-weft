# Card 6 — judge-reads-facts

**Edit:**
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncerprompt_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `contracts/stencils/claims_test.go`
- `CONSTRAINTS.md`

**Uses:**
- `internal/shedadapters/bouncerfacts.go`

**Intent:** Turn the judge from a second reviewer into a sit-rep agent: it reads the facts file, the latest review and the previous ledger, and decides by the D6 rule (D2, D6).

`judgeCall` calls `writeRoundFacts` for round N after reading the report and before filling the prompt, on every judge call, including one that ends up attaching to a live judge, since the render is deterministic.
A write failure degrades like an unreadable judge template, because the prompt would name a file that does not exist.
`judgeValues` drops `artifacts` and gains `facts_path`;
`report_path` still names the latest review.
`ArtifactPaths` stays on `BouncerConfig` for the seed pass alone; update its field doc to say so.

`bouncer-template-judge.md` is rewritten, header comment included:

- Role: a review-loop judge that reports whether the loop has converged from fresh reviews, never a reviewer of the artifacts.
- Reading order: the facts file, the latest review, the previous ledger (`(none)` on round 1).
  No artifact path, no fixer report and no instruction to check fixes against artifacts.
- The rubric stays embedded, scoped to writing the focus file and to reading what its classes mean, and is no mandate to re-review.
- Fixed findings never count as convergence, since a fix is new text no fresh reviewer has seen;
  the review's own top-level `verdict:` is no convergence signal.
- The D6 decision rule as written in the decision record: `CONTINUE` while the latest round carries a gating-class finding at MEDIUM or worse or any BLOCKING finding;
  `CONVERGED` when it carries neither;
  `CIRCLING` when such findings remain and the facts show no progress across at least two fresh rounds, a parse-error row being no evidence;
  a parse-error row for the latest round means `CONTINUE`.
- The departure bullet: the judge may relabel one finding's effective class or severity in either direction only where that finding's own review entry shows the mislabel, never relabels a BLOCKING finding downward, and names every departed finding's ID in the rationale.
- The focus-departures ratification, the three output files, the ledger carry-forward rule and the focus-entry rules stay.

Tests:
`bouncer_judge_test.go` — the rendered judge prompt names the facts path, the review path and the previous ledger path, and contains neither any `ArtifactPaths` entry nor the round's fixer-report path;
the facts file exists for round N after a judge call;
a judge call whose round-1 review lacks `class` still spawns the judge;
the rendered prompt carries the parse-error-means-`CONTINUE` sentence and the never-relabel-BLOCKING-downward sentence.
`bouncerprompt_test.go` and `bouncer_seed_test.go` fill the judge stencil with the new marker set.
`claims_test.go` — the judge stencil claims the facts reading list, the decision rule, the departure bound and the fixed-findings rule, and wantNone `{{.artifacts}}`.

`CONSTRAINTS.md`'s Review Round Invariant gains the D9 line: a review segment converges only on a judge verdict over a fresh review, and fixed findings never converge on their own.

**ImpactSummary:** Changes what every Bouncer judge pass reads and how it decides, in all three loom review segments; the seed pass and its prompt are untouched.

**Commit:** 6: bouncer: the judge decides from round facts and the latest review
