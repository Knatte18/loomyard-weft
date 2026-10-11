# Card 10 — pattern-overview-trim

**Prosa:**
- `PATTERN.md`

**Create:**
- `pattern/PATTERN-push-both-sides.md`
- `pattern/PATTERN-edit-directive.md`
- `pattern/PATTERN-wait-mark-display-only.md`
- `pattern/PATTERN-spawn-color.md`
- `pattern/PATTERN-hub-wide-config.md`

**Uses:**
- `internal/pattern/check.go`

**Intent:** Shorten the overview to at most 15000 bytes, decision D4, so the next task that adds an entry does not have to cut unrelated wording.
Every entry keeps its name, its trigger, its requirement and its `(test)` mark; no entry is dropped or merged, and no rule changes.
Detail leaves the entry line for the entry's background file: an entry that has one gets the detail appended there, and the five entries named under Create, whose lines are the longest without a background file, each get a new file holding the detail the line loses, and the line gains its `— [background](pattern/<name>.md)` link, whose bytes count toward the target.
The cut comes mostly from the longest lines, those over about 230 characters, so each loses a clause of mechanism, example or history that its background file now holds; a line under 180 characters is left alone.
Multi-byte characters count: the size is bytes, as `pattern.Check` measures it, so the target is checked with `wc -c`.
Every new background file is named by exactly its own entry's link, keeping `Check`'s orphan and link rules green, and the ordering of entries and headings is unchanged, so a reviewer can diff each shortened line against its prior wording for a dropped requirement.

**Verify:** go test ./internal/pattern && [ "$(wc -c < PATTERN.md)" -le 15000 ]
