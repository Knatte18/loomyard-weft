```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's F1–F3 proposals as the fixer applied them.

Per-finding list, each fixed as proposed:

- **F1:** no findings. The new split matches the rubric's "A decision record alone." bullet: the goal points under `## Goal` and the `Added after Discussion` entries count, and the record's other sections are excluded. I built one counter-case: a plan run whose record has no `D<n>` headings. The lens would then list only the `Added after Discussion` entries. The plan's carrying cards are built from the record's decisions (rubric line 19), so such a case is not credible, and the removed fallback is not missed.
- **F2:** no findings. The reviewer's fix inserts `code-scope` after `code-tests` and before `models-rules`, which is the template's order. The doc's sentence, "repeats the template's lenses and appends the new one", is true again.
- **F3:** no findings. The deleted row asserted only static prose. The review states that count, order, shape and the emptied fan are pinned elsewhere. The deletion leaves no other test or caller broken, and the lens-phrase map entry was independent of the other rows.

Possible LOW, not a finding: the summary and lens now say "the board entry under its `## Goal`". The rubric says `## Goal` holds the board entry, so the wording agrees with it.

Tool calls:
1. `grep` of the rubric's "A decision record alone." bullet in `contracts/stencils/loom/loom-rubric-code-review.md`. The context held only the review's paraphrase of it, not the rubric's text.
