# Decision record — pattern-line-cap-text-only

## Goal

The PATTERN entry cap measures the rule text alone, so a writer wording a rule works against one number whatever the entry's name and whether it has `(test)` or a background link.

## Scope

- `internal/pattern`: replace the whole-line cap with a rule-text cap and export the function that extracts an entry line's rule text.
- `internal/planparser`'s `pattern-entry-line-cap` plan check measures a prescribed line's rule text against the new cap.
- `internal/discussionparser`'s `pattern-change-over-cap` measures a PATTERN changes bullet's text against the new cap, so `lyx loom decision add --pattern-text`, which runs the record's own check on the bullet it appends, refuses an over-cap text when the decision is recorded.
- The plan and discussion specs under `contracts/specs`, the plan stencil line naming the cap, and the package docs.

Out of scope: the overview byte cap and its headroom, which stay as they are.

## Decisions

### The rule text is the part between the name and the tail

Decision: an entry line's rule text is what follows ``- `<name>` — `` up to, and without, a trailing ` — [background](...)` link and then a trailing ` (test)` marker.
Rationale: this is the part a writer words; the name, `(test)` and the link are fixed by the entry's identity and enforcement, not by the wording.

### One cap constant, MaxEntryTextChars = 280

Decision: `pattern.MaxEntryLineChars` (360, whole line) is replaced by `pattern.MaxEntryTextChars` (280, rule text only), and Check's long-line finding measures the rule text.
Rationale: 280 is the room the longest-named entry with a link had under the old cap, so no text that fit before for that entry is refused, and every current entry's text is below it.

### discussionparser keeps its own copy of the cap and the tail strip

Decision: `internal/discussionparser` stays stdlib-only and holds its own cap constant and tail strip; a test pins the constant and the extracted text equal to `internal/pattern`'s on shared rows.
Rationale: the package's leaf allowlist admits no import, and `internal/pattern` imports stencil packages.

### Check identifiers stay

Decision: the plan check keeps the name `pattern-entry-line-cap`, the discussion check `pattern-change-over-cap`, and Check's finding kind `long-line`; only what they measure and their messages change.
Rationale: the names are stable identifiers in specs, stencils and run records, and renaming them buys no behavior.

## Constraints

- `PATTERN-leaf-packages`: `internal/discussionparser` imports the standard library only.
- `PATTERN-sole-parsers`: planparser and discussionparser stay the sole parsers of their files.
- `PATTERN-test-speed`: tests are tier0, table rows on the existing tests.

## Auto-mode assumptions

- The cap value 280 is chosen by the writer; the board entry names no number.

## Open risks

- A discussion bullet whose text itself ends in a ` (test)` written as prose would have it stripped before measuring; the effect is a few characters of slack, never a false refusal.

## Acceptance criteria

- Check reports `long-line` for an entry whose rule text exceeds `MaxEntryTextChars`, and none for a line longer than the old whole-line cap whose rule text fits, with and without `(test)` and a link.
- The plan gate's `pattern-entry-line-cap` fails a prescribed line whose rule text is over the cap and passes one whose name and link push the whole line past 360 while the text fits.
- The discussion check's `pattern-change-over-cap` measures only the bullet text, a trailing `(test)` and link excluded, and `lyx loom decision add --pattern-text` refuses an over-cap text.
- The discussionparser cap and text extraction agree with `internal/pattern`'s.
- The repository's own `PATTERN.md` passes Check.
