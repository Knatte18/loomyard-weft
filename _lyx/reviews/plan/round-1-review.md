---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/02-planparser-renumber.md:65
    summary: No card creates the journal's scratch directory .lyx/plan, and RenumberIO has no MkdirAll, so the first insert-card or remove-card in any real worktree fails writing the journal; every planned test uses an existing temp dir and misses it
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-planparser-restore.md:26
    summary: RestoreRenumber holds journaled card names to cardSlugRe, which the plan format does not impose on existing slugs, so a journal left by renumbering a plan with any other slug cannot be restored
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/02-planparser-renumber.md:72
    summary: The prose-mention pattern matches only singular card N, so lines like this plan's own Applies to cards 2, 3, 4 are never reported, short of the decision record's every line that names a moved card's old number
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-planparser-restore.md:11
    summary: Card 3 adds RestoreRenumber as a plan-directory writer but leaves pattern/PATTERN-sole-parsers.md out of its Edit list, so the background's closed write-path list, which card 2 extends, ends false
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-loom-card-verbs.md:50
    summary: Card 4's bound paragraph claims all three verbs touch only the Card Index of 00-overview.md, but restore-renumber writes back whole journaled files and discards later edits, which card 4's Intent never states
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-planparser-renumber.md:74
    summary: No planned test carries the acceptance criterion that archive-*/ and round-N/prior-generation/ contents are untouched by all three verbs
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-loom-card-verbs.md:62
    summary: Card 4's verb tests re-assert behavior card 2's and card 3's planparser tests already pin, and both cards plan one success test per verb
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/04-loom-card-verbs.md:51
    summary: loom's pre-run reads a verb's first positional argument as a run-id, so a non-integer N such as .. or a/b is refused at arming, not by the position refusal card 4 promises, and card 4 does not exempt the verbs as reject is exempted
  - id: F9
    severity: LOW
    class: design
    location: _lyx/plan/04-loom-card-verbs.md:62
    summary: The lease test passes whenever the probe goroutine happens to reach its lock call after the verb releases, so a verb that released the lease early could still pass it
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/plan/02-planparser-renumber.md:55
    summary: The precheck refuses only a readable journal, which lets a verb whose journal is present but unreadable go on and overwrite the only restore record
---

### [BLOCKING] The journal's scratch directory is never created

**Location:** `_lyx/plan/02-planparser-renumber.md:65`, with `:50` (the seam) and card 4's `newRenumberDeps` at `_lyx/plan/04-loom-card-verbs.md:46`.

**Issue:**
`RenumberJournalPath` is `<anchor>/.lyx/plan/renumber-journal.json`, and `writeRenumberJournal` writes the temporary file "through `ops.WriteFile`, then `ops.Rename`s it into place".
`RenumberIO` holds only `ReadFile`, `WriteFile`, `Rename` and `Remove`, and no card creates `.lyx/plan`.
Nothing in the tree creates that directory today: `PlanScratchDir` is new, and the only `MkdirAll` on the verbs' path is `websterengine.AcquireStateMutation`'s, which creates `.lyx/webster` (`internal/websterengine/state.go:103`).
`os.WriteFile` into a missing directory fails with ENOENT, so in a fresh worktree every `insert-card` and `remove-card` call is refused at the journal write as `ErrRenumberIOFailed`.
That joins the raw-I/O row's "re-run the refused verb", and the re-run fails the same way.
Card 2's tests use "a temp dir" for the journal and card 4's use "a temp scratch dir"; both directories already exist, so no planned test reaches the failure.
Followed as written, the plan ships verbs that cannot complete a call.

**Fix:**
Card 2 creates the journal's parent directory before the temporary write, either through a `MkdirAll` field added to `RenumberIO` or as a named step of `writeRenumberJournal`.
A failure there joins the same raw-I/O path with the plan directory untouched.
One row of `TestRenumber_IOFailureReversed`, or one fixture in `TestInsertCard`, passes a `journalPath` whose parent does not exist yet, and card 4's verb test fixtures do the same through `newRenumberDeps`' real path shape.

