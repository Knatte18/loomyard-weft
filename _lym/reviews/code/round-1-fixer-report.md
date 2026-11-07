# Round 1 fixer-report

## Changes

- F1 (LOW): `TestSharedHub_PackedTemplateHoldsNoLooseObject` in `internal/hubforge/template_test.go` now collects, in its existing walk, every regular file under a `refs/heads` or `refs/remotes` directory, and fails when any exists.
  It is an assertion of the existing test, not a new test.
  Fixed differently from the `fix` key in two respects, both found by running the proposed check:
  - Reflog files under `logs/refs/` also match `refs/heads/` and `refs/remotes/`, so they are excluded.
  - `refs/remotes/<remote>/HEAD` is a symbolic ref that `pack-refs --all` always leaves loose, so it is excluded.
  The path is matched relative to the template root, so a `logs` ancestor of the root cannot make the check vacuous.
  The `packRepositories` doc comment in `internal/hubforge/template.go` now says "no loose branch ref", matching what the test pins.
  Commits: the assertion, then the scoping from the self-check.

## Disputed

None.

## Deferred

None. F2 is a `decision`-class finding; it went to the parent, and the fixer-report leaves it out.
The parent's recorded decision (creel #1008) keeps `--cruft`, so the code stays as it is.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the `packRepositories` comment said "no loose ref" while symrefs stay loose; fixed as proposed (comment now says "no loose branch ref").
- fix-proposal: LOW, the `/logs/` match covered the whole absolute path; fixed as proposed (path taken relative to the root, match narrowed to `/logs/refs/`).
- fix-proposal: LOW, the `HEAD` exclusion also skipped a real `refs/heads/HEAD`; fixed as proposed (exclusion limited to `refs/remotes`).
- fix-affected-set: no findings.
