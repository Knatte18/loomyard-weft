# Comment lint: enforce scribe:prose line-length rule on // comment lines

`comment-lint-line-length` · Task · enhancement, loom · running · Drift-Verify

Depends on: [`comment-cleanup`](design-comment-cleanup.md)

## Seen

`lym loom lint-comments` (`internal/commentlint`) checks only where a `//` comment line breaks.
Nothing bounds a comment line's length.

## Wanted

`scribe:prose` decides how prose is written; `internal/commentlint` only enforces its rules mechanically (operator, 2026-11-07).
Scribe's task `prose-line-length-rule` adds a line-length rule under "Line breaks".
Once that rule has landed and is deployed, make the lint check `//` comment lines the diff adds or changes against it, as the skill words it: the limit, what is counted and what is exempt.
A finding fails the gate as the existing break findings do.
Where the skill's rule cannot be checked mechanically, check the part that can and state the rest in the package's `doc.go`.

Requested by the `comment-cleanup` writer on the operator's word (2026-11-07).
Scribe's rule landed as `6f061ba8f4bc` and is deployed (2026-11-07); read it from `scribe:prose`, under "Line breaks".
