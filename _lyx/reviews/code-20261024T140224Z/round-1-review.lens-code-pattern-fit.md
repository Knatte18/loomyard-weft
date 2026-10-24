```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit. Rules checked against the diff, held or broken.**

Held:
- **Declared PATTERN changes landed.** All 22 changed or kept entries in `PATTERN.md` match the record's declared text. All 72 removed IDs are gone from the overview. The overview holds 30 entry lines at 6966 bytes, under `HeadroomBytes`. I compared the entries by eye against the declarations, not with a script.
- **`pattern.Check` background rule.** The `pattern/` files match the overview's links exactly (the `diff` came back empty), so there is no orphan and no unlinked file. The longest entry line is 317 characters, under `MaxEntryLineChars` (360). I did not run `pattern.Check` itself.
- **Background merge shape.** `pattern/PATTERN-shed-recipe-registry.md` has `## Setting lookup`, `## Command rows` and `## Entry gates` sections. The shed-driver and pane-text-injection files also gained `##` sections.
- **Commit messages and departures.** The one commit carries no `Pattern-Departure:` trailer, and the diff needs none.
- **Dropped entries.** `CLAUDE.md` states the cgo build (line 12). The background files of the four dropped entries are deleted.
- **Moved-rule test messages.** Failure messages in the moved-rule tests (`mutation-record`, `glyph-conversion-chokepoint`, `agent-name`, `dev-prod-binary-separation`) now point at the owning package doc. Messages for absorbed entries point at the survivor ID.
- **No enforcing test removed.** The only non-comment, non-message Go changes are the new `doc.go` files and the message edits.
- **`tools/sandbox/doc.go`.** It is the only package doc in `tools/sandbox`. It states both moved rules and the `.dev-bin`/`resolveLyx` enforcement.
- **Comment line breaks.** Added `doc.go` comments keep one sentence per line.
- **Scope.** The diff leaves `internal/pattern`, `testdata/` and the `docs/research/` and `docs/benchmarks/` report texts unchanged.

Broken:

1. **LOW / consistency** — a removed ID is still cited in a live file. `internal/discussionparser/validate_test.go:258` has the fixture line ``"- `PATTERN-hub-suffix` — changes: ..."``, and `PATTERN-hub-suffix` is a removed ID. The record's acceptance criterion says no live file outside `docs/research/`, `docs/benchmarks/` and `testdata/` cites a removed ID. The string is inert parser input, so the fix is to swap it for an invented ID such as `PATTERN-old-rule`, or for a surviving one. This is the same leftover the focus file flagged.

Not checked: whether `docs/overview.md` and the swept `docs/` and `contracts/` citations resolve, which belongs to the affected-set lens.

Tool calls (3):
1. Bash: listed `package main` doc comments in `tools/sandbox`, compared `pattern/` files against `PATTERN.md` links, searched added `doc.go` lines for multi-sentence lines, listed `##` headings in merged background files, and measured the longest `PATTERN.md` line. My context lacked all of these.
2. Bash: grepped `internal/pattern` for the `HeadroomBytes` and `MaxEntryLineChars` definitions. My context lacked the cap values.
3. Bash: grepped `internal/pattern/check.go` for `MaxEntryLineChars =` and `CLAUDE.md` for the cgo line. The previous grep did not return the cap value.
