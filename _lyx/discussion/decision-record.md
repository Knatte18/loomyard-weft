# Decision record — cleanup-1012

## Goal

Three independent cleanups from recent PR-Gate reviews:

1. `tools/tokencount` gets an operator switch that turns off the archive-tag date cutoff, so a skewed tag date cannot drop sessions with no way to recover them.
2. The read-only command classifier in `internal/fabricengine/refscanner.go` closes reader paths that can run code without a visible substitution marker by rejecting, in a reader segment, an unquoted parenthesis such as a zsh glob qualifier and any `$` not followed by whitespace, a double quote or the segment's end.
   The dollar rule covers expansions such as `${x@P}` and the ANSI-C quote `$'…'`, which hides the rest of a command from the quote scanner.
   `internal/fabricengine/doc.go` documents why reader globs and flags need no other bound and why git admits an unquoted glob after `--`.
   The brief's "bound audit reader globs" is met by the parenthesis and dollar rules, not by a general glob or flag bound on readers; D2 gives the reason.
3. `internal/lyxcwd/enforcement_test.go` keeps one of its two `loomyard-weft` carve-outs, with no change in what the fabric-vocabulary scan admits.

## Scope

In:

- A `-no-cutoff` flag on `tools/tokencount` and its description in the `tools/tokencount/main.go` doc comment, in the same commit.
- In `IsReadOnlyCommand`, a reader segment holding an unquoted `(` or `)`, or a `$` the dollar rule of D2 rejects, is not read-only; its doc comment says so.
- A `doc.go` paragraph in `internal/fabricengine` with the reader rationale, the reader-list admission rule and the git pathspec reason, pointing at `IsReadOnlyCommand` for the full rule set.
- New rows in the existing `IsReadOnlyCommand` test table and in the existing `fabricVocabularyFailures` fixture table.
- Removing the `recordsRepositoryName` constant and its `strings.ReplaceAll` from `fabricVocabularyFailures`.

Out:

- Any change to which commands `IsReadOnlyCommand` admits beyond the parenthesis and dollar rules, including teaching the quote scanner ANSI-C quoting.
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
With it, a slug's earlier runs that share its project directory feed every per-run figure the cutoff gates: tokens, cost, working intervals, the session count, the Webster forks, the Merriam starts, the begin-batch base, the transcript window and with it the orch charge, a span start taken from the earliest transcript when the seed has no `started_at`, and the run's plan.
The plan is read from the newest plan commit before the run's first Webster fork, so with the flag an earlier run's fork can move that point back and select an earlier run's plan, or none.
With the flag, a main-session transcript with no timestamped user or assistant line is counted too, since the zero bound skips the start check.
The flag's description in the doc comment says all of this, so an operator combining it with `-calibrate` knows the base and Merriam rows may come from an earlier run, that the plan, and with it the peak table, the start table's base and the identity card counts, may come from an earlier run, or that the run may be listed with no plan commit before the first fork.
The same description says an untimestamped transcript is counted, replacing the sentence that it is left out when a bound exists.

### D2 — readers: reject an unquoted parenthesis and a dollar expansion; globs and flags otherwise stay free, and the reason is documented

A reader segment (`readOnlyCommands`: `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq`, `cut`) is read-only only when it holds no unquoted `(` or `)` and no `$` the dollar rule below rejects.
Under zsh a glob qualifier such as `cat *(e:'touch x':)` or `ls *(+fn)` runs shell code during expansion, and the classifier already assumes bash or zsh with any glob option set; `lyx`, `git` and `sed` words already reject an unquoted parenthesis through the shell-word split.
Readers are not put through that split, since it also rejects `$` and backslashes and would make reads like `grep 'a\.b' x` non-read-only.

A dollar expansion runs code without any substitution marker: bash's `${x@P}` expands a prompt string holding `$(…)`, an arithmetic subscript such as `${a[y]}` evaluates the variable's value, and under zsh with `GLOB_SUBST` `cat $x` expands a qualifier glob held in `x`, as zsh's `$~x` does with no option set.
So the dollar rule denies by default: a reader segment is also not read-only when a `$` that its view with only single-quoted spans blanked keeps (the view the substitution scan already uses) is followed, in the segment's raw text, by anything other than whitespace, a double quote or the end of the segment.
The follower is read from the raw text because the view blanks a single quote itself, so `$'` would otherwise read as a `$` before a space.
Every parameter and arithmetic expansion is rejected, whatever follows the `$`: a name, a digit as in `$1`, a special parameter as in `$@`, a brace or bracket, zsh's `$~x`, `$=x`, `$^x` and `$+x`, and the ANSI-C quote `$'…'`, inside double quotes as well as outside.
A regex anchor stays read-only: in `grep a$ f` and `grep "^a$" f` the `$` is followed by whitespace or a double quote, and in `grep 'a$' f` it is blanked.
The `IsReadOnlyCommand` doc comment's sentence that an expansion stays non-read-only becomes true for readers by this rule.

