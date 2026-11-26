```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: everything outside the changed lines that the diff's changes force, judged covered or not.

**Per change, the set found:**

1. **`SectionBullets` now continues a bullet through an unindented line** (`internal/discussionparser/partsplice.go:68`).
   - `requireBullets` / `SplicePart` (`partsplice.go:117-121, 206`): covered. They compare the part's texts against `SectionBullets`. `renderBullets` (`partsplice.go:230-241`) writes continuation lines indented, so a spliced bullet reads back the same text.
   - `raddleparser` `coverage` (`internal/raddleparser/check.go:179`): covered. It matches carried texts exactly against `SectionBullets` output, so it now accepts the whole two-line bullet.
   - `contracts/specs/raddle-proposal-spec.md:49` ("each the bullet's whole text as the decision record holds it") and `:87-88` (criterion-uncarried/unknown): still true. Not changed, and none needed.
   - `TestSectionBullets`, `TestSplicePart` (`partsplice_test.go:80-104, 192-196`): rows added for the new behaviour. The test's doc comment is not updated (finding 2).
   - Carry-over writers and readers (`dropCarryOverBlocks` at `identity.go:36`, `carryover.go`): their marker lines open with `&lt;`, which `continuesItem` excludes. Covered.
   - Package doc `internal/discussionparser/doc.go`: states no continuation rule. Nothing left stale.
2. **`bulletLines` / `CheckDecisionIDs` coverage now reads unindented continuation lines** (`decisionids.go:184`).
   - `CheckDecisionIDs` callers (discussion gates, Darn-Write guard, `lym loom validate-discussion`, as the spec lists): behaviour only, no signature change. Covered.
   - `contracts/specs/loom-discussion-spec.md:61-65`: updated, but the new sentence is placed so that it breaks the spec's reasoning (finding 1).
   - `TestCheckDecisionIDs` (`validate_test.go:631`): row added. Covered.
3. **New `continuesItem`** (`decisionids.go:199`): its callers are only `bulletLines` and `SectionBullets`. Covered.
4. **Other readers of record bullets that keep their own rule:**
   - `patternBulletRange` (`patternchanges.go:310`) reads a single bullet line verbatim. `## PATTERN changes` bullets are one line by format, so it is outside this change.
   - `internal/partsparser` and `contracts/specs/loom-parts-spec.md:61` keep "indented lines" for a different file format (parts), not the decision record. Not affected.
5. **Repository rules:**
   - `PATTERN-sole-parsers` (`pattern/PATTERN-sole-parsers.md:68`): `discussionparser` stays the sole reader. Covered.
   - Declared PATTERN changes: the directive declares none, so there is nothing to land.
6. **Commit messages:** `1c292fd21` carries a `Root-Cause:` trailer naming the defect the diff removes (continuation read only through indented lines). `7d65780fc` is a comment-wrap fix. There are no `Pattern-Departure:` or `Loosened-Check:` trailers, and none is needed.

**Findings:**

1. **LOW · consistency** · `contracts/specs/loom-discussion-spec.md:62-64`
   - **Summary:** The two new sentences sit between "Each id appears as a whole word in a bullet of `## Acceptance criteria`." and "So `D1` is not met by `D10`, …". "So" now follows the bullet-ending rule, which it does not follow from: `D10` failing to cover `D1` follows from the whole-word rule. The spec's reasoning chain is broken.
   - **Evidence:** `contracts/specs/loom-discussion-spec.md:61-64`, and the diff `001-contracts-specs.diff:8-11`.
   - **Fix:** Move the two new sentences after the "So `D1` …" sentence, for example after line 65.

2. **NIT · consistency** · `internal/discussionparser/partsplice_test.go:79`
   - **Summary:** The doc comment of `TestSectionBullets` still lists "plain and continuation bullets, a fenced bullet, a carry-over block and an absent heading". It does not name the new heading-ends-a-bullet row, so it no longer states everything the test pins.
   - **Evidence:** `partsplice_test.go:79` and `:93`.

**Tool calls:**
1. grep `contracts/`, `stencils/`, `docs/`, `pattern/` and `PATTERN.md` for continuation and indent wording: the context lacked the docs that state a bullet rule.
2. grep for criterion-unknown and `SectionBullets` in specs and docs, and for the raddle spec file: the context lacked the raddle spec.
3. Read the raddle spec's bullet lines and the discussion spec around lines 55-70: the context had only the diff hunk, not the surrounding spec text.
4. grep `doc.go` and `ParsePatternChanges` for bullet-continuation rules: the context lacked the package doc and the PATTERN-changes reader.
