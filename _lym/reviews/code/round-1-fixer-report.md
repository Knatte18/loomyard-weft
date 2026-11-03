# Fixer-report, code round 1

## Changes

- F1 (decision class): not touched. The parent recorded the `<handle>: <recipe>` status title (creel #528), and the finding is outside the fixer's surface.
- F2: fixed as proposed. `internal/battencli/commanddeps_test.go` builds `commandDeps` over a hub board and asserts `Title()` is `T4: run`, the bare slug for an entry of another slug, and the bare slug with no board; it also fails on a nil `Title`. Commit `1a65695a`.
- F3: fixed as proposed, by accepting and documenting. `internal/orchcli/cli.go` (the `runsWindowTitle` field) and `internal/loomcli/doc.go` (a sentence after the status strand's description) state that the title is read when the strand is added or replaced, so a live strand of the running build keeps it. Commit `993ddf2b`; the `doc.go` clause was then moved into its own sentence after the self-check (see Self-check), committed with this report's round.
- F4: fixed differently. The proposal was a row in the `ensureStatusStrand` tests over a fake reed capturing the `AddSpec`; no such fake exists, because `loomCLI.reed` is the concrete `*reedengine.Engine`. Instead `ensureStatusStrand` builds its spec through a new `(*loomCLI).statusStrandSpec`, and `TestStatusStrandSpec_TitlesTheWindowWithTheRunsHandleAndRecipe` asserts the title is `T4: darn` and the command is the checklist command. Commit `b2dc7de7`.
- F5: fixed as proposed. The start harness's `runsWindowTitle` answers the distinctive `harnessRunsWindowTitle`, and `TestStart_EnsuresTheRunsStrandBeforeTheOrch` asserts the added or replaced spec's `Display.Title` is that value. Commit `2f156dd7`.

Gate: `lym gate test ./internal/loomcli ./internal/battencli ./internal/orchcli` passed (tier0, 0 failed).

## Self-check

- fix-code-correctness, LOW: the F4 test pins the helper's content, not the call from `ensureStatusStrand` — deferred, see below.
- fix-consistency: no findings.
- fix-proposal, LOW: the `doc.go` clause detached "placed right after" from its subject — fixed; the clause is now its own sentence after "and the wait line."
- fix-affected-set: no findings.

No fork returned without a report.

## Disputed

None.

## Deferred

- fix-code-correctness LOW (call from `ensureStatusStrand` to `statusStrandSpec` unpinned): reverting that one call to an inline spec with `c.recipe` still passes. Pinning it needs a seam on `loomCLI.reed`, which is a concrete `*reedengine.Engine`, and a refactor of the reed field is beyond this finding's surface.
