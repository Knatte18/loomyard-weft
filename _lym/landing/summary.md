# Burler review: quote-colon guidance, halt-note routing, top-rule fix

A reviewer that writes an unquoted `: ` into review-file frontmatter still fails the round with an invalid-YAML error.
This change makes the cause easier to avoid and to trace.
It does not add a repair re-prompt, which both issues also asked for.

Closes #151
Closes #152

## What changes

- The review-format stencil (`contracts/stencils/burler/burler-review-format.md`) now says that every frontmatter value must be valid YAML.
  A value holding `: `, or opening with a YAML indicator character, must be one double-quoted string, with `"` and `\` escaped inside it.
  The existing rule against a quoted fragment followed by trailing prose is kept.
- `burlerengine` (`verdict.go`) adds a hint to the invalid-YAML error when the parser reports `mapping values are not allowed`.
  The hint tells the reviewer to quote the whole value.
  `quotedSummaryHint` is renamed `quotingHint` and takes the parse error.
  The quoted-summary pattern is tightened: it skips backslash-escaped quotes, stays on one line,
  and no longer matches a fully quoted summary followed by a `#` comment.
- The halt note (`haltnote`, `loomcli/halt.go`) gives each history row an optional `routed to <producer>` field.
  A step that handed the run on, by a routed Stuck or a reroute, is now told apart from the step that stopped it.
  This answers the `Code-Bouncer / stuck` row sitting next to a halt at `Code-Burler-Review`.
  `Parse` still reads rows without the field.
- `claudeengine/session.go`: `isTopBoxRule` also accepts an input-box top rule whose label is cut with an ellipsis
  and leaves one rule glyph, as in ` ly:run:code-burler… ─`.
  This is outside the issues' scope.

## Design decisions

- The halt note keeps the `producer` line as it was; the routing is carried on the history rows only.
- The colon hint keys on the parser's error text, so a change in that wording in the YAML library would silence it without failing a build.
  A test pins the current wording.

## To check by hand

- Not done: sending the parse error back to the reviewer for a repair attempt, and writing the frontmatter through a serialising verb.
  Confirm that deferring both is acceptable for closing #151 and #152.
- `quoting-hint-fragment-regex-preempts-colon-hint` in `internal/burlerengine/verdict.go`:
  `malformedQuotedSummary` is checked before the colon case in `quotingHint`.
  A header that holds a quoted fragment and also an unquoted colon on another line gets only the fragment hint.
  The diff has already narrowed the pattern; check whether a remaining header shape still hides the colon hint.
