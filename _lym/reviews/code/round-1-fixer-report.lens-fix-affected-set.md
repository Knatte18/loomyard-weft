```yaml
lens: fix-affected-set
outcome: reported
```

No findings that I can pin on the fix; one coverage gap is left.

Fork lens fix-affected-set. I reran each set-premise finding's search against the changed tests.

**Findings checked**
- F1 (rows reaching the kept-range clause): covered for the clause itself. The en-dash range rows `cards f–f+2` and `cards f–f+1` reach `previous < e.position && e.position <= number` on insert and on remove. The boundary row, where the insert lands at f+2, leaves `cards f–f+1` unchanged and unlisted.
- F3 (skipped Card Index and heading lines): the Card Index half is covered by delta's Summary, `intent of delta after card f+2`. The heading half was not added as a row, but the moved-card comparison already pins it: each moved card's expected `# Card N —` heading is renumbered once and the scan would shift it again.
- F4 and F2: covered by the added rows. I did not rerun F2's set, the write-order dependence.
- F5, the spec wording set (`contracts/specs/loom-plan-spec.md:226`, `internal/loomcli/doc.go:126-127`, `internal/loomcli/renumber.go:268`): all three state the same rule, so the fix left no twin stale.

**Unchecked**
- F2: I did not verify the write order, that is, whether `01-alpha.md` is written before the Card Index rewrite. The added row that faults the overview write is only a real journaling test if the overview is written after alpha.

**Finding**
- LOW. `internal/planparser/renumber_test.go`, the new `Spans` and `Pair` lines. F1's evidence says the hyphen, em dash and `and`-list forms of `cardMentionRe` (`internal/planparser/renumber.go:625`) are reached by no row. The fix adds only en-dash ranges, so a regression in the `-` or `—` alternation of `mentionNumberRe` (`internal/planparser/renumber.go:628`) would still pass. The review's `fix` key asked only for the en-dash range rows, so this is a gap in the review's proposal, not a defect in the carried-out fix.

**Calls made**
- Grepped `cardMentionRe`, `mentionNumberRe` and "bare reference" wording across `*.go` and `*.md`: to find every other site that states the rule and the dash forms the regex accepts, which my context lacked.
