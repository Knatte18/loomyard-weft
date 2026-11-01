# Fixer report, round 1

## Changes

- **F1** fixed as proposed in `crucible/orchestrator-prompt.md`: `LYM_TEST_TIER=2` (or `3` for the LLM-driving line) in front of each smoke `go test`, `-tags` dropped, and `export LYM_TEST_TIER=2` before the loop that runs the compiled binary.
  The self-check found the same commands in `crucible/README.md` and `crucible/review-prompt-template.md`; both are fixed the same way.
  The summary's "Left as they are" bullet for `crucible/` is removed.
- **F2** fixed as proposed: `internal/testkit/tmuxkit/tmuxallowed_test.go`, a table over unset, 0, 1, 2, 3, 0-2, 1-3 and a malformed value.
- **F3** fixed with my own change: the proposal's example was a subprocess re-execution of the test binary.
  A subprocess in a tier0 file breaks the tier-purity guard (`exec.Command` is a banned token).
  `TestRequire_FailsOnAMalformedSelection` runs `Require` in-process against a recording `testing.TB` whose `Fatal` and `Skipf` end the goroutine, and asserts failure, no skip, and the variable named in the message, for `tier1` and `2-0`.
- **F4** fixed as proposed: `exportTierSelection` in `cmd/testtiming/main.go`, `main` calls it, and `TestExportTierSelection` asserts `LYM_TEST_TIER` is `1-2` after `tier1,tier2`, `0` for no tags, and unchanged after a gap error.
- **F5** fixed as proposed, rows only plus the split:
  - `TestFindGuardTests`: non-literal tier, other qualifier and one-argument rows, each expecting the file's tag tier.
  - `TestScanSpawns`: a fixture whose test opens with `tiergate.Require(t, 2)`, expecting out of process.
  - `cmd/lym/tierpurity_test.go`: the scan body is `tierPurityFailures` over source text, with `TestTierPurityFailures` rows for the support-file skip, a tier0 banned token, a tier1 tmuxkit reference, the allowlisted case and tier2.
- **F6** fixed as proposed: `TestMain` of `internal/battencli` installs the refusing seam ("unit tests push no pair") when `tiergate.Selected()` is tier0 alone, the counting real push otherwise.
  The comment names the bound, a run selecting tier0 alone.
- **F7** fixed at every site the review listed, and at the sites the affected-set fork found: ten testmain and helper headers saying "carries no build tag", and fourteen test-file comments saying "untagged" or "integration-tagged".
  `PATTERN-test-speed.md` line 167 now agrees with the `cmd/testtiming/main.go` wording.
  The word `untagged` in `contracts/specs/loom-plan-spec.md` (plan-gate token) and `internal/loomshed/doc.go` is left: the code still matches it.
- **F8** fixed per the parent's decision (creel #372, recorded in the decision record): `_lym/landing/summary.md` Verification now gives main's and the branch's per-tier counts from `lym gate test --upto 2 ./...` run on each, and names the tier0 difference.
  Main: 22502, 3790, 268 passed. Branch: 22547, 3790, 268 passed.
  The tier0 difference of 45 is 23 new `tiergate` and `TestTierSelection` passes, 26 from this round's tests and 10 fewer from the retired `TestIsTierTagged_RecognizesKnownTagsList` with its nine subtests, leaving 6 that no named test accounts for.
  The summary says so in as many words.

Gate repair: the gate found fixed-column-wrapped comment lines in three F7 rewordings (`internal/fabricengine/rawworktree_test.go`, `internal/hubforge/testmain_test.go`, `internal/loomcli/testmain_test.go`).
Each was broken at sentence boundaries, one sentence per line, in a further F7 commit.
A second gate run still flagged `internal/hubforge/testmain_test.go:10`, whose sentence ran over two lines at a comma; it is now two sentences, one per line, in a second repair commit.

Gate after the last edit: `lym gate test --upto 2 ./...` green on all three tiers before the follow-up wording edits, `--upto 0 ./...` green after them.
Commits are one or more per finding; F4 and F5 also carry the F7 wording of the files they edit.

## Disputed

None.

## Deferred

None.

## Self-check

Four forks returned; their reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: crucible README and review template still on `-tags` — fixed as proposed (F1 above).
- fix-consistency, LOW: `PATTERN-test-speed.md` line 167 wording — fixed as proposed.
- fix-consistency, LOW: summary line 42 vs the Verification command — unverified by the fork; line 42 describes the orch's gate config (`-tags` kept beside `LYM_TEST_TIER`), the Verification section names the command run, so they do not conflict. No change.
- fix-affected-set, MEDIUM: same crucible files as above — fixed as proposed.
- fix-affected-set, LOW: the wider set of stale build-tag wording — fixed as proposed (F7 above).
- fix-proposal, LOW (F6 comment overstates the refusal): fixed as proposed; the comment now says a run selecting tier0 alone.
- fix-proposal, LOW (`Tier2FileReferencingTmuxkitSpawn` row does not pin `tier < tmuxTier`): no change; the tier1 row pins the comparison, the fork judged no change needed.
- fix-proposal, LOW (fixture import outside `fixtureModule`): no defect per the fork, no change.
- fix-proposal, LOW (`recordingTB` embeds a nil `testing.TB`): no change; the fake is bounded to the calls `Require` makes and a new call would fail loudly.
- fix-proposal, LOW (`export LYM_TEST_TIER=2` persists in the pasted block): no change; harmless as the fork says, and the block's later lines do not read the variable.
