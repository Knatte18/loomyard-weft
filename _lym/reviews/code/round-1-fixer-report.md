---
dispositions:
  - id: F1
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The review proposed a fallback to the lowest declared tier whose command holds {args}, and a measuring step that asks the parent when none exists. The parent's ruling on F2 drops every --tier fallback, so MeasureCall returns a refusal instead: Gate's own, one for a card-scope key and one for a key that lists a tier whose command holds no {args}. The target word, the doc comments and the TierUse/MeasureCall rows follow."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The review asked the wiring test to build a table from a gate.yaml lacking the darn-write key and assert the call MeasureCall renders. A hub file that lacks a template key gets it filled at load, and the parent's ruling makes a refused key fail the table build. The new subtests instead assert the literal call for a targets tier and a no-targets tier, and the refusal for a darn-write key that lists an undeclared tier."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Fixer report, round 1

## What changed

Commits `ef86b850`, `7ac7dd2b` and the self-check commit that follows them group the findings by package, since several findings share files.

- F1: the new tier setting `results: trx` (`gateslot.Tier.Results`, `gateslot.ResultsTRX`) is the opt-in.
  Load accepts it only on a `vstest` tier whose command holds `{args}`.
  `runInvocations` injects the results words only for such a tier, so a tier whose `{args}` reach another program keeps its meaning and no reconcile reinterpretation is needed.
  The green fingerprint hashes the setting when set.
  Updated: the `docs/dotnet-hub.md` Tiers and timing-tier sections and the tier0 example, the gatecli `doc.go` paragraph, the `runInvocations` doc, the gateslot `doc.go` and `template.yaml`, and `_lym/landing/summary.md`, whose Merge danger names the new key and the darn prompt's gate.yaml read, and whose Open risks follow.
- F3: both shell-spawning tests moved to `internal/gatecli/tierrun_spawn_test.go`, `//go:build !windows`, with `tiergate.Require(t, 1)` and no Windows skip.
- F4: `Config.MeasureCall` returns `(string, error)`.
  It refuses what Gate refuses, a key of scope `card`, and a key listing a tier whose command holds no `{args}`.
  Rows for a no-`{args}` tier, a card-scope key, no tier0, an undeclared tier and a key mixing a targets tier with one without joined the `MeasureCall` table.
- F5, F6: `envOverrideLine` drops the role clause and the session's value.
  `TestRunTierNamesEnvThatDiffersFromTheSession` asserts the new line and that the session's value is in neither stderr nor the log.
- F7: the results-file table gained rows for two invocations that disagree with their console, the Error, Timeout, Aborted and NotExecuted outcomes, a missing and a malformed file, a passing run whose file holds a run message, and the gate's words reaching the command ahead of the call's own.
- F8: the env-override integration case sets a variable the tier overrides in the gate's own environment, asserts the line in the gate output and the tier log, and asserts no line for an equal value.
- F9: see the dispositions.
- F10: `resultsArgsRefusal` refuses a call's own `--results-directory` or a `trx` logger, in the plain, joined and `-l` spellings, in `planTierRuns` before the tier takes slots and in `runInvocations`.
- F11: `trxFile` decodes each `RunInfo`'s outcome, and only `Error` messages become causes.
- F12: `runInvocations` flushes the counter before it reads the results file.
- F13: `doc.go` and the summary say a tier without `results` gets no crash cause.
- F14: the env line is computed and printed once in `runTierChecked` and heads the result's log; on Windows variable names compare in upper case.

## Disputed

Nothing.

## Deferred

Nothing.

## Rulings

- F2, the parent's ruling "Drop the MeasureCall tier fallback" (decision record, added entry): `Config.MeasureCall` returns Gate's refusal as an error and keeps no `--tier` fallback; the darn prompt render fails with it; no PATTERN change.
  Carried out in full: `MeasureCall` has no fallback, `darnwiring.go` returns the error, and the gateslot docs and the summary describe it.
  The darn table build therefore also fails for a hub with no `gate.yaml`, which commit `2b395b65` had kept buildable.
  `TestWire_LanguageSkills` pinned that old behaviour, so its no-gate.yaml row now expects the build to fail with "gate config absent".
  I did not flag the ruling to the parent: it names this consequence in its rationale.

## Self-check

- fix-code-correctness, MEDIUM: `-r` refused as a results directory. Fixed: dropped from `conflictingResultsWord`, with a row that lets `-r linux-x64` through.
- fix-proposal, MEDIUM: the same `-r` finding. Merged with the one above.
- fix-code-correctness, LOW: the env line prints once per attempt, so a rerun prints it twice. Disputed, `every fix worse`: a once-per-call print must move out of `runTierChecked` (`internal/gatecli/tierrun.go`), and each attempt's log, written per attempt at `internal/gatecli/run.go:631`, still needs the line, which splits the line's ownership across two files for a rerun that itself starts the tier again.
- fix-proposal, LOW: the same rerun finding. Merged with the one above.
- fix-code-correctness, LOW: no test pinned the flush before the results replace the counts. Fixed: a row whose last console line has no newline.
- fix-code-correctness, LOW: the joined and short logger and results-directory spellings were untested. Fixed: rows for `--logger:trx`, `--results-directory=...` and `-l trx`.
- fix-proposal, LOW: the refusal came after the slots and the shared build. Fixed with my own change: `planTierRuns` refuses the same words through the shared `resultsArgsRefusal`, with a row in the plan refusals; `runInvocations` keeps the check for direct callers.
- fix-proposal, LOW: nothing warned that a `darn-write` tier needs `{args}`. Fixed: a line in the gate template's key comments.
- fix-consistency: no findings.
- fix-affected-set: no findings.

Gate repair: the gate flagged a wrapped comment line in `internal/gatecli/doc.go` and another in `internal/gatecli/tierrun.go`, which no test had run on.
Both comments now break at sentence boundaries, one sentence per line, with none at 120 characters or more.

Reach repair: the gate found no mapped test reaching the TRX reading and counter-replacement code, which only tier1 tests exercised.
`internal/gatecli/trx_test.go` adds tier0 tests for `readTRXResults`, the vstest counter's start and replace of an invocation, `resultsFileLines` and `removeResultsDir`, with no process spawned.

The disposition of F1 stays `fixed` although the preferred route was taken as written; F4 and F9 are `fixed-differently` for the reasons in the frontmatter.
