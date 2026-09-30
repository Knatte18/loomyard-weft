# Webster verify: triage a red integration suite before halting

A red plan-level `## verify:` no longer halts a webster run by itself.
The integration stage now triages the failure and halts only when the task broke a test.

- A failure that goes green on one rerun at head is recorded as **flaky**, and the run stays `done`.
- A failure that also fails at the baseline commit is recorded as **pre-existing**, and the run stays `done` with a warning.
- A failure that passes, is absent, or never ran at the baseline is a **regression**.
  The run is demoted to `stuck`, with a `StuckReason` naming the regressing tests and the bisect-localized card.

Motivated by `ide-hub-workspace` blocking twice on the flaky `TestRemove_AddSucceedsAfterWarpBranchDeleted` in a package the task did not touch.

`_lyx/webster/reports/integration.yaml` now carries `failures` (id, kind, output tail capped at 40 lines) and a `triage` block (verdict, rerun result, baseline SHA, flaky, pre-existing and regression lists), so neither the driver nor the hub has to read the fork transcript.

## What changes in the code

- `internal/websterengine/verifyparse.go` parses `go test` output into identities: `<package>.<TopLevelTest>` (subtests collapse), `<package>` for build, setup or init failures, and one opaque identity when the command fails without any parsed Go failure.
- `triage.go` runs in `runIntegrationStage`'s unleased middle phase: rerun the whole verify command once at head, then check out the baseline detached, run it, and restore the branch.
  Report, summary, state and friction writes happen under the lease in the final phase, as before.
- Bisect now localizes by the regressing identities instead of the command's exit code.
  With a pre-existing failure present at every SHA, the exit-code predicate blamed card 1 by arithmetic.
  An opaque regression keeps the exit-code predicate.
- The integration fork stencil redirects the verify output to a told log under the `.lyx` scratch dir, and Go parses it.
  A missing log is not an error; identities then come from the rerun.
- The Master stencil's `FAILED` branch ends `outcome: done`, and the integration verdict belongs to webster.
  The old loud error for `done` over a `FAILED` suite is gone; `done` with no integration report still fails loud.
- Recording: a non-regression verdict adds one `RunResult` warning per category, an `## Integration suite triage` section in `summary.md`, a `webster-verify-triage` friction note (only when the friction dir is set), and a log warning.
  A regression extends the existing `## Integration suite failed` section with identities and tails.
- The `Describe` stencil (`landing-template-describe`) now carries the triage section into the change description as a check item.
- `begin-batch` keeps a batch's recorded `StartSHA` across a re-begin.
  The baseline is the HEAD recorded before the first batch began, and a re-begin would otherwise move it past commits an earlier fork landed.
- Docs: `internal/websterengine` package doc, `docs/overview.md` webster row, and `contracts/specs/webster-spec.md`.

## Design decisions to review

- Go never rewrites Master's `outcome.yaml`.
  After a regression demotion it still says `done`; the report's `triage.verdict`, the `-1` state record and the `summary.md` section are the record.
  Demoting `done` to `stuck` is safe because a `done` run already has a validated `summary.md`; promoting `stuck` to `done` would need both added.
- Triage also runs when Master ended `stuck` over a `FAILED` report.
  A non-regression verdict never promotes it; the outcome stays `stuck`.
- Every ambiguous case halts.
  A failure is excused only when shown to fail at the baseline, and an opaque failure is never pre-existing.
- With no bisector or no recorded start SHA, a red rerun is a regression with "not localized" and "baseline unavailable" warnings.
- The baseline is the recorded pre-batch HEAD, not a parent of a card SHA, because a batch can hold several commits.

## Known limits

- A regression in a test that is also flaky can pass the single rerun and be recorded flaky.
- A baseline whose `&&` chain stops early hides later pre-existing failures, so they halt as before.
- A regressing test that did not run at an intermediate bisect SHA counts as passing there, so bisect can blame a later card.
- Two extra whole-verify runs add wall-clock to every red integration stage.
- Unusual output (`-json`, custom runners) may parse only as package or opaque identities, which degrades toward halting.

## To check by hand

- The tagged `integration` tests that spawn a real shell and git (baseline checkout and restore, bisect by identity) were not run in this description step; run them.
- The stencil board copies have drifted from the worktree sources, so a production deploy (`update-plugins.sh`) is needed before the new stencils apply in production.
