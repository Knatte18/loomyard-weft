---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed wording says a test of its targets; after F1 a command that takes no targets builds whatever the targets say, so the comment says a test the run executes."
  - id: F3
    disposition: fixed-differently
    reason: "The proposed wording says a run whose targets' tests may use the binary; after F1 that is false for a command that takes no targets, so the comment says a run whose tests may use it."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The proposed sentence omits two conditions the code builds on: a directory or test file that cannot be read or parsed, and (after F1) a command that takes no targets."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: `needsPrebuiltLym` returns true for a command that holds neither `{targets}` nor `{test_targets}`.
  The subtest `CommandWithoutTargetsRunsWhole` in `TestInvocationsUseLymBinary` pins it.
  The doc comments in `prebuild.go`, `doc.go`, `tierrun.go` and `copies.go` state the new condition.
- F2: the `PrebuiltLym` field comment states the condition (fixed differently: not the proposed "a test of its targets").
- F3: the `PrebuiltLymEnv` comment states the condition (fixed differently: not the proposed "targets' tests").
- F4: dropped the `lym gate test --tags` comparison in `docs/benchmarks/running-tests.md`.
- F5: `pattern/PATTERN-test-speed.md` lists every condition that builds (fixed differently: not the proposed sentence).
- F6: the copies prebuild test is now a table, renamed `TestRunTierCopies_PrebuildsLymOnceWhenATestMayUseIt`,
  with a row that asserts no build for a target whose tests do not use the binary.
  The existing row keeps its assertions.
- F7: added rows for `.`, `..` and `../plain` to `TestInvocationsUseLymBinary`.
  The `..` row goes beyond the two proposed, since the finding names that branch too.

F1, F5 and F7 share one commit.
A later commit adds the self-check correction to F5.
Another rewrites the F2 comment as two sentences, after the comment line check refused its wrapped line.

`lym gate test --gate code-burler-fix` passed after that last commit.
The new tests were not run against the unfixed code.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (the `pattern/PATTERN-test-speed.md` list left out a file that cannot be parsed): fixed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
