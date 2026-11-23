---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

## Changes

- F1: `internal/loomcli/lintcomments_integration_test.go` doc of `TestLintCommentsCmd_OverARepository` now says the steps that run both verbs assert agreement, and the last step checks that `lint-comments --base` reports an uncommitted edit.
  Committed as `2512d2748`, then narrowed in a second commit after the self-check (see below).
- F2: `_lym/landing/summary.md` gains a sentence under `## What changed`: an untracked `.go` file the base still holds is compared with its base copy, so only its changed lines count.
  The file is under `_lym`, so it is edited in the working tree and not committed.

- Gate repair: `TestCheckedCallInvariant_RawSitesMarkedAndPinned` failed because the `cat-file -e` probe in `internal/commentlint/diff.go` calls `gitexec.RunGit` without a `//gitexec:raw` marker.
  Added the marker and pinned `internal/commentlint` at one raw site in `cmd/lym/checkedcall_test.go`.
  The probe reads the exit code as its answer, so the raw form is correct.
  `lym gate test ./cmd/lym ./internal/commentlint` passes.

- Second gate report (reach): it lists `changedGoFiles`, `readChange`, `Lint`, `LintCommitRange` and `lintChanges` in `internal/commentlint` as reached by no mapped test.
  No code changed.
  These functions run `git`, and `pattern/PATTERN-test-isolation.md` and `pattern/PATTERN-test-speed.md` say a `tier0` run spawns nothing, so no `tier0` test can drive them.
  `TestLint_OverARepository` (`internal/commentlint/lint_integration_test.go`, tier 1) already drives all five over a real repository, and `TestLintCommentsCmd_OverARepository` drives them through the verb.
  The pure parts, `parseNameStatus` and the rule functions, have `tier0` tests in `lint_test.go`.

## Self-check

- fix-proposal, LOW, F1 wording "steps up to the base commit" still overstated, since three of those steps run one verb: fixed (reworded to "the steps that run both verbs").
- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
