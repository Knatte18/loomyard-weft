---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: crucible/review-prompt-template.md:122
    summary: the template still tells the instantiator to find live tests by their go:build tier2 or tier3 tag, which no test file carries any more
    fix: "In crucible/review-prompt-template.md lines 122, 129, 132 and 203, replace the `//go:build tier2`, `//go:build tier3` wording with the run-time form: a live test is one that opens with `tiergate.Require(t, 2)` or `tiergate.Require(t, 3)`, found with grep for `tiergate.Require(t, 3)`. In line 203, a new live test is written with that Require call as its first statement instead of a build tag."
---

### [LOW] Review template still locates live tests by build tag

**Location:** crucible/review-prompt-template.md:122
**Issue:** The commit moved the tier from `//go:build tierN` lines to `tiergate.Require(t, N)` calls, and round 1's F1 fix updated the smoke commands in this template.
Lines 122, 129, 132 and 203 still direct the instantiator to list "every `//go:build tier3` test function" and to write a new live test under a `//go:build tier2` tag.
A grep for that tag finds no test file, so the cost declaration that guards against spawning real LLM sessions would list nothing, and the EXECUTION BAN would be written from an empty list.
The three directive targets themselves are sound; this is the only stale spot found around them.

## Verdicts on the focus directive

- `tmuxAllowed` (internal/testkit/tmuxkit/tmuxallowed.go): reads `tiergate.Selected()`, so it follows `LYM_TEST_TIER`.
  `TestTmuxAllowed_FollowsTheTierSelection` pins unset, single tiers 0 to 3, the ranges `0-2` and `1-3`, and the malformed value `tier2`.
  The test sets the variable with `t.Setenv`, so it is correctly not parallel and says why.
  Not a defect.
- crucible smoke commands: the orchestrator prompt now sets `LYM_TEST_TIER=2` for the serial smoke and `export LYM_TEST_TIER=2` before the compiled-binary loop, with the LLM-driving case at tier 3.
  The README matches.
  No `-tags tier2` command is left, so no smoke test skips green.
  Not a defect.
- battencli TestMain: with `LYM_TEST_TIER` unset or `0`, `Selected` returns `{0,0}`, so the refusing seam is installed and a real pair push cannot run.
  Under a range such as `0-2` the real seam is installed for the whole binary, which the comment names as the tier0-alone bound.
  No test pins the refusal, which is a harness guard and not a behavior of the package; I raise nothing.
  Not a defect.
