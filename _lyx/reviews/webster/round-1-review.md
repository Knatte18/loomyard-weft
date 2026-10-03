---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/testkit/scankit/scankit.go:275
    summary: AssertImportAllowlist has no stale-entry detection, so the converted import-allowlist guards of cards 8 and 9 lack the tightening both cards require
  - id: F2
    severity: MEDIUM
    location: cmd/lyx/clitree_test.go:117
    summary: the walk's bogus check accepts any error envelope, dropping the unknown-subcommand pin the deleted tests carried, so config bogus passes while refusing for an unrelated reason
  - id: F3
    severity: LOW
    location: cmd/lyx/clitree_test.go:47
    summary: a cliTreeFindings entry skips its whole subtest and goes stale only when the group disappears, so a fixed finding is never reported, and the board notes bare entry misdescribes the output
  - id: F4
    severity: LOW
    location: contracts/stencils/claims_test.go:219
    summary: the dropped-concepts mustNot claims became case-sensitive, so a capitalised Chain or Oversized now passes where the moved test lowercased the text
  - id: F5
    severity: LOW
    location: cmd/lyx/tmuxisolation_test.go:50
    summary: compilesUnder satisfies each file's constraint on any platform independently and ignores GOOS filename suffixes, so a TestMain compiling on one platform covers tests compiling only on another
  - id: F6
    severity: LOW
    location: cmd/lyx/tmuxisolation_test.go:116
    summary: the kit-call check keys on identifier names, so an aliased tmuxkit import is missed and the unqualified Main exemption holds for any package named tmuxkit, not only the kit's own directory
  - id: F7
    severity: LOW
    location: internal/battencli/commitstatus.go:108
    summary: new and changed Go comments across the diff are fixed-column wrapped, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F8
    severity: LOW
    location: _lyx/plan/00-overview.md
    summary: the test-accounting Shared Decision's before-and-after leaf test counts are recorded nowhere in the branch
  - id: F9
    severity: NIT
    location: internal/shedverbs/step_test.go:24
    summary: touched test doc comments restate the test name and count the envelope keys, against D22
  - id: F10
    severity: NIT
    location: contracts/stencils/claims_test.go:430
    summary: TestSectionOf does not cover the heading-with-trailing-clause match card 24 added to sectionOf
---

Range: `_lyx/shed/test-suite-derived-guards/status.json` gives `product.parent` `main`, and `_lyx/loom/rework/` does not exist, so this round reviews `git diff $(git merge-base main HEAD)..HEAD` (merge base `22e46970e`, cards 1–34).
`go build ./...`, `go vet ./...` (untagged, `integration`, `smoke`), `go test ./...` and `go test -tags integration ./...` all pass on HEAD.

### [MEDIUM] Import-allowlist guards in cards 8 and 9 carry no stale detection

**Location:** `internal/testkit/scankit/scankit.go:275` (`AssertImportAllowlist`), called by every `seam_enforcement_test.go` cards 8 and 9 convert.

**Issue:** Cards 8 and 9 each state "Every converted scan gains a floor where it had none and stale detection on its allowlist", and the overview's `scan-conversion-protocol` and D2 name stale-entry detection as one of the two tightenings for every converted scan.
`AssertImportAllowlist` checks only that each import is stdlib or allowed; an allowed path no production file imports is never reported.
Both commit bodies say "No stale entries found" while also saying the import allowlists "carry no stale detection", so the first claim was never checked.
A `go list` comparison shows no allowlist in cards 8 or 9 is stale today, so no verdict changes now, but the required tightening is missing.
Card 7 keeps its leaf guards untightened by its own text ("the check each guard runs today is kept, not tightened"), and `internal/gitkit`'s allowlist does carry three entries nothing imports (`lyxcwd`, `lyxdirs`, `weftname`), so the tightening cannot be applied by changing `AssertImportAllowlist` for every caller.

**Fix:** Add a stale-checking form of the assertion to `scankit`, with the pure helper reporting the allowed paths no file imports, and switch the import guards of cards 8 and 9 to it, leaving card 7's guards on the untightened form.
Cover the stale case in `scankit_test.go`.

### [MEDIUM] The CLI walk's bogus check accepts any refusal

**Location:** `cmd/lyx/clitree_test.go:117` (`envelope.RequireErr(t, out.String(), "")`).

