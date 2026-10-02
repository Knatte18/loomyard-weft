# Card 15 — self-contained-focus

**Prosa:**
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `contracts/stencils/burler/burler-focus-directive.md`

**Edit:**
- `contracts/stencils/focusdirective_test.go`

**Intent:**
Stop a Burler focus directive from forcing the reviewer to read prior-round files before its own review is saved (D13), keeping each round an independent judgment.

- The judge template's "What a `focus` entry may say" list gains a rule: an entry restates every site, commit and claim it depends on, and never refers the reviewer to a prior round's review, fixer report, or finding ID.
- `burler-focus-directive.md` states that the directive never licenses reading prior-round files before the review is saved; a directive that points into one is followed only as far as its own text goes, and the gap goes under `## Focus departures`.

Neither file gains or loses a marker, so the stencil tests on their marker sets keep passing.

Tests in `focusdirective_test.go`: the judge template carries the self-contained rule (restates sites and never cites a prior round's finding ID), and the focus directive carries the prior-round rule and names `## Focus departures` for the gap.

**ImpactSummary:** stencil-test assertions only; the two stencils change what the judge and the Burler round read, and no Go code reads their text.
