---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-round-gate-headroom.md
    summary: card 6 changes what a told merge base reader means but leaves three statements of the old meaning standing, two of them in files it does not target
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/10-pattern-overview-trim.md
    summary: the uniform 200-byte line ceiling, link included, forces 83 to 120 byte cuts on the six entries that gain a new link, pressing against D4's no-rule-change requirement
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/03-lint-pattern-verb.md
    summary: card 3 renames TestLintCommentsCmd_OverARepository on an Edit card, a rename no decision licenses and whose Edit glyph names a symbol the card removes
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-pattern-headroom.md
    summary: card 2's ImpactSummary says the oversize message's only reader is its test and that it gains one word, contradicting D5's Publish failure record reader and the added byte count
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/01-gofmt-guard.md
    summary: card 1 says its in-memory table pins three outcomes and then lists four rows
---

### [MEDIUM] Card 6 leaves the old reader-selects-form meaning stated in three places

**Location:** `_lyx/plan/06-round-gate-headroom.md`, the `shedrecipe.Env` sentence and the Edit list.

**Issue:** Card 6 parts the merge base reader from the gate's form: after it, a non-nil `VerifyMergeBase` with `VerifyWholeDiff` false is the round form plus the headroom lint.
Three statements of the old meaning survive the card as written:

- `contracts/specs/shed-recipe-spec.md:62` says the `verify` gate "takes the whole-diff form when `Env.VerifyMergeBase` is set and the round form when it is nil".
  The card does not target this file, so the normative spec becomes false for every loom pair run.
- `internal/shedrecipe/recipe.go:61`, the `VerifyMergeBase` field's comment, says "Nil selects the gate's round form; set selects its whole-diff form."
  `internal/shedrecipe#Env` is a target, but the Intent says only that `VerifyWholeDiff` is "documented beside `VerifyMergeBase`", never that this sentence is rewritten.
- `contracts/recipes/loom-recipe.yaml:309-314`, the Webster-Burler row's `gates` comment, says the gate "first lints the comments the round added, then runs the impacted-set command".
  After card 6 the headroom lint runs first, and the file is not a target.

CLAUDE.md requires docs to land in the same commit, and the card already rewrites the matching text in `NewVerifyGate`'s doc, `internal/loomshed/doc.go` and `resolveGateSpec`'s doc, so these three are the gaps in that same sweep.

**Fix:** Add `contracts/specs/shed-recipe-spec.md` and `contracts/recipes/loom-recipe.yaml` to card 6's `**Edit:**` list.
In the Intent, state that the spec's `verify` sentence and the `VerifyMergeBase` field comment say the reader is the merge base for the headroom lint, or for both lints under `VerifyWholeDiff`, and that `VerifyWholeDiff` selects the form, and that the recipe comment names the headroom lint ahead of the comment lint.
Both files exist, so `path-missing` admits the addition.

### [LOW] Card 10's per-line ceiling squeezes the six link-gaining entries hardest

**Location:** `_lyx/plan/10-pattern-overview-trim.md`, Intent sentence "Each trimmed line ends at most 200 bytes long, its link included".

**Issue:** The card's choice of entries is consistent with D4: I recomputed the set of entry lines at 230 bytes or longer, and it is exactly the 27 background files under `**Prosa:**` plus the 6 files under `**Create:**`, under either reading of the `wc -c` count (with or without the newline), and every file the card writes is a declared target.
Trimming every one of the 33 to 200 bytes frees 1461 bytes against the 1217 the 16217-byte overview must lose, so the target is reachable.
But the single 200-byte ceiling counts the new 49 to 61 byte link on the six entries that have no background file today.
Those lines must lose 83 to 120 bytes each: `PATTERN-push-both-sides` goes from 267 bytes to 200 with a 53-byte link added, leaving about 115 bytes for its trigger and its requirement, which today run about 235 bytes and carry five distinct obligations (code branch with it, rebase-free, under the push lock, never left to an agent, prime pushes records-only).
D4 lets detail move to the background file but requires that each entry keep its requirement and that no rule change, so the ceiling pushes an implementer toward dropping a clause of the rule itself on exactly these lines.

**Fix:** State the bound as the 15000-byte total plus the per-line rule only for lines that already carry a link, and give the six link-gaining lines their own ceiling, for example 230 bytes link included; with the other 27 lines at 200, that still frees about 1280 bytes, above the 1217 needed.
Alternatively keep 200 and say in the Intent that a requirement clause never leaves the line, only mechanism, example or history.

### [LOW] Card 3 renames the comment lint's scenario test without a license

**Location:** `_lyx/plan/03-lint-pattern-verb.md`, "The test is renamed `TestLintVerbs_OverARepository` in place".

