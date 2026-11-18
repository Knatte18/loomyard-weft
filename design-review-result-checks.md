# Result checks in Code-Review, loosened-check rulings in Drift, and PR-Review send-backs to Code-Review

`review-result-checks` · Task · enhancement, stencil, drift, fabric, burler, landing, loom, shed, testkit, docs

Decision record: [Sketch](decision-review-result-checks.md)

Depends on: [`records-push-main-weft-ended`](design-records-push-main-weft-ended.md)

Filed from the Sketch `code-review-result-checks`.

Takes: the Sketch's decision record; the review base from loomcli's taskMergeBase; burlerengine.ParseReview; shedengine's reroute history entries; internal/friction
Delivers: the rubric's result-check bullets and review-input reading, a shared Loosened-Check fragment in every committing brief, Drift's per-file loosened-check rulings with their Stuck and verdict lines, the pr-findings verb with its send-back mark, deferred clear and friction note, Code-Bouncer's full generation on a send-back file, and the figures rule in the Describe and darn writer briefs
Exports: the Loosened-Check commit trailer; lym loom pr-findings <file>; the loom.pr_review_small_max key; on_reroute: Code-Bouncer on PR-Review in tapestry, darn and sampler; the ReviewInput seam returning every input written after the settle

The part changes these packages:

- contracts/stencils (about 250 changed lines)
- contracts/recipes (about 10 changed lines)
- internal/driftengine (about 600 changed lines)
- internal/fabricengine (about 200 changed lines)
- internal/loomcli (about 500 changed lines)
- internal/loomshed (about 60 changed lines)
- internal/loomengine (about 40 changed lines)
- internal/shedrecipe (about 80 changed lines)
- internal/shedadapters (about 350 changed lines)
- internal/landingshed (about 400 changed lines)
- internal/pullrequest (about 60 changed lines)
- internal/testkit/envkit (about 20 changed lines)
- docs (about 30 changed lines)
