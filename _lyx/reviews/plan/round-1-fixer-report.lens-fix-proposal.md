```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer's proposals that the fixer carried out as written.

**Findings**

1. MEDIUM. Card 11, the `runBuild` bullet's new sentence: "A build whose shell cannot start … is refused as `runInvocations` refuses a tier", plus the new missing-`dir` row in `TestGateTest_Scenario`.
   - Summary: the proposal says what a refused build prints, but not what happens next. It leaves open whether the call stops, as a tier refusal stops it, or treats the build as failed and blocks the opted-in tiers.
   - The two readings conflict. A shell that never starts writes no build log, so `buildFailure` would be empty. On the blocked path, `reportBlocked`'s `see <log>` and the new `BlockedBy` would name nothing.
   - The new row checks only the refusal text. It does not check the exit code, whether tier0 and tier6 get blocked records, or whether a tier2 that does not opt in still runs.
   - Evidence: `tierResult.refusal` is documented as "set instead of an outcome when the tier could not run at all" (`internal/gatecli/tierrun.go:38`). The card 11 bullet "A failed or timed-out build is a failed run … `reportBlocked` writes its run record … `BlockedBy` the build's log path" covers only a build that ran.

2. LOW. Card 6, line 53 as rewritten: "A failed `git diff`, `git ls-files` or `git show` … returns `Whole` naming the git error".
   - Summary: the rewrite conflicts with line 39, "A cancelled `ctx` is the one error; it is checked before each git read".
   - If `ctx` is cancelled during a git read, the read fails. Under the new sentence that returns `Whole` instead of the context error. So a cancelled call would go on to plan a whole scope instead of stopping the way the go scoper's derivation error stops it.
   - Evidence: the two sentences in card 6, and card 8's "A cancelled context is the derivation's one error, returned as the go scoper's derivation error is."

Checked with no finding: F1 (an inert group mixed with a narrowed group falls back to the module scan, which runs more guards but none wrongly), F2 (`Unused.cs` under `Lib.csproj` reaches `NoTestClass`), F3, F6, F7, F8, F9, F10 and F11.

Tool calls:
- grep of `tierResult` and `sed -n` of `internal/gatecli/tierrun.go:143-170`, to see how a tier refusal is carried and when a start failure is detected, since my context did not hold the `tierResult` struct.
