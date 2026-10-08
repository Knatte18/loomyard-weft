# webster: accept short head_sha, fix resume attribution, overview frame

Webster's `record-batch`, `recover-batch`, `reset` and `rebaseline` no longer force a re-drive on three observed paths, and `tools/tokencount` can re-measure the Merriam fixed context.

## What changes

- **Abbreviated `head_sha`.**
  A new helper in `internal/websterengine` resolves a report's `head_sha` to the one commit it names.
  It runs where the report is consumed: `record-batch`, `recover-batch` and `reset --to report-head`.
  A 40-hex value passes unchanged; a 4 to 39 character hex prefix resolves through `git rev-parse --disambiguate`, keeping commits only.
  The full SHA then feeds the merge-only walk, the digest's `HeadSHA` and `CardSHAs`, and reset's target, so a stored run record never holds an abbreviation.
  A prefix naming no commit or several is refused with a new row in `refusal-spec.md` that tells the fork to rewrite `head_sha` as `git rev-parse HEAD`.
  A non-hex value is left as given and meets the existing mismatch or missing-commit refusal.
- **Stopped-then-resumed fork.**
  Each fork batch's state now records the transcripts attributed during its current bracket.
  `record-batch` re-audits them in full with any new ones, and they count toward attribution, so a report written by a transcript an earlier call consumed is recorded instead of archived as unattributable.
  `begin-batch` and `recover-batch` clear the list whenever they write the batch's record.
- **Missing overview frame hash.**
  A state without `PlanOverviewFrameHash` derives the recorded overview frame from the stored plan baseline copy.
  `rebaseline` and the plan-drift advice use it, so a Card-Index-only change to `00-overview.md` is accepted with follow-up cards named by `--card`, and the hash is recorded.
  With the baseline copy absent, `rebaseline` refuses as before.
- **Merriam calibration.**
  `tools/tokencount` measures a Merriam session's start at the message holding the prompt Read's result, not at the skill-load turn.
  The calibration report gains a Merriam fixed-context section: line count, derived fixed figure, median and the current `merriamFixedContext`.
  A second Merriam session of a run is listed among the skips.
  The reconstructed Merriam base is brought to the texts `MerriamBase` counts, using the directive fill, now exported from `internal/pattern`.

Closes #456
Closes #457
Closes #463

The first half of #463, the plan gate's done-card scoping, landed earlier; this change is the rebaseline half.

## Design decisions

- Only object names are matched for `head_sha`, never refs, so `HEAD` or a branch is never accepted as a head.
  The resolved commit still meets the checks a full SHA meets at its site.
- A correctness finding wins over an unresolvable `head_sha`: the batch fails on its finding and the failed digest records an empty head, not the new refusal.
- A re-begin of a non-terminal batch clears the bracket's own transcript list, since every begin-batch launches a new fork.
- `merriamFixedContext` stays a constant; it is not moved into config.
- Out of scope: a route from Plan-Bouncer back to Webster, and recovering a report a pre-fix binary already archived.

## To check by hand

- Finding identity (`findingIdentity`, the violation `Key`) must be stable across re-reads of a grown transcript, or a finding repeats on the second `record-batch` of a bracket.
- `d1-head-sha-resolution-point-vs-correctness-path`: confirm the resolve point in `RecordBatch` leaves the correctness-failure path with the full SHA for an abbreviation of HEAD and an empty head for an unresolvable value.
- `d4-subtracted-texts-differ-from-merriambase`: confirm the texts the calibration subtracts from the fixture's line count match those `MerriamBase` counts.
- Run `go test ./...` and `go test -tags integration ./...`; neither was re-run after the final fixes in this description's step.
- Run `lyx tokencount` calibration over a real run and compare the Merriam fixed figure with `merriamFixedContext`; the fixtures were not checked against a live run.
- `cmd/lyx/testdata/test-budget.yaml` changed for the new tests' count budget.
- The `merriamFixedContext` comment in `internal/websterengine/merriambase.go` changed; no value changed.
