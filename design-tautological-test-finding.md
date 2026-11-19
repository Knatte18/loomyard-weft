# Code review flags tests whose expected value is computed the way the code computes it

`tautological-test-finding` · Note · enhancement, burler, stencil

Source: `docs/research/survey/pocock-skills-2026-11-19/report.md` (survey of Matt Pocock's skills repository, 2026-11-19), candidate 3, which holds the design.

## Idea

The Code-Review rubric's Test economy list gains a bullet: a test whose expected value calls the code under test, a shared helper or the same formula is a finding naming the test and the independent value it should assert.
`scribe:testing` gains the matching rule. A mutation survivor on a line whose covering test recomputes its value is evidence for the finding.