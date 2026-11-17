# Review rounds converge: parsed fixer rebuttals, incremental re-review with round checks, and a Go stalemate escalation

`review-rounds-converge-build` · Task · feature, burler, shed, stencil, fabric, loom, arbiter, cwd · running · Plan-Burler-Review

Decision record: [Sketch](decision-review-rounds-converge-build.md)

Filed from the Sketch `review-rounds-converge`.

Takes: burlerengine's review parser, review gate and Go-written skipped fixer-reports; the review and fix producers and the Bouncer's settle path in shedadapters (settleUnlessRefused, settleUndecided, softCapRule, designWrongKeys, routeDecisions); BouncerConfig's closure seams wired in shedrecipe; fabricengine's private write-tree and diff-tree helpers; discussionparser.AddedDecisionTexts; internal/state's JSON records; arbiterstop's settleable causes
Delivers: fixer-report frontmatter with per-finding dispositions and its fix check; the Go-rendered prior-findings block in prior_rounds; the review file's prior and carries keys judged by a round-check gate entry with D4's incremental bound; a per-round snapshot record and changes file over two general fabric verbs; the stalemate escalation cause counted by Go, settled by the circling verbs and the arbiter, set by the loom key review_stalemate_rounds; lym burler validate-review running the round checks in a run directory; the burler stencils and parent notices stating the new rules
Exports: the fabric working-tree id and tree-diff verbs; the burlerengine fixer-report model and round-context type; the shedadapters round-context builder; EscalationStalemate

The part changes these packages:

- internal/burlerengine (about 1100 changed lines)
- internal/shedadapters (about 1300 changed lines)
- internal/fabricengine (about 200 changed lines)
- internal/fabriccli (about 80 changed lines)
- internal/burlercli (about 120 changed lines)
- internal/shedrecipe (about 40 changed lines)
- internal/loomengine (about 30 changed lines)
- internal/loomcli (about 20 changed lines)
- internal/arbiterstop (about 15 changed lines)
- internal/lymdirs (about 5 changed lines)
- contracts/stencils (about 150 changed lines)