### [MEDIUM] Restore's card-name rule is stricter than the plan format

**Location:** `_lyx/plan/03-planparser-restore.md:26`

**Issue:**
The decision record bounds `restore-renumber` to names that are "a card file name or `00-overview.md` at the top level of `planparser.PlanDir`".
Card 3 narrows "card file name" to `NN-<slug>.md` with `cardSlugRe` on the slug.
`cardSlugRe` is the rule `insert-card` holds a new slug to.
The decision record says the plan format has no slug rule of its own: `cardIndexLineRe` takes any `\S+`, and `cardFileName` is `%02d-<slug>.md` (`internal/planparser/parse.go:125`, `:370`).
`InsertCard` and `RemoveCard` never check existing slugs against `cardSlugRe`.
Their journal therefore lists names such as `05-Fix_flag.md` for a plan the gate accepts, and `restore-renumber` would refuse that journal as `ErrRenumberJournalInvalid`.
The refusal's way forward then deletes the only restore record.
`NN` is also two-digit shorthand for a number `%02d` renders as `100-…` past 99.

**Fix:**
Hold each journaled name to a top-level base name: no separator of either kind, not `.` or `..`, and either `00-overview.md` or a name that splits as `<digits>-<non-empty, non-whitespace slug>.md`, with no `cardSlugRe` check.
That still refuses every outside path card 3 lists.
`TestRestoreRenumber` gains a row whose journal names a card slug outside `cardSlugRe` and restores it.

### [MEDIUM] Prose mentions in plural or list form are not reported

**Location:** `_lyx/plan/02-planparser-renumber.md:72`

**Issue:**
D1 requires the envelope to list "every line in the current plan's files that names a moved card's old number in prose", with `card 8` and `Card 8` as examples.
Card 2's pattern `\b[Cc]ard\s+(\d+)\b` needs whitespace directly after `card`.
It misses `cards 2, 3, 4`, `Cards 6–8` and every later number in a list.
That form is the usual cross-reference in a plan's `## Shared Decisions`: this plan's own overview writes `**Applies to:** cards 2, 3, 4` five times (`_lyx/plan/00-overview.md:27`).
An insert before card 2 of this plan would report none of those lines, and they would silently point at the wrong cards.

**Fix:**
Widen the scan to `[Cc]ards?` followed by a list or range of numbers, reporting the line when any number in it is an old number of a moved card.
`TestInsertCard`'s prose-mention assertion gains an `Applies to: cards …` line in the fixture's overview.

### [MEDIUM] PATTERN-sole-parsers is left naming a closed write-path list that excludes RestoreRenumber

**Location:** `_lyx/plan/03-planparser-restore.md:11` (the `**Edit:**` list) against `_lyx/plan/02-planparser-renumber.md:78`.

**Issue:**
`pattern/PATTERN-sole-parsers.md` states planparser's write paths "and no others".
Card 2 edits that line to add `InsertCard` and `RemoveCard`.
Card 3's `RestoreRenumber` rewrites and removes card files and `00-overview.md`, so it is a third new write path.
Card 3 updates `doc.go`'s write-path list (`:35`) but not the PATTERN background, so after card 3 the authoritative pattern file names a closed set that omits a live writer.

**Fix:**
Add `pattern/PATTERN-sole-parsers.md` to card 3's `**Edit:**` list, and have its Intent name `RestoreRenumber` beside the two card 2 adds.

### [MEDIUM] Card 4 misstates restore-renumber's bound and omits what it lets through

**Location:** `_lyx/plan/04-loom-card-verbs.md:50`