**Issue:** No decision asks for the rename; the record's `PATTERN-test-economy` constraint asks only that the verb's tests extend the test that covers `lint-comments`'s envelope.
The card lists `internal/loomcli#TestLintCommentsCmd_OverARepository` under `**Edit:**` and then removes that symbol, so its own Edit glyph names nothing after the card runs, and the rename travels outside the plan's `Rename` type and its mechanic, which card 5 uses for the comparable `darnMergeBase` rename.

**Fix:** Drop the rename: keep the test's name and update only its doc comment and the file header to name both verbs.

### [LOW] Card 2's ImpactSummary misstates the oversize message's readers

**Location:** `_lyx/plan/02-pattern-headroom.md`, `**ImpactSummary:**`.

**Issue:** It says `Check`'s oversize message gains "one word" and that "the message's only reader is the test asserting it".
The Intent adds the number of bytes over the cap, a clause, not a word.
D5's rationale is that this message is read in Publish's failure record and in every full verify's `TestOverview_RealFilePassesCheck` output, which is the reason the change exists.
The ImpactSummary must carry a true blast-radius conclusion, and this one contradicts the decision it implements.

**Fix:** Reword it to: adds three exported symbols to the leaf; `Check`'s oversize message gains the bytes over the cap, read by `TestCheck`, by `TestOverview_RealFilePassesCheck`'s failure output and through it by Publish's failure record; no code parses the message.

### [NIT] Card 1 counts its in-memory rows wrong

**Location:** `_lyx/plan/01-gofmt-guard.md`, "which pin those three outcomes".

**Issue:** The sentence names three outcomes and then lists four rows, the fourth being the import-order case.

**Fix:** Say "which pin these outcomes", or "four".

## Focus verdicts

**Card 6, `loomCLI.wire`: not a defect.**
The planned wiring does one `os.Stat` of `fabricengine.OriginRecordPath(location)`, which is `filepath.Join(l.AnchorPath(), OriginRecordRel())` (`internal/fabricengine/origin.go:50`) and opens or creates nothing.
`fabricengine.ReadOrigin`, which calls `originLockPath` and so `ensureWeftLockDirAt`, is reached only through `readRecordedParentBranch` inside the lazy reader, at gate time, as darn's reader already does.
`hubLocation` (`internal/loomcli/wiring_test.go:130`) seeds `<anchor>/_lyx/config/` and no `_lyx/fabric/`, so the stat answers not-exist, never another error, and the reader stays nil; every wire test built on it passes unchanged.

**Card 6, `TestWire_DarnRecipe` and the other wire tests: not a defect.**
Because wire only stats the path, a fixture that writes any file at `fabricengine.OriginRecordPath(loc)` satisfies the new loom-seed assertion that the reader is set, and the darn seed's assertion holds since `wireDarn` keeps setting the reader unconditionally.
The only other assertion on `VerifyMergeBase` in a wire test is the loom-seed step of the same test (`wiring_darn_test.go:80`).
`TestWire_EverySeamFilled` is untouched, since `envkit`'s `nilLegal` already skips `VerifyMergeBase` (`internal/testkit/envkit/envkit.go:40`).
`TestWire_WebsterParentBranchReadsPairOrigin` writes an origin record and so gets the reader after card 6, but it asserts nothing on `VerifyMergeBase`, and no `TestWire_Real_*` test does either.
The Edit list covers every wire test whose assertions change.

**Card 10: targets complete, and the entry choice matches D4.** The one concern is F2.

**Cards 1 and 3, new top-level tests: justified.**
Card 1 adds `TestGofmt_ModuleFilesAreFormatted`, the one untagged guard D7 licenses, with the `cmd/lyx` budget row raised by one as D7 states.
Card 3 adds no top-level test and extends `TestLintCommentsCmd_OverARepository`, the integration test that covers `lint-comments`'s envelope, as the record's `PATTERN-test-economy` constraint asks; the rename on that card is F3.
Outside the focus, card 8's `TestProcProbes_OwnProcess` is justified too: no existing `tmuxkit` test exercises the `/proc` probes in Tier 1.

## Checked and clean

- `gofmt -l .` lists exactly the six files card 1 names, and `gofmt -d` matches the card's description of each fix, including the doc-comment list item in `internal/pattern/doc.go` and the curly-quote rewrite in `posixShell.Quote`'s comment.
- `scankit`'s `skipDir` skips every directory whose name contains `testdata`, so the walk skips the only nested module, `internal/planglyph/testdata/glyphchain`.
- `gitexec.Run` returns git's stdout untrimmed, so card 3's `git show` read and its `os.ReadFile` read compare byte for byte apart from the CRLF case the card normalises.
- Exactly the six test files D8 names carry the `"/proc` literal outside `tmuxkit`, so card 9's Verify line checks the acceptance criterion.
- Only the Webster-Burler and Darn rows use the `verify` gate, so the headroom lint never runs at Publish, as D2's Bound requires.
- Card order holds: card 6 uses card 3's `patternlint` and card 5's `pairMergeBase`, card 9 uses card 8's probes, and card 10 appends after the background-file edits of cards 4, 6 and 7.
