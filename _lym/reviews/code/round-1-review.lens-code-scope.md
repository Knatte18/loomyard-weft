```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points, and check whether every goal point is built.

**Goal points**, from the board entry under `## Goal`. The record has no `Added after Discussion` entries.
- **G1, from Seen and Idea:** a record bullet whose second line is unindented is no longer cut at its first line. The entry allows either of two ways:
  - (a) the record format check refuses such a line when the record is stored;
  - (b) the parser reads the line as part of the bullet, as markdown does.
- **G2, from Seen:** raddle can carry the whole criterion, so the raddle-propose gate accepts it instead of refusing it as `criterion-unknown`, because the gate matches against the parser's bullet text.

**Changes, each traced**
- **C1:** `continuesItem` is new, and `SectionBullets` now continues a bullet through it (`internal/discussionparser/decisionids.go:194-204`, `internal/discussionparser/partsplice.go:68`). This builds G1 by option (b). It also builds G2: `raddleparser` `coverage` compares against `SectionBullets` (`internal/raddleparser/check.go:179`), so the whole bullet is now a known criterion.
- **C2:** `bulletLines` now uses `continuesItem` (`internal/discussionparser/decisionids.go:184`). This builds G1: the acceptance coverage check reads record bullets the same way, so a decision id on the continuation line counts.
- **C3:** the doc comments of `bulletLines` and `SectionBullets`, plus the commit 7d65780fc that splits a comment line (`internal/discussionparser/decisionids.go:169-170`, `internal/discussionparser/partsplice.go:36-38`). Forced by C1 and C2, which made the old "indented line" wording stale.
- **C4:** two new sentences in `contracts/specs/loom-discussion-spec.md:62-63` stating where a bullet ends. Forced by C2: they document the coverage rule that the spec's section states and that the change alters.
- **C5:** test rows `UnindentedContinuationLine`, `HeadingEndsBullet` and `IDOnUnindentedContinuationLineIsCovered` (`internal/discussionparser/partsplice_test.go:92-93`, `internal/discussionparser/validate_test.go:631-634`). Forced by C1 and C2, as coverage of the changed behavior.

**Each goal point, with the changes that build it**
- **G1:** C1 and C2, by option (b). Option (a) was not taken, and the entry lets the diff choose between the two.
- **G2:** C1.

**Findings:** none. Every hunk traces to G1 or G2 or is forced by a change that does. No goal point is left unbuilt, and no `Pattern-Departure:` trailer is needed.

Tool calls: none. My inherited context already held the record, the diff, the commit log and the raddle consumer.