**Issue:** The deleted `TestMountedUnknownSubcommand` asserted `envelope.RequireErr(t, out, "unknown subcommand")`, and the per-CLI copies card 12 deleted asserted `"unknown"`.
The walk that replaces them accepts any `ok:false` envelope.
Running the built binary from a non-git directory shows `lyx config bogus` exits 1 with `{"error":"not a git repository","ok":false}`: the unknown-subcommand path is never reached, yet the walk passes it and `cliTreeFindings` has no `config#bogus` entry.
Card 11 says a group whose bogus invocation differs "is a finding to report, not a case to special-case silently", and the moved assertion lost its pin.

**Fix:** Require `"unknown subcommand"` in the bogus envelope and add the groups that then fail to `cliTreeFindings` with an accurate reason.

### [LOW] Known-finding exemptions never notice a fix and one reason is inaccurate

**Location:** `cmd/lyx/clitree_test.go:47` (`cliTreeFindings`) and the two `t.Skip("known finding")` sites.

**Issue:** An entry skips the whole subtest, so for `board notes#bogus` the exit-code check (which passes today) is skipped along with the failing single-envelope check.
An entry only goes stale when its group disappears: if a group's behaviour is fixed, the skip keeps hiding it and the entry lives on.
The `board notes#bare` reason says the group refuses "instead of listing its verbs"; the binary prints the error envelope, then the full help listing, and exits 1, so it does list its verbs.

**Fix:** Run the subtest's assertions for an exempted group as well, and fail with "finding fixed, remove the entry" when they all pass, so an entry is stale once its finding is fixed.
Reword the `board notes#bare` reason to what the group does: emits the `not initialized here` envelope ahead of its listing and exits 1.

### [LOW] Dropped-concepts claims lost their case folding

**Location:** `contracts/stencils/claims_test.go:219`, `:225`, `:227`, `:332`.

**Issue:** The deleted `TestTemplates_NoDroppedBatchConceptsRemain` lowercased the template text before checking `"oversized"` and `"chain"`.
The claims table matches case-sensitively, so a sentence opening with `Chain` or `Oversized` now passes.
The commit body discloses this, but the move weakens a `mustNot` claim rather than carrying it over.
The scope change (stencil by stencil instead of the composed fork and recovery text) covers the same files, because the fork and recovery templates are the two prefixes composed with `webster-body-implementer.md`, all three of which carry the claims.

**Fix:** Give `claim` a case-insensitive form for `mustNot` and use it for the two dropped-concept words.

### [LOW] Tmux isolation scan treats platforms per file

**Location:** `cmd/lyx/tmuxisolation_test.go:50` (`compilesUnder`) and its use at `:197`.

**Issue:** For each tag set, a file counts as compiling when any platform profile satisfies its constraint, and a `TestMain` file counts on the same terms, independently.
A package whose tagged tests compile only on Linux and whose `TestMain` compiles only on Windows passes, though on Linux no `TestMain` runs.
File names carrying an implicit `GOOS` (`_windows_test.go`, `_linux_test.go`, which the tree already uses in `internal/proc` and `internal/reedcli`) are treated as compiling everywhere.
No package in the tree hits either case today.

**Fix:** Evaluate each (tag set, platform) pair, requiring a compiling `TestMain` on every pair where any test file compiles, and fold a file name's `GOOS` suffix into its constraint.
Add a fixture case for each.

### [LOW] Kit-call detection keys on identifier names

**Location:** `cmd/lyx/tmuxisolation_test.go:116` (`declaresTestMain`).

**Issue:** The qualified check matches any selector `tmuxkit.Main`, so a file importing the kit under an alias fails the scan, and another package imported as `tmuxkit` would satisfy it.
The unqualified exemption is granted to any file whose package clause is `tmuxkit`, so a second package named `tmuxkit` elsewhere in the tree, with its own `Main`, would pass; card 34 grants the exemption to the kit's own tests only.

**Fix:** Resolve the local name bound to the kit's import path from the file's imports, and grant the unqualified form only to files in `internal/testkit/tmuxkit`.

### [LOW] Fixed-column wrapping in new and changed comments