**Issue:**
Card 4 introduces all three verbs.
Its single bound paragraph says "these verbs … touch only the current generation's card files and the Card Index of `00-overview.md` …, never … the overview's frontmatter or any other overview section".
That holds for `insert-card` and `remove-card` but not for `restore-renumber`.
`restore-renumber` writes back every journaled file's full pre-call bytes (D2, card 3 `:29`), including all of `00-overview.md`, and removes every journaled created name.
D2 names the cost: it "discard[s] any edit made to them since the interrupted call".
D2 also bounds what it may write to the journal's top-level names and makes it refuse an undecodable journal.
Card 4's Intent states none of that for the verb it introduces.
Card 3 states the cost only for the planparser function's doc comment.
The attack-surface rule binds the card that introduces the verb, and card 4's sentence is wrong about the third verb rather than silent.

**Fix:**
Split card 4's bound paragraph into the insert/remove bound as written and a `restore-renumber` bound.
The second says the verb holds the same lease, writes and removes only the base names the journal lists at the plan directory's top level, refuses an absent or undecodable journal or one naming any other path, and restores whole pre-call files, discarding any edit made to them since the interrupted call.
Card 4's `doc.go` section states the same cost.

### [MEDIUM] The archive-untouched acceptance criterion has no test

**Location:** `_lyx/plan/02-planparser-renumber.md:74`, `_lyx/plan/03-planparser-restore.md:31`, `_lyx/plan/04-loom-card-verbs.md:62`

**Issue:**
The decision record's acceptance criteria include "`archive-*/` and `round-<N>/prior-generation/` contents are untouched by all three verbs".
Card 3's `TestRestoreRenumber` covers a journal naming such a path, but no test runs `InsertCard`, `RemoveCard` or a successful `RestoreRenumber` against a plan directory that holds those subdirectories.
The byte-for-byte comparisons in cards 2 and 3 are stated over "the directory's files", and their `plankit.Write` fixtures hold only top-level plan files.

**Fix:**
Seed the fixtures of `TestInsertCard`, `TestRemoveCard` and the restore rows of `TestRestoreRenumber` with an `archive-1/` and a `round-1/prior-generation/` holding card-named files.
Extend the byte-for-byte comparison to those subtrees.
These are rows and assertions in the planned tests, not new top-level tests.

### [MEDIUM] Card 4's tests duplicate card 2's and card 3's, and both cards test per verb

**Location:** `_lyx/plan/04-loom-card-verbs.md:62`, `_lyx/plan/02-planparser-renumber.md:74`

**Issue:**
Under `PATTERN-test-economy`, a test must cover a behavior no other test covers.
- `TestRenumberVerbs_Refusals` reaches every planparser precheck and input refusal again and asserts the plan directory byte-identical for each one.
  Card 2's `TestRenumber_Refusals` already pins that every such refusal leaves the directory byte-identical with no journal written.
- `TestInsertCardVerb` asserts "the stub on disk", which `TestInsertCard` already pins.
- `TestRestoreRenumberVerb` replays an injected reversal failure through to a byte-identical restore and repeats the partial-failure case.
  Card 3's `TestRestoreRenumber` already pins both.
What loomcli adds is the envelope shape, `renumberRefusal`'s message and way forward for each sentinel, the begun-card bound and the lease.
Separately, `TestInsertCard`/`TestRemoveCard` and `TestInsertCardVerb`/`TestRemoveCardVerb` are one test per verb for one behavior each: a successful renumber, and a success envelope.
The same cards already write their refusal and I/O tests as one table over both verbs.

**Fix:**
Narrow card 4's tests to what loomcli owns.
The refusal table asserts exit code and way-forward text per row, and the begun-bound rows also assert the directory is untouched, since only loomcli reaches them.
Fold the two success tests into one table over both verbs in each of cards 2 and 4, and lower the test-budget raises to match.

### [LOW] A non-integer N is read as a run-id before the verb runs

**Location:** `_lyx/plan/04-loom-card-verbs.md:51`

