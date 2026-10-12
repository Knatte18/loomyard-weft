# Tokencount -no-cutoff, reader paren/dollar guards, one weft carve-out

Three independent cleanups.

## What changes

- `tools/tokencount` gets an opt-in `-no-cutoff` flag.
  A skewed archive-tag date could silently drop a run's sessions, with no way to recover them.
  With the flag, every run is treated as a slug with no older archive tag, so no transcript is left out and the shed span start is not clamped by a tag date.
  This reuses the existing "no such tag" path, so no new counting logic exists.
  The report header carries one line saying the cutoff is off.
  Without the flag the report is unchanged.
- `IsReadOnlyCommand` in `internal/fabricengine/refscanner.go` rejects, in a reader segment (`cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq`, `cut`), an unquoted `(` or `)` and any `$` not followed by whitespace, a double quote or the segment's end.
  This closes reader paths that run code without a visible substitution marker, such as a zsh glob qualifier (`cat *(e:'touch x':)`), `${x@P}`, `${a[y]}` and the ANSI-C quote `$'…'`.
  Regex anchors stay read-only (`grep a$ f`, `grep 'a$' f`, `grep "^a$" f`).
  The change only narrows what is read-only.
- `internal/fabricengine/doc.go` documents why reader globs and flags need no other bound, the admission rule for the reader list, and why git admits an unquoted glob after `--`.
- `internal/lyxcwd/enforcement_test.go` drops the `recordsRepositoryName` constant and its `strings.ReplaceAll`, keeping `fabricvocab.BareTokenInRootDoc` as the one carve-out.
  The scan admits the same text on the current tree.

## Design decisions

- `-no-cutoff` lifts both archive-tag lower bounds, so a slug's earlier runs sharing its project directory feed every figure the cutoff gates.
  The plan is read from the newest plan commit before the run's first fork, so an earlier run's fork can move that point back and select an earlier run's plan, or none.
  A main-session transcript with no timestamped line is counted too.
  The flag's description in the `tools/tokencount/main.go` doc comment states this, including the effect on `-calibrate`.
- Reader globs and flags are not bounded further.
  No flag of a listed reader writes a file or runs a command, so a glob expanding into a flag-shaped name does nothing a typed flag could not.
  Bounding flags would need a per-reader allowlist that turns everyday reads such as `grep -rn x --include=*.go internal` non-read-only.
- The dollar rule also guards the command-wide scans: `blankQuotedSpans` knows no ANSI-C quote, so `$'\''` could hide a later substitution or redirection from them.
  Rejecting `$'` in the segment closes that.
- `BareTokenInRootDoc` stays because it lives in the owner package, has its own test and matches the name as a whole word.
  It differs from the removed `ReplaceAll` only for an occurrence glued to a word character, and no occurrence in `CLAUDE.md` or `README.md` is.

## To check by hand

- A future reader added to `readOnlyCommands` with a writing flag (for example `sort -o`) would be admitted unchecked; the documented admission rule and the test rows are the only guard.
- The new fixture rows in `internal/lyxcwd/vocabscan_test.go` (a `README.md` with `[loomyard-weft](https://github.com/Knatte18/loomyard-weft)`, and the same text plus a bare `weft`) carry a glyph/member inconsistency flagged in review as `card3-test-fixture-member-glyph`; confirm the rows read correctly.
- Run `tools/tokencount` with `-no-cutoff` against a real slug whose tag date is skewed, and confirm the header line and a left-out count of 0.
