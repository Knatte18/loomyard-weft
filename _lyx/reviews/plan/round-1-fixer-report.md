# Fixer report — plan, round 1

## Changes

- F1 (fixed as proposed): card 1's board note slug is now `creel-c13-stacked-rewrite`, 25 characters, within `boardengine.MaxSlugLength` (32); every other payload key, the `--body-file` body and the never-writes-`creel-sandbox-suite` rule are unchanged.
- F2 (fixed as proposed): card 2's `TestRunSuite_PerSuiteSpec` bullet now plans `os.ReadFile(tt.spec.fileName)` from `tools/sandbox` compared for equality with `tt.spec.doc`, replacing the heading assertion; the sentence that the existing `tt.spec.doc` check cannot catch a crossed embed is kept.
- F3 (fixed as proposed): card 1's removal list now reads "loses the `---` after S8, S9 with its trailing `---`", matching line 30's S8 Verdict line straight to `## Session log format` (confirmed against `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`, where S8's Verdict is followed by `---` and then S9).

## Disputed

None.

## Deferred

None.
