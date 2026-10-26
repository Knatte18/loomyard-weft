# PATTERN cap measures the rule text only, not the name and tail

The PATTERN entry cap used to count the whole overview line: bullet, backticked name, rule text, `(test)` and background link.
The room left for the wording shrank with the name's length and with whether the entry had a link, so a writer wording a rule could not see the limit, and a text that passed at Discussion could fail at Plan-Write.
The cap now counts the rule text alone, so the limit is one number whatever the entry.

## What changes

- `internal/pattern`: `MaxEntryLineChars` (360, whole line) is replaced by `MaxEntryTextChars` (280, rule text).
  The new `EntryRuleText` cuts an entry line to its rule text: what follows ``- `<name>` — ``, without a trailing ` — [background](...)` and then a trailing ` (test)`.
  `Check`'s `long-line` finding measures that text, and its message now reads "entry rule text is N characters, over the 280 cap".
- `internal/planparser`: the plan gate's `pattern-entry-line-cap` measures the rule text of each fenced entry line a card prescribes.
- `internal/discussionparser`: `pattern-change-over-cap` measures a PATTERN changes bullet's text without any trailing `(test)` or link.
  `lyx loom decision add --pattern-text` already runs the record's own check on the bullet it appends, so it now refuses an over-cap text against the same number when the decision is recorded.
- The discussion and plan specs and the plan stencil describe the rule-text cap.

## Design decisions

- **280 as the cap.** It is the room the longest-named entry with a link had under the old cap (`PATTERN-cwd-resolution`, 360 − 79), so no text that fit there before is refused; every current entry's text is at most 267.
- **discussionparser keeps its own copy.** The package is stdlib-only by its leaf allowlist and `internal/pattern` imports stencil packages, so it holds its own cap constant and tail strip.
  `TestPatternEntryTextCap_EqualsThePatternCap` pins both against `internal/pattern`.
- **Identifiers stay.** The check names `pattern-entry-line-cap` and `pattern-change-over-cap` and the finding kind `long-line` are stable identifiers in specs and run records, so only what they measure and their messages change.
- **The overview byte cap and its headroom are untouched.**

## Tests

- `TestCheck` gains a row with a long name, `(test)` and a link around a text at the cap, a line well past the old 360 cap that now passes, and pins the over-cap message.
- `TestEntryRuleText` covers each tail combination, a `(test)` inside the text, and a line without a backticked name.
- `TestValidate_PatternEntryLineCap` and `TestValidate_PatternChangeOverCap` gain rows for a text at the cap behind a long name and tail, and a text over the cap before a tail.
