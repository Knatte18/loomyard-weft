```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the F1/F2 changes in `pairbranch.go`, `add.go` and the new tests, run against their callers.

I found two MEDIUM issues and no BLOCKING one.

**MEDIUM — `TaskBranchOnOrigin` now fails where it used to answer.**
- Changed text: `originTaskBranch` in `pairbranch.go` now ends in `originOtherPrefixBranch`.
- Summary: callers that only ask whether a task branch is on origin now get an error in two new cases, and pay for two extra `ls-remote` calls on every miss.
- Evidence, first case: when both name lookups miss, `originOtherPrefixBranch` calls `RecordsRepoRoot`, and its failure is returned as an error. Before, `originTaskBranch` needed no weft repository and answered false.
- Evidence, second case: an ambiguous match across prefixes is now returned as an error.
- Evidence, callers: `TaskBranchOnOrigin` serves `battencli/cancel.go:755,789`, `battencli/cleanup.go:102,179,189`, `beamcli/deps.go:60`, `beamcli/target.go:103`, `battencli/wire.go:1151` and `battencli/readopt.go:125`. `taskbranchdiscard.go:71` also uses `originTaskBranch`. All of them can now fail or slow down on a hub with an unresolvable records repository, or when two pairs of one slug exist under different prefixes.
- Evidence, beam forks: `wire.go:1151` passes a beam name as the slug, so an unrelated `x/<beam>` branch that has an `x/<beam>-weft` branch would now match.
- Whether the first failure is a regression depends on `RecordsRepoRoot` failing on a half-torn hub, which `TestPairTaskBranch_ReadsThePairItself/UnresolvableRecordsRepository…` suggests it can. The fix proposed only `originPairBranch` and `PairTaskBranchOrOrigin` for the fallback. Extending it to `originTaskBranch` went beyond the review.

**MEDIUM — twin left on the current prefix: `RecordsBranchLive` in `remoteleftover.go:84`.**
- It still computes `RecordsBranchName(t.cfg.BranchPrefix + slug)`.
- Its only non-test caller is `boardcli/cli.go:82`, which does not go through `Add` and decides whether a pair is live.
- For a pair made under another machine's prefix it reads not live, while `Add` with `AdoptOnly` now adopts that pair.
- The two lookups now disagree for such a pair.

Checked and sound, with no finding:
- **`slugOfBranch`:**
  - `hanf/foo` gives `foo`.
  - `a/foo` under `b/` gives `foo`.
  - `main` is unchanged.
  - Under an empty prefix, `HasPrefix` is always true and the branch is unchanged, so the existing `TestTaskBranchSlugs` rows still hold.
- **`originOtherPrefixBranch`:**
  - No origin on either repository gives `""`.
  - One suffix match with a records branch gives that branch.
  - Two matches give a sorted, joined error.
  - A suffix match without a weft branch is dropped.
  - A branch such as `x/barfoo` is not matched for slug `foo`.
- **`Add` with `AdoptOnly`:**
  - The lookup returns `prefix+slug` when nothing is found, so the previous behaviour is kept.
  - A lookup error is returned before any mutation.
  - The lookup runs under the teardown lock and before the probes.
- **Added tests:**
  - `OneOtherPrefixIsFoundAndReAdopted` fails with the defect in place, because `PairTaskBranchOrOrigin` returns `hanf/pm-readopt`.
  - The `TwoOtherPrefixesAreRefused` assertion needs the fallback to hold.
  - The new `TestTaskBranchSlugs` row fails before the fix, because the slug stays `other-machine/adopt-fix-x-t5`.

Further tool calls (2 of 6):
1. Read `pairparts.go`, `taskbranchdiscard.go`, `remoteleftover.go` and the `BranchPrefix` uses in `add*.go`, because my context held only the edited spans.
2. Grepped the callers of `RecordsBranchLive` and `TaskBranchOnOrigin`, because the callers' behaviour was not in my context.