The other code-running and writing vectors in a reader segment are bounded at command level before any segment is judged: the command and process substitution markers (`$(`, a backtick, `<(`, `>(`), every output redirection but the two dropped stderr forms `2>&1` and `2>/dev/null`, and backgrounding are rejected command-wide.
Those scans, and the split into segments, rely on the dollar rule's rejection of `$'`: `blankQuotedSpans` knows no ANSI-C quote and treats the `'` of a `\'` inside `$'…'` as closing the span, so in `cat $'\'' $(touch x) f` the next `'` opens an unterminated span that hides the substitution, and any later separator or redirection, from every command-wide scan.
The segment holding that `$'` is then rejected: a reader segment by the dollar rule, and a `lyx`, `git` or `sed` segment by the shell-word split, which rejects any `$`.
Input redirection with `<` stays admitted, because it only reads.

Apart from that, readers keep unquoted globs and free flags.
The `doc.go` paragraph states why: no flag of a listed reader writes a file or runs a command, so a glob that expands into a flag-shaped name can do nothing a typed flag could not.
It states the admission rule for the list: a reader joins it only if none of its flags writes or executes; a reader that blocks, such as `tail -f`, writes nothing and is admitted.
It states the git reason: an unquoted glob is rejected except after the `--` of `log`, `show`, `diff` and `status`, because git reads every word after `--` as a pathspec, so an expanded name is a path and never a flag.
For every other rule it points at `IsReadOnlyCommand`'s comment rather than restating it.

Rationale: the classifier bounds globs in `lyx`, `git` and `sed` words because the shell can expand a glob into a name that spells a flag, and those commands have writing flags.
The readers have none (checked against the GNU flag sets of each, where the only file-naming flags, such as `grep -f`, `wc --files0-from` and `jq --rawfile`, read), so bounding their globs while their flags stay free guards nothing, and bounding their flags would need a per-reader allowlist that turns everyday reads such as `grep -rn x --include=*.go internal` non-read-only.

Bound: the change only narrows what is read-only; a reader command with an unquoted parenthesis or a `$` the dollar rule rejects now fails the read-only check and the webster audit treats it as any other non-read-only command.

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

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d2-dollar-rule-follower-enumeration (class design, severity MEDIUM, open in rounds 2)
  - d2-reader-dollar-expansion-unbounded (class design, severity BLOCKING, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- With `-no-cutoff`, a run whose project directory holds a transcript starting before the previous tag's date counts it: `LeftOut` is 0 and no "Session transcripts left out" line appears.
  A main-session transcript with no timestamped user or assistant line is counted with the flag and left out without it when a bound exists.
- With `-no-cutoff`, the report header carries the cutoff-off line; without it, the line is absent.
- Without the flag, the existing left-out behaviour for finished and in-flight runs is unchanged.
- With `-no-cutoff`, a shed span whose seed starts before the older tag's date is not clamped to that date.
- `IsReadOnlyCommand` returns false for `cat *(e:'touch x':)`, `ls *(+fn)`, `grep x (a)`, `cat ${x@P}`, `head -n ${a[y]} f`, `cat $x`, `cat $~x`, `cat $1`, `cat $@`, `cat "${x@P}"` and `cat $'\'' $(touch x) f`, and true for `cat *`, `grep -rn x --include=*.go internal`, `jq '.a | map(.b)' f.json`, `grep a$ f`, `grep 'a$' f`, `grep "^a$" f` and `git diff -- *.go`; it returns false for `git diff *.go`; rows already in the table may meet these, and the existing rows still pass.
- `internal/fabricengine/doc.go` names the reader rationale, the reader-list admission rule and the git pathspec reason for the `--` exception.
- After D3, `fabricVocabularyFailures` reports no failure on the current tree; in its fixture table a `README.md` holding `[loomyard-weft](https://github.com/Knatte18/loomyard-weft)` passes, and the same text plus a bare `weft` fails.

## Notes for the plan writer

- The bound reaches `CountRun` through `runBound(finished[slug], running)` in `tools/tokencount/main.go`; the shed clamp is `ShedRecord.Older`, set in `readShedRecord` in `tools/tokencount/shed.go`.
- The end-to-end header check can extend the existing integration test in `tools/tokencount/time_integration_test.go`.
- The `IsReadOnlyCommand` table lives in `internal/fabricengine/refscanner_test.go`; the reader check is in `readOnlySegment`, and `blankQuoted` gives the quote-blanked view.
- The fixture table is `TestFabricVocabularyFailures_Fixtures` in `internal/lyxcwd/vocabscan_test.go`.
- The `doc.go` paragraph fits beside the existing `RefScanner` and `ReferenceRule` paragraphs.
