---
verdict: BLOCKING
rationale: "Round-1 findings F1-F6 are all fixed in the current plan, but the review missed that card 2's Load contract is false: yaml.v3 v3.0.1 (decode.go:641-645) truncates a float such as 1.5 into an int field without error, and yamlengine.Resolve round-trips the scalar unchanged, so the specified 'non-integer values fail the unmarshal' test fails and D5's non-integer rejection is not carried."
---

# Plan-Review round 1 — judge verdict

## The review's findings

The review raised six findings under an `APPROVED` verdict, and the fixer's report says it fixed all six.
The current plan confirms each fix:

- **F1:** card 5 now creates `cmd/lyx/exitsweep.go` and `cmd/lyx/exitsweep_test.go`, and states why cliwire is ruled out.
  `internal/cliwire/doc.go:26` does fix its production dependency set to the standard library plus `internal/standalonestate`, so the reasoning holds.
- **F2:** card 3 targets `internal/configsync/configsync_test.go` with a `ReconcileAll(dir, true)` seeding test.
  `Result.Applied` exists (`internal/configsync/configsync.go:35`).
- **F3:** card 3's ImpactSummary names both reach points, fabric clone and hubforge hubs among them.
- **F4:** the `lyx --help` integration case now runs with both an absent and an invalid `logger.yaml`.
- **F5:** card 4 states the `SetDurableSinkDir(t.TempDir())` redirect for every case.
- **F6:** card 5 rewords `isLyxWorktree`'s doc comment.

The review carries no `## Focus departures` section, so there is nothing to ratify.
Its focus-directive verdicts check out:
`internal/configsync` has no build-tagged tests, so the verify section covers every targeted package.

## What the review missed

**Card 2's rejection of non-integer values does not hold as specified.**
Card 2 says `Load` "unmarshals both keys into `int` fields" and that "a non-integer value (a float, a string) fails the unmarshal and returns that error".
Its test list requires non-integer values for each key to return an error.
The repo pins `gopkg.in/yaml.v3 v3.0.1`.
Its `decode.go:641-645` decodes a resolved `float64` into an `int` field by truncating it, with no error.
`configengine.LoadOrTemplate` passes the file through `yamlengine.Resolve` (`internal/yamlengine/resolve.go:25-49`), which round-trips it through a `yaml.Node` and leaves the scalar `1.5` untouched.
So `trace_retention_count: 1.5` loads as `1` and passes the positivity check.
The card's own test fails, and the plan does not carry the decision record's D5 ("non-positive or non-integer value") or the acceptance criterion "`logger.yaml` rejects zero, negative, and non-integer values for both keys".
A quoted string does fail the unmarshal, so the defect is limited to floats.

**Fix:** have card 2 check the scalar's type before converting.
For example, unmarshal into `yaml.Node` fields and require the `!!int` tag, or unmarshal into `any` and require an `int`.
Then name a float value such as `1.5` explicitly in the test list.

## Minor, not blocking

Card 5's first integration case says the fixture's `logger.yaml` "sets `trace_retention_count: 2`".
`configengine.load` rejects a file missing any template key (`yamlengine.MissingKeys`).
If the fixture carries only that key, `Load` returns an error and the exit hook falls back to the 200-trace default.
The case would then fail its "exactly two seeded traces survive" assertion.
The card should say the fixture carries both keys.
