Task: check that the F1 fix covers every site of the "last failing tier" wording.

The only finding whose premise is a set is F1, and the fix covers it. I searched `.go`, `.md` and `.yaml` files, leaving out `_lyx` and `docs/research`. The query was "last failing", "last failed", "last tier summary", "failing one in a log" and "every later run still ran".

- No stale "last failing tier" wording is left.
- The remaining hits are unrelated: poll-failure comments in `internal/statuspane/cache.go:31` and `internal/battencli/overvieworch.go:42`.
- `internal/gateslot/doc.go:116` now says "first".
- The old closing-line text "every later run still ran" is gone from the code and docs.
- `internal/verifytree/doc.go:34` and `pattern/PATTERN-verified-tree.md:17` never said "last", so there was nothing to change in them.

No findings.

Tool calls:
1. Bash grep over `.go`, `.md` and `.yaml` files for the stale wording, to re-run F1's own search. My context held only the edits, not a full-repo scan.
