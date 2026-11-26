```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to a goal point of the decision record and each goal point to the changes that build it.

**Decisions.** The answer key is a decision record alone. It has no `Added after Discussion` entries, so its goal points come from the board entry's `## Expected`, `_lym/discussion/decision-record.md:21-22`:
- **G1:** when a later commit in the range touches the test files the stored ruling compares against, such as the class a deleted check moved to, Drift-Verify judges the loosened file again instead of reusing the cached ruling.
- **G2:** at least, the halt's way forward names the parent's ways out of a ruling the code has since answered.

**Changes and where they trace.**
- **C1, reuse rule for uncaused rulings.** It adds `rangeRulingKey` and `holdsUncausedCheck` (`internal/driftengine/loosened.go:46-62`), adds `rangeKey` to `listedFiles` with the changed reuse predicate in `listFiles` (`loosened.go:71-117`), stores `RangeKey` in `newRulings` (`loosened.go:142-157`), and adds the `JudgedFile.RangeKey` field (`internal/driftengine/records.go:175-177`). This traces to G1.
  - The trigger is any change to the range's file diffs, which is wider than "the test files the ruling compares against".
  - The wider trigger covers G1's case and applies only to rulings that hold an uncaused check, which halt anyway.
  - The change description gives the reason: the ruling does not record which files it compared (`_lym/landing/summary.md:20-22`).
  - G1 introduces its example with "such as", so the wider trigger is within what G1 asks for. No finding.
- **C2, a third way forward in the halt.** `uncausedReason` and its doc comment (`loosened.go:182-193`) trace to G2.
- **C3, package doc.** The reuse rule and the way forward in `internal/driftengine/doc.go:240-243` are forced by C1 and C2.
- **C4, test case.** The test case at `internal/driftengine/verify_test.go:1343-1372` is forced by C1 and C2.
- **C5, commit `af96c882e`.** It rewraps comments, and in the range diff it touches only comment lines that C1–C3 add. It is forced by them.

**Decisions and the changes that build them.**
- G1 is built by C1, with C3 and C4.
- G2 is built by C2, with C3 and C4.

**Findings.** None in this lens. No change lacks a trace, no goal point lacks a change, and no commit in the range carries a `Pattern-Departure:` trailer.

Out of lens, for the affected-set fork: `docs/overview.md:1122` still names only two ways out, a trailer commit or a revert. C2 makes it stale, and the diff does not update it.

Tool calls: none.
