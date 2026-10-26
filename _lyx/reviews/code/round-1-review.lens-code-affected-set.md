```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each changed cap behavior, I list what reads or asserts it and whether the diff covered it.

**Per-change sets**
- `MaxEntryLineChars` renamed to `MaxEntryTextChars`. A repo-wide grep finds no remaining code reference. The only leftovers are `docs/research/*` measurement records and a comment in `check_test.go:29` that names "the old 360-character" cap on purpose. Covered.
- `Check`'s long-line message changed to "entry rule text is …". The only assertion is in `check_test.go`, updated with `wantMessage`. Covered.
- The `pattern-entry-line-cap` message changed ("with a rule text at %d characters"). The only parser of it is the planparser test's `Sscanf`, updated. `internal/loomcli/parity_test.go` matches only on the check ID and was updated to `MaxEntryTextChars+1`. Covered.
- The `pattern-change-over-cap` detail changed ("a rule text of %d characters"). The discussionparser tests were updated, including `append_test.go`. Covered.
- Specs and stencil. Both specs and `loom-template-plan.md` were updated. A grep finds no other contracts, stencils or recipes that mention the cap. Covered.
- `PATTERN.md` and `pattern/` have no cap text. Covered.
- Package docs. `pattern/doc.go`, `planparser/doc.go` and the `patternchanges.go` header were updated. **Not covered:** two stale places in `discussionparser`, F1 and F2 below.

**Findings**

F1: MEDIUM, scope. Location: `internal/discussionparser/doc.go:11`. The package doc still says Validate refuses a bullet "whose PATTERN.md entry line would run past the PATTERN format checker's line cap". The check now measures the rule text against the rule-text cap, and the decision record's scope lists "the package docs". Evidence: `doc.go:11`, versus `patternchanges.go:152-183`. Fix: reword to "whose rule text, without a trailing `(test)` or background link, would run past the PATTERN format checker's rule-text cap".

F2: LOW, consistency. Location: `internal/discussionparser/validate.go:61`. The comment says the length check yields a Finding per bullet "whose overview line is longer than a PATTERN.md entry line may be". The check now measures the rule text. Fix: reword to "whose rule text is longer than a PATTERN.md entry's rule text may be".

**Tool calls**
1. Grep over md/go/yaml for `pattern-change-over-cap`, `long-line`, "overview line" and cap phrasing, plus `sed -n 5,16p internal/discussionparser/doc.go`. My context lacked the stale-reference search, and it surfaced `doc.go:11` and `validate.go:61`.
