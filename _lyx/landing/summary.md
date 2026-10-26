# PATTERN entry cap counts the rule text only

The PATTERN entry cap measured the whole overview line, so the room left for a rule's wording shrank with the entry name and with whether the entry carries `(test)` or a background link.
A writer could not see that number when wording a rule, and an over-cap text recorded at Discussion failed only at Plan-Write.

The cap now counts the rule text alone: what follows ``- `<name>` — ``, without a trailing ` — [background](...)` link and then a trailing ` (test)` marker.
The limit is one number whatever the name.
`lyx loom decision add --pattern-text` runs the record's own check on the bullet it appends, so an over-cap text is refused when the decision is recorded.

## Changes

- `internal/pattern`: `MaxEntryLineChars` (360, whole line) is replaced by `MaxEntryTextChars` (280, rule text).
  The new exported `EntryRuleText` extracts the rule text, and `Check`'s `long-line` finding measures it.
- `internal/planparser`: `pattern-entry-line-cap` measures a prescribed line's rule text through `pattern.EntryRuleText`, and its message names the rule text.
- `internal/discussionparser`: `pattern-change-over-cap` measures a changes bullet's text, with the trailing `(test)` and link excluded, against the new cap.
- `contracts/specs/loom-plan-spec.md`, `contracts/specs/loom-discussion-spec.md`, the plan stencil line naming the cap, and the three package docs describe the new measure.

## Design decisions

- 280 is the room the longest-named entry with a link had under the old cap, so no text that fit before for that entry is refused, and every current entry's text is below it.
- `internal/discussionparser` stays stdlib-only, as `PATTERN-leaf-packages` requires, so it keeps its own cap constant and tail strip rather than importing `internal/pattern`.
  A test pins the constant and the extracted text equal to `internal/pattern`'s on shared rows.
- The check names `pattern-entry-line-cap` and `pattern-change-over-cap` and the finding kind `long-line` are unchanged, since they are stable identifiers in specs, stencils and run records.
  Only what they measure and their messages change.
- The overview byte cap and its headroom are untouched.

## To check

- A changes bullet whose text itself ends in a prose ` (test)` has it stripped before measuring, which gives a few characters of slack and never a false refusal.
