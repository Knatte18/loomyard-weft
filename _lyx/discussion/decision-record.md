# Decision record — cleanup-1012

## Goal

Three independent cleanups from recent PR-Gate reviews:

1. `tools/tokencount` gets an operator switch that turns off the archive-tag date cutoff, so a skewed tag date cannot drop sessions with no way to recover them.
2. The read-only command classifier in `internal/fabricengine/refscanner.go` closes the one reader-glob path that can run code, an unquoted parenthesis such as a zsh glob qualifier, and `internal/fabricengine/doc.go` documents why reader globs and flags need no other bound and why git admits an unquoted glob after `--`.
   The brief's "bound audit reader globs" is met by that parenthesis rule, not by a general glob or flag bound on readers; D2 gives the reason.
3. `internal/lyxcwd/enforcement_test.go` keeps one of its two `loomyard-weft` carve-outs, with no change in what the fabric-vocabulary scan admits.

## Scope

In:

- A `-no-cutoff` flag on `tools/tokencount` and its description in the `tools/tokencount/main.go` doc comment, in the same commit.
- In `IsReadOnlyCommand`, a reader segment holding an unquoted `(` or `)` is not read-only; its doc comment says so.
- A `doc.go` paragraph in `internal/fabricengine` with the reader rationale, the reader-list admission rule and the git pathspec reason, pointing at `IsReadOnlyCommand` for the full rule set.
- New rows in the existing `IsReadOnlyCommand` test table and in the existing `fabricVocabularyFailures` fixture table.
- Removing the `recordsRepositoryName` constant and its `strings.ReplaceAll` from `fabricVocabularyFailures`.

Out:

- Any other change to which commands `IsReadOnlyCommand` admits.
- A tolerance window on the tokencount cutoff, and any automatic skew detection.
- Changes to `fabricvocab.BareTokenInRootDoc` or `recordsRepoNamePattern`.

## Decisions

### D1 — tokencount: a `-no-cutoff` flag drops both archive-tag lower bounds

With `-no-cutoff` set, every run is treated as a slug with no older archive tag: `runBound` yields the zero time, so `CountRun` leaves no transcript out, and the shed record's `Older` is zero, so the span start is not clamped by a tag date.
This reuses the existing "no such tag" path, so no new counting logic exists.
The flag defaults to off, so the default report is unchanged.
When set, the report header carries one line saying the cutoff is off and every transcript in each slug's project directory counts.
The doc comment's "no flag turns the filter off" sentence is replaced by the flag's description.

Rationale: the cutoff drops sessions chiefly when a tag's commit date is later than a later run's first transcript, as a clock skew would make it, and the per-run left-out count tells the operator when to reach for the off switch.

Bound: the flag is opt-in per invocation and touches nothing outside the report.
With it, a slug's earlier runs that share its project directory feed every per-run figure the cutoff gates: tokens, cost, working intervals, the session count, the Webster forks, the Merriam starts, the begin-batch base, the transcript window and with it the orch charge, and a span start taken from the earliest transcript when the seed has no `started_at`.
The flag's description in the doc comment says this, so an operator combining it with `-calibrate` knows the base and Merriam rows may come from an earlier run.

### D2 — readers: reject an unquoted parenthesis; globs and flags otherwise stay free, and the reason is documented

A reader segment (`readOnlyCommands`: `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq`, `cut`) is read-only only when it holds no unquoted `(` or `)`.
Under zsh a glob qualifier such as `cat *(e:'touch x':)` or `ls *(+fn)` runs shell code during expansion, and the classifier already assumes bash or zsh with any glob option set; `lyx`, `git` and `sed` words already reject an unquoted parenthesis through the shell-word split.
Readers are not put through that split, since it also rejects `$` and backslashes and would make reads like `grep 'a\.b' x` non-read-only.

Apart from that, readers keep unquoted globs and free flags.
The `doc.go` paragraph states why: no flag of a listed reader writes a file or runs a command, so a glob that expands into a flag-shaped name can do nothing a typed flag could not.
It states the admission rule for the list: a reader joins it only if none of its flags writes or executes; a reader that blocks, such as `tail -f`, writes nothing and is admitted.
It states the git reason: an unquoted glob is rejected except after the `--` of `log`, `show`, `diff` and `status`, because git reads every word after `--` as a pathspec, so an expanded name is a path and never a flag.
For every other rule it points at `IsReadOnlyCommand`'s comment rather than restating it.

