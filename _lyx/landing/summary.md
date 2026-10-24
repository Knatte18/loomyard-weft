# Rebuild the run tail: Merge-In fixer, Code-Perch, locked Finalize

Loom and darn now share one tail after the code is written, and each recipe fixes its own merge and test failures inside it.
The parent lock covers only Finalize, and the pull-request rows go.

- Loom: `Webster` → `Merge-In` → `Code-Bouncer` ring → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`.
- Darn: `Preflight` → `Darn-Write` → `Darn-Bouncer` ring → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`, with no Merge-In.

Removed: Merge-Review, Publish, PR-Align, PR-Gate, PR-Rework, `lyx loom approve` and `lyx loom reject`, the PR-closing comment, `alignment_pr_approve`, and every route that sent a verify failure to another row, including batten's publish retry.

## What changes

- **Merge-In** takes no parent lock and writes nothing in the parent pair.
  It merges the parent's upstream tip, or the local tip read under the lock when the push is skipped or there is no upstream.
  `fabricengine` gains `MergeInAt(source, warpSHA)`, and `mergeresolve.Resolve` and its `Fabric` seam carry the SHA.
- **Merge fixer** (`internal/landingshed`): one cold Opus session, shared by Merge-In and Finalize, with its own stencil and a report spec and parser.
  At Merge-In it aligns: it reads what the parent brought in since the aligned tip against the decision record, fixes the failing tests, and escalates a conflict with a decision instead of choosing.
  At Finalize it fixes failing tests only.
  The merge record's reviewed-tip field becomes the aligned tip.
- **Code-Perch** replaces the Webster-Review perch and runs after Merge-In.
  It reviews the task's diff plus the merge fixer's latest report, with the decision record and plan as its answer key.
  Merge-Review rows, rubric and seams are gone.
- **Finalize** is one producer under the parent lock.
  It fast-forwards the parent, merges it in, runs the `landing` gate and the finalize-mode fixer, then lands with a `Lyx-Task: <slug>` trailer and pushes.
  It opens no pull request.
  A loom conflict, a large merge, a spent fixer cap or an over-cap fix reroutes to Merge-In (loom) or Darn-Bouncer (darn), keeping the commits.
  The next reroute past shed's cap halts the run.
- **PR-Review** is a stub row behind `pr_review` in `loom.yaml` and `darn.yaml`, default `false`.
  Either way it is Done without opening a PR or calling GitHub.
- **Gates**: Darn-Write, Darn-Burler-Fix and Code-Burler-Fix run the `landing` call.
  The `publish` gate kind is gone, and the Publish failure record is now the landing failure record, written by Merge-In and Finalize and read only by the fixer.
- **Merge base**: readers of the task's diff (Code-Perch, Describe, the headroom and comment lints) take the merge base with the merge record's merged tip, since the local parent can now lag it.
- **Config**: `webster_*` perch keys become `code_*`, `require_pr_to_base` becomes `deploy_branches` (read only by batten's Land row), and `merge_*`, `alignment_pr_approve`, `rework` and the gate `publish` list are retired.
  New `landing.yaml` keys: `fixer`, `fixer_timeout_min`, `merge_fix_attempts`, `finalize_fix_attempts`, `finalize_fix_max_lines`.
  Each module's migration carries its renames, and profiles and board `run_settings` are migrated at read time with a Warn per line.
- **Landing halts** (fixer spent, escalation, fixer timeout, budget-0 over-cap, Finalize reroute cap) are handed back to the orch.
  The Go driver, `arbiterstop`, batten's binary re-step and the LLM driver stencils skip them, and the stop envelope and `lyx shed status` carry a `hand_back` field.
- Batten's idle release keeps the parent lock of a running Finalize or one with a live verify marker.
- `lyx report convergence`, `tokencount`, the orch role stencil, `CLAUDE.md`, `README.md`, `docs/overview.md`, specs, PATTERN lines and package docs follow.

## Design decisions to know

- Finalize's fixer is capped at `finalize_fix_max_lines` added plus deleted lines, and any change to a test file of the run's language, `testdata`, gate or build config, or a test skip or build constraint is over the cap whatever its size.
  Per-language test-file patterns live in `internal/planparser`.
  `language: none` or an unknown language treats every file as a test file.
- Where the way back holds no review (a Code-Review budget of 0, or the budget-0 darn recipe), an over-cap or spent fix halts instead of rerouting.
  The same caps bind Merge-In's fixer at a Code-Review budget of 0, measured from the HEAD after the merge and conflict session.
- A conflict the conflict session resolved is no fixer change, so at a perch budget of 0 it lands with the `landing` gate alone.
- Finalize re-checks its lock record just before landing, but the re-read and the squash are not atomic; a release between them is caught only by the parent push's rejection.
- An in-flight run parked on a removed row is restarted, not migrated, so this should land while no other run is in flight.

## Check by hand

- Finalize and Merge-In were not run against a real parent with a competing run; run one loom and one darn task end to end, with `pr_review` off and on.
- Fixer-run turn handling in `internal/landingshed`: a hung later turn must read as timeout or no-report, never `fixed`, over a failed gate.
- `internal/loomcli`'s package doc and comments: confirm no retired PR-Gate, Merge-Bouncer or reject-review text remains.
- The over-cap resume at Merge-In over a still-failing gate (the fixer runs on the attempts left) has no test of its own.
- The verify gate failed once and a fixer commit (`dc3edaca`) cleared it; re-check `cmd/lyx` `TestDriverStencil_IsRecipeBlind`, `internal/shedbuild` `TestBuild_EveryRegisteredEngineBuilds/PRReview` and the `internal/loomrecipe` `TestResume_*` and `TestSequence_FullRunBlocksAtMergeIn`.
- Stale mentions of the Publish record or row remain in `contracts/specs/loom-final-summary-spec.md` and `webster-spec.md`.
- `findRunPullRequest` in `loomcli/runstart.go` and `parentlock.Store.HandOff` have no production caller now; `TimingAttempts.Alignment` stays and reads zero.
- Hub-wide orch rules on Bolt that name PR-Gate, Publish or batten's verify retry, and other board entries naming the removed rows, are for the orch to rewrite after landing.
- Merge-In spawns an Opus session on every merge that brings parent commits in, and under a `language: none` plan at budget 0 any fixer change halts Merge-In.