**Location:** `internal/battencli/commitstatus.go:108` and the other sites below.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks in new and changed Go comments.
Many comments the branch adds or rewrites break mid-sentence at a column instead, among them:
`internal/battencli/commitstatus.go` (the rewritten `newCommitStatusSeam` doc and the moved marker-write comment);
`cmd/lyx/tmuxisolation_test.go` (file comment, `tmuxIsolationFailures`, the scan test's doc);
`internal/testkit/scankit/scankit.go` (package doc, `importViolations`, `AssertImportAllowlist`);
`internal/loomcli/bootstrap_test.go` (`driverScanMinFiles`, `scanRepoForDriverFieldReads`);
`internal/loomrecipe/coverage_guard_test.go` (`TestCoverageGuard_EveryLoomRowHasAnEngine`);
`internal/shedcli/cli_test.go` (file comment);
`internal/battencli/commitstatus_test.go`, `internal/planglyph/*_enforcement_test.go`, `internal/planparser/shape_*test.go`, `internal/fabricengine/mergevocab_test.go`, `internal/lyxcwd/docslink_test.go`, `internal/lyxcwd/enforcement_test.go`, `contracts/stencils/citation_enforcement_test.go`, `internal/configreg/*_test.go`, `internal/shedverbs/*_test.go`, the seam guards' `TestToldGeometryInvariant_AllowlistOnly` docs and `tools/sandbox/pathresolve_guard_test.go`.

**Fix:** Rewrap each added or rewritten comment block to one sentence per line, breaking long sentences only at independent-clause boundaries.

### [LOW] Leaf test counts not recorded

**Location:** `_lyx/plan/00-overview.md`, Shared Decision `test-accounting`.

**Issue:** The decision requires leaf test counts (top-level tests without subtests plus every subtest, from `go test -json`) to be recorded before card 1 and after the last card.
No commit body and no `_lyx/webster` artifact carries them.

**Fix:** Measure both counts (the merge base and HEAD, untagged and `integration`) and record them in the fix commit's body.

### [NIT] Test doc comments restate names and count keys

**Location:** `internal/shedverbs/step_test.go:24` and `:74`.

**Issue:** D22: a touched test drops a doc comment that restates its name, and no comment counts rows.
`TestStepEnvelope_KeySetIsExactlySixteen asserts the envelope's key set is exactly the sixteen documented keys` restates the name and counts the keys;
`TestStepEnvelope_FieldMapping tables the routing shapes, asserting each field's mapping onto the envelope` restates the name.

**Fix:** Delete both doc comments.

### [NIT] Trailing-clause heading match is untested

**Location:** `contracts/stencils/claims_test.go:430` (`TestSectionOf`).

**Issue:** Card 24 extended `sectionOf` to accept a heading followed by a trailing clause, but `TestSectionOf` only covers exact headings and a missing one.

**Fix:** Add a case where the claim names the leading part of a longer heading.

## Settled without a finding

- **Card 6 owner maps.** Card 6 names "owner sets" and "allowlists" as separate things to keep verbatim, so "stale detection on any allowlist it carries" covers `enforcementAllowlist` and `docsLinkAllowlist`, which both have it.
  `weftnameImportOwners` mirrors the owner set CONSTRAINTS.md's Fabric Vocabulary Invariant names, and the commit's reason holds: `internal/hubforge` is listed but imports no `weftname`.
- **Card 10 shallow walks.** The `planglyph` chokepoint, `planparser` shape and `shedcli` table scans walked recursively at the merge base and still do.
  None of those directories holds a Go-bearing subdirectory (`internal/planparser/testdata` holds no `.go` file), so the scanned set and verdicts are unchanged.
  Every scan that read one directory with `os.ReadDir` (`planglyph` status, `fabricengine`, `gitrepo`, the `reedengine` scans) is now `Shallow`.
- **Card 28.** The dispositions and their order are unchanged, commit and log texts are byte-identical through the caller-supplied prefixes, `battencli`'s marker write runs only after a successful commit and before the push, and `statuscommit` imports only `fmt` and `internal/logger`.
- **Card 30.** `Main`'s sweep lists only sockets under its own directory's `tmux-<uid>/`, the path check uses the 61-byte key bound (with `TestMaxKeyBytes_HoldsReedServerName` pinning it against `reedengine.ServerName`), a missing `tmux` skips only the sweep, Windows runs the tests unchanged, and `TMUX` and `TMUX_PANE` are unset.
  The `hermeticenv_test.go` directory entry and the `spawnobservability_test.go` entry for `tmuxkit.go` give accurate reasons.
- **Per-card mechanical checks.** Card 16's rename leaves no `recognizedFormat` anywhere outside `_lyx`, and the doc comment opens with `RecognizedFormat`.
  The helpers the deleted files of cards 11 and 14 defined (`collectMissingShorts`, `assertPath`, `namedPath`, `durableSet`, `transientSet`) have no remaining callers.
  The created kits (`scankit`, `plankit`, `stencilkit`, `tmuxkit`) and `statuscommit` have no predecessor under `internal/testkit` or elsewhere at the merge base.