Rationale: the classifier bounds globs in `lyx`, `git` and `sed` words because the shell can expand a glob into a name that spells a flag, and those commands have writing flags.
The readers have none (checked against the GNU flag sets of each, where the only file-naming flags, such as `grep -f`, `wc --files0-from` and `jq --rawfile`, read), so bounding their globs while their flags stay free guards nothing, and bounding their flags would need a per-reader allowlist that turns everyday reads such as `grep -rn x --include=*.go internal` non-read-only.

Bound: the change only narrows what is read-only; a reader command with an unquoted parenthesis now fails the read-only check and the webster audit treats it as any other non-read-only command.

### D3 — keep `fabricvocab.BareTokenInRootDoc`; remove the test-local `ReplaceAll`

`fabricVocabularyFailures` drops the `recordsRepositoryName` constant and the `strings.ReplaceAll` on root-file text.
Root files keep their bare-token check through `fabricvocab.BareTokenInRootDoc`.
`fabricVocabularyFailures`'s own doc comment keeps naming `loomyard-weft` as the name root files may spell.

Rationale: `BareTokenInRootDoc` lives in the owner package, has its own test (`root_doc_admits_records_repo_name_only`), and matches the name as a whole word.
The two differ only for an occurrence glued to a word character, such as `_loomyard-weft`; every occurrence in `CLAUDE.md` and `README.md` sits between brackets, slashes or parentheses, so the scan admits the same text.
The stripped text also fed the host-phrase check, but root files are outside its scope, so that path changes nothing.

## Constraints

- `PATTERN-fabric-vocabulary`: the bare-token scan still fails on every side token in a root file other than the records repository's name.
- `PATTERN-comment-line-breaks`: new Go comments, including the `doc.go` paragraph, are one sentence per line; neighbouring fixed-column text in `doc.go` is left as it is.
- `PATTERN-test-economy`: tests extend existing tables and tests; no new top-level test unless an existing one cannot carry the case.
- `PATTERN-test-speed`: the tokencount flag test sits at the lowest fitting tier.
- `PATTERN-documentation-lifecycle`: the classifier's design lives in `internal/fabricengine/doc.go`.

## Auto-mode assumptions

- The board entry's "add a flag … if it shows" is read as "add the flag now": the brief says to guard the cutoff, and the board has no `tokencount-full-runs` entry to wait on.
- `-no-cutoff` also lifts the shed span's `Older` clamp, since that clamp reads the same tag date.

## Open risks

- A future reader added to `readOnlyCommands` with a writing flag (for example `sort -o`) would be admitted unchecked; the documented admission rule and the test rows are the only guard.

## Acceptance criteria

- With `-no-cutoff`, a run whose project directory holds a transcript starting before the previous tag's date counts it: `LeftOut` is 0 and no "Session transcripts left out" line appears.
- With `-no-cutoff`, the report header carries the cutoff-off line; without it, the line is absent.
- Without the flag, the existing left-out behaviour for finished and in-flight runs is unchanged.
- With `-no-cutoff`, a shed span whose seed starts before the older tag's date is not clamped to that date.
- `IsReadOnlyCommand` returns false for `cat *(e:'touch x':)`, `ls *(+fn)` and `grep x (a)`, and true for `cat *`, `grep -rn x --include=*.go internal`, `jq '.a | map(.b)' f.json` and `git diff -- *.go`; it returns false for `git diff *.go`; rows already in the table may meet these, and the existing rows still pass.
- `internal/fabricengine/doc.go` names the reader rationale, the reader-list admission rule and the git pathspec reason for the `--` exception.
- After D3, `fabricVocabularyFailures` reports no failure on the current tree; in its fixture table a `README.md` holding `[loomyard-weft](https://github.com/Knatte18/loomyard-weft)` passes, and the same text plus a bare `weft` fails.

## Notes for the plan writer

- The bound reaches `CountRun` through `runBound(finished[slug], running)` in `tools/tokencount/main.go`; the shed clamp is `ShedRecord.Older`, set in `readShedRecord` in `tools/tokencount/shed.go`.
- The end-to-end header check can extend the existing integration test in `tools/tokencount/time_integration_test.go`.
- The `IsReadOnlyCommand` table lives in `internal/fabricengine/refscanner_test.go`; the reader check is in `readOnlySegment`, and `blankQuoted` gives the quote-blanked view.
- The fixture table is `TestFabricVocabularyFailures_Fixtures` in `internal/lyxcwd/vocabscan_test.go`.
- The `doc.go` paragraph fits beside the existing `RefScanner` and `ReferenceRule` paragraphs.
