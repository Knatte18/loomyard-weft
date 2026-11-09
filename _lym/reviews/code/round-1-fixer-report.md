# Round 1 fixer-report

## Changes

Each fix is its own commit, `<target>: fix <id> — …`.

- F1: `pattern/PATTERN-test-speed.md` says `-redundancy` judges a candidate over the blocks of its own package's files and adds a "Contained module-wide" column. The "no coverage" bullet reads "covers nothing of its own package".
- F2: `spliceReport` in `cmd/testtiming/splice.go` rewrites the header from `renderReport(tags, nil)`. Its doc comment says so. `TestSpliceReportRefreshesStaleHeader` fails with the old code.
- F3: `TestWriteMapReadMap` writes a second map that sets `BuildFailure` and holds fewer tests and blocks, reads it back, and asserts it equals the second map.
  The second write comes after the unknown-key check, not right after the first round trip as the `fix` key said: the unknown-key check reads back the first map.
- F4: `TestJudgePackage` has a second run where a kept candidate is contained in the package and in the module. It asserts `candidate`, `moduleContained` and the keep text.
- F5: a stand-in test `TestStandInSkipWithProfile` prints a SKIP line and writes a profile, and `TestRunAlone` has a `SkipWithProfile` row expecting `ActionSkip` with that profile's block.
- F6: `TestKeyBlocks` has a two-file case: the same body in `p.go` and `q.go` keys with each file's own `File`, and a generic receiver `T[K]` keys as `T.Do`.

Self-check follow-ups, committed together as one `F2` commit, plus a gate repair that split the over-long `-redundancy` flag doc line in `cmd/testtiming/main.go` into two sentences:

- The `-redundancy` flag doc in `cmd/testtiming/main.go`, the `runRedundancy` doc comment and the PATTERN splice sentence say the header is rewritten too.
- The comment in `TestKeyBlocks` no longer claims the ordinal assertion pins per-file counting.

## Disputed

None.

## Deferred

- Regenerating `docs/benchmarks/test-redundancy.md` (lens fix-proposal, finding 1): F2's `fix` key offered it as the alternative to the code change.
  It needs a whole-module coverage run under a gate slot that rewrites every package's section, which is a run of minutes, not a fix of this diff.
  Until the next whole-module run, a `-pkg` splice puts the current header over unmeasured sections that keep the older three-column form.
  I left it for the operator or the orch to schedule.

## Rulings

None.

## Self-check

- fix-consistency, MEDIUM: `main.go` flag doc still said the header stays byte-identical. Fixed as proposed.
- fix-consistency, LOW: `runRedundancy` doc omitted the header rewrite. Fixed as proposed.
- fix-consistency, LOW: PATTERN splice sentence omitted the header rewrite. Fixed as proposed.
- fix-code-correctness, LOW: the cross-file ordinal assertion cannot fail from the defect, since the key already holds `File`. Fixed with my own change: the assertion stays, as it is correct, and the comment no longer claims it pins per-file counting; the `File` and generic-receiver assertions are the ones that bite.
- fix-affected-set, LOW: `internal/testcover/doc.go:13` still says "covers nothing". Disputed on ground premise false: that sentence describes `Judge`, which judges the blocks a caller supplies and has no package view, so it holds as written.
  I list it here, not under Disputed above, because the fork raised it after the review; the evidence is `internal/testcover/verdict.go` `notJudgedReason`, which reads `len(r.Blocks) == 0`.
- fix-proposal, LOW: refreshed header sits over unmeasured older sections. Deferred, see above.
- fix-proposal, LOW: `docs/benchmarks/test-redundancy.md` not regenerated, the same item. Deferred, see above.