**Issue:**
Card 4 says "A `<N>` that is not an integer takes the position refusal."
loom's persistent pre-run arms every non-group verb.
`resolveRunID` takes `args[0]` as the run-id for every verb but `reject`, whose exception exists because its positional argument is not a run-id (`internal/loomcli/arm.go:86`).
`resolveRecipe` then runs `shedrun.ReadSeed`, which calls `ValidateRunID` (`internal/shedrun/runid.go:28`).
So `remove-card ..` or `remove-card a/b` is refused at arming with a shed arming error and never reaches the position refusal.
An integer `N` also arms the run addressed as `N`, not the worktree's own run.
Card 4 lists neither `resolveRunID` nor this path.
None of card 4's tests can reach the non-integer case, since they drive the verb functions with an `int`.

**Fix:**
Exempt the three verbs from the positional run-id read beside `reject`, adding `internal/loomcli#loomCLI.resolveRunID` to card 4's `**Edit:**`.
Alternatively, drop the non-integer sentence and give the cobra shell's parse refusal its own reach.

### [LOW] The lease test passes for a verb that released the lease early

**Location:** `_lyx/plan/04-loom-card-verbs.md:62`

**Issue:**
The test starts a goroutine from the injected `ops` that calls `websterengine.AcquireStateMutation`, then checks that the acquisition is recorded after the verb's last apply step.
If the goroutine is scheduled only after the verb returns, it acquires afterwards whether or not the verb held the lease, so the assertion holds for a broken verb too.
Scheduling decides the outcome, which `PATTERN-test-speed` and `scribe:testing` treat as a defect.

**Fix:**
Wrap `renumberDeps.acquireLease` around the real `AcquireStateMutation` so the test records when it is acquired and released.
Have the injected `ops` assert, at each apply step, that the lease is held.
That pins the acceptance criterion deterministically, without a goroutine race.

### [LOW] Precheck keys on a readable journal, not an existing one

**Location:** `_lyx/plan/02-planparser-renumber.md:55`

**Issue:**
D2 says both verbs "refuse to start while a journal exists".
Card 2 refuses only "a readable journal".
Under that wording, a journal that exists but cannot be read falls through to `writeRenumberJournal`, whose rename replaces it and destroys the only restore record of the interrupted call.

**Fix:**
Refuse with `ErrRenumberJournalExists` on a successful read, and treat any read error other than `fs.ErrNotExist` as `ErrRenumberIOFailed` before anything is written.

## Verdicts on the focus directives

- Fidelity: D1–D5, the apply order, the journal contents, the lease, the begun bound and each failure injection are carried, with the gaps F1, F2, F3, F5 and F6.
  The overview's `scratch-path-ownership` decision reads D2's "its path declared in `internal/lyxdirs`" as a planparser accessor built from the `lyxdirs` token.
  That is not a defect: `internal/lyxdirs` declares only the two tokens, and `PATTERN-durable-vs-ephemeral-state` puts each module's scratch accessor beside its durable one, as `websterengine.ScratchDir` does.
  No card adds unlicensed scope.
- Attack surface: insert and remove are bounded in card 4, and the begun bound in card 1 matches `DispatchScope`'s begun set; `restore-renumber` is F5, and the journal-presence wording is F10.
- Card order: card 4 uses only symbols cards 1–3 create, and card 5 follows card 4; no card depends on a later one.
- Edit-card glyphs: card 4 lists `Command` and `verbUsesLightweightWiring` as member glyphs, and the other `Edit` entries are test files, `doc.go`, YAML or markdown.
  No signature changes, so no re-sign arrow is missing.
- Test economy: cards 2 and 3 table their refusals and I/O failures over both verbs; F7 covers card 4's duplication and the per-verb success tests.
  The budget raises match the current counts: planparser 86, websterengine 199 and loomcli 195, each exactly at its row.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover planparser, websterengine, loomcli, `cmd/lyx` and `contracts/stencils`, in the template's order; `lyx loom validate-plan` reports only informational findings.
- Card 5: the reachability rule is paired between `loom-template-plan.md` and the rubric's `## Also flag`, with the same scope, the raw-I/O exemption and the constructor-only clause.
  The rubric flags nothing the stencil tells the writer not to write.
