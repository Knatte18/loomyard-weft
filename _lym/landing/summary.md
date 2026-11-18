# Result checks in Code-Review, Drift and PR-Review send-backs

Code-Review let result-level issues through to PR-Review: test limits widened with no measured cause, departures kept on obsolete measurements, set-up differences described as agreement, and improvement claims with no before and after figures.
This change closes the gap in three places, in the tapestry, darn and sampler recipes, the ones with a Code-Review segment.

Closes #139

## What changes

- **Code-Review rubric.**
  `contracts/stencils/loom/loom-rubric-code-review.md` gains three generic bullets under `## Also flag`:
  loosened checks, tests that no longer check what their name says, and agreement or improvement claims without figures.
  Its `## Review input` section also reads each input the review-input note names.
  Every recipe's Code-Bouncer and Code-Burler-Review rows read this one rubric.
- **`Loosened-Check:` commit trailer.**
  The form is `Loosened-Check: <test> — <what was loosened> — <measured cause>`, one per check, on any commit of the review range.
  One shared stencil fragment states it, and every brief of a session that commits code in these recipes includes it.
  Where Go writes a role's commit, it appends the trailer lines the role's handoff carries.
  `ReadForEngine` in `internal/stencilstore` now resolves plain fragment includes for this.
- **Drift rules on loosened checks.**
  Drift-Verify and Drift-Recheck give the judge each file that exists at the review base and that the change touches, with its diff, and the range's commit messages.
  `internal/fabricengine` supplies both, so `internal/driftengine` runs no git.
  The judge rules each file `none`, or per loosened check `caused` (a trailer names a measured cause) or `uncaused`, each with a reason.
  An `uncaused` ruling makes the row Stuck, and the way forward is a trailer commit or a revert, then `lym loom resume`.
  Rulings are stored keyed by a hash of the file's diff and the range's trailer lines, so Drift-Recheck spawns no judge for a file already ruled.
  The verdict's regression section lists each loosened check with its ruling and cause.
  Judgement records written before this change still read.
- **PR-Review send-backs.**
  New verb `lym loom pr-findings <file>` takes a findings file with its verdict line.
  It sends the run back to Code-Bouncer when a finding is BLOCKING or the findings plus the fix-here count exceed `loom.pr_review_small_max`.
  Otherwise it answers fix-here and adds the findings to the count.
  The send-back copies the file to the loom scratch directory as `pr-findings-<n>.md`, writes a friction note recording Code-Review's miss, and marks the pull-request record.
  The tapestry, darn and sampler recipes give PR-Review `on_reroute: Code-Bouncer`.
  The re-entered Code-Bouncer starts a full generation on a send-back file, and the reviewer's prompt names the file in its own review-input note.
  The PR-Review brief tells the agent to run the verb on every findings set before fixing any of it.
- **`loom.pr_review_small_max`.**
  A new integer loom key, default 3, which a task's `run_settings` may override.
- **Figures rule.**
  The Describe brief and the darn writer's description rule say a claim of agreement or improvement carries before and after figures from the current code, and is left out otherwise.

## Design decisions

- The trailer is the cause's record because every writer commits before Drift-Verify and Code-Review, and no change description exists yet at Code-Review in tapestry.
  The cause is the committer's attestation: nothing re-measures it, and the regression section shows every loosened check in the pull request body.
- Only files existing at the review base are ruled, so a check loosened in a test the same change adds is left to the rubric.
  The judge decides what counts as a test and as loosened; no language or path pattern is hard-coded.
  Writers' own drift gates run without the judge, so an uncaused loosening first shows at Drift-Verify.
- The send-back falls back to fix-here when the recipe's PR-Review row declares no `on_reroute`, the target's entry gate does not hold, or the target approves without review (Code-Review budget 0).
  Each case is decided from the built rows and settings, never the recipe name.
- PR-Review clears the send-back mark, the fix-here count, `parent_review_asked` and its session id only after shed persisted the reroute.
  The row clears at the start of its `Call` when its newest history entry is a reroute whose output is the mark's file.
  Every other entry keeps all four, so a crash before the persist, a dirty-tree resume or a reroute-cap resume reroutes the same send-back.
  The landing wiring injects a reader of the run's shed status into the row for this.
- The `ReviewInput` seam now returns every input written after the settle, the newest send-back file and the Parent-Align fixer report alike.
  A send-back file forces a new full generation even on an unchanged subject.
- Accepted risks: the PR-Review agent can skip the verb or transcribe lower severities, and commits after the last Drift-Recheck are checked by neither the rubric nor Drift.
  The fix-here count and the reroute cap bound both where the edge can fire.
- A peer run, `drift-judge-and-line-range`, changes the drift judge's snapshot check and line-range matching in `internal/driftengine`; whichever lands second meets a merge in that package.

## To check by hand

- Four briefs include the Loosened-Check fragment at their end; check that each renders it.
- A burler fix-step test was changed to render the fragment's include (`internal/burlerengine`); check it still pins what its name says.
- Review finding, Code-Bouncer wiring: `battencli` is intentionally nil in the card 3 and card 9 seams but is not listed as such; check the wiring lists it.
- Review finding, `CaptureReview` signature: the changed signature breaks existing tests; check every caller and test compiles and passes.
- Review finding, `resolve-run-id` in `internal/loomcli/prfindings.go`: its pr-findings path has no test; check one reaches it.
- Gate results not run: batch checks ran at the fast tiers only; run the integration tier before merging.
