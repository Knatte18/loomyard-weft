---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/ownerarchive_integration_test.go:158
    summary: no test row pins that teardown and --drop-local archive/ tags are not listed, which D12 and the last acceptance criterion require
    fix: In TestReadOwnerArchives add a row that pushes a tag named archive/root/<sha> to origin for a commit the branch lacks, next to the archive-owner tag, and asserts only the archive-owner tag is returned and no archive/ tag is fetched locally.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/ownerarchive_integration_test.go:168
    summary: the records tag is never judged against a records branch that holds it, so a comparison against the code branch would pass
    fix: In TestReadOwnerArchives add a row with archiveRecords and a push of the records tip to origin that expects nothing listed; the existing pushTip step pushes only the code side, so make it push dir, which is the records repository for that row.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/cli.go:55
    summary: the --here and --archives wiring (resolveCall, deps.Worktree, PullBolt, hubArchiveDeps, flag registration) is driven by no test
    fix: Add one hubforge-based test in internal/adoptcli that runs Command() with --here from the prime and from a beam worktree and asserts the prime ends owned_here with no record written, so a wrong Worktree name fails it.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/here.go:53
    summary: the merge of the call's flags into a resumed record's request is unpinned, so dropping --no-orch from lym s would pass every test
    fix: In TestRunAdopt add a resume row with the call's NoOrch set and an unfinished record, and assert the worker args or the request it runs carry the flag.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/caller_test.go:112
    summary: no refusal rows for --here with root or --all, for --here with --discard, or for a PullBolt error
    fix: Add rows beside HereWithASubjectIsRefused for a request with Root set, a request with All set, discard together with here, and a PullBolt that returns an error, each asserting a non-zero exit and the refusal text.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/ownerarchive_integration_test.go:158
    summary: ReadOwnerArchives has no row for a non-prime slug resolved through the pair's task branch, a branch absent on origin, or several tags sorted by slug, side and tag
    fix: Add rows to TestReadOwnerArchives for a branch absent on origin that lists the tag, and for two tags that return sorted; add a non-prime pair row if hubforge can build one cheaply.
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: cmd/lym/shortcuts_test.go:63
    summary: TestShortcuts_FileAndRootListing is contained by TestCLITree_EveryCommand but pins the help index text of each shortcut, which that test never asserts
    fix: Mark the test with //testtiming:keep pins the shortcut rows of the help index and the shortcut file against the root, directly above its func line.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/ownerarchive.go:801
    summary: any RemoteURL error is read as no origin remote, so a real read failure returns an empty list that reads as no archives
    fix: In readSideOwnerArchives return nil only when the error says the remote is missing, and return the error wrapped otherwise.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/adoptcli/doc.go:4
    summary: the package doc flag sentence still lists only --keep-local, --drop-local, --no-orch and --discard
    fix: Add --here and --archives to the sentence that begins They take the flags.
---

### [MEDIUM] Teardown archive exclusion has no test

**Location:** internal/fabricengine/ownerarchive_integration_test.go:158
**Issue:** D12 and the last acceptance criterion say `lym adopt --archives` lists no teardown or `--drop-local` archive. The code relies on the `ls-remote` pattern `archive-owner/*` not matching `archive/*`, and no row creates an `archive/` tag, so a prefix change would pass every test.

### [MEDIUM] Records tags are not tested against a records branch that holds them

**Location:** internal/fabricengine/ownerarchive_integration_test.go:168
**Issue:** `pushTip` pushes only the code side, so the records row is listed whether the comparison uses the records branch or the code branch. D12 requires the weft branch for weft tags.

### [LOW] CLI wiring of --here and --archives is untested

**Location:** internal/adoptcli/cli.go:55
**Issue:** Every adoptcli test builds `callerDeps` or `archiveDeps` by hand, so `resolveCall`, `deps.Worktree = location.WorktreeName`, `hubCallerDeps.PullBolt` and `hubArchiveDeps` run in no test. The code lens read `PrimeName` against `WorktreeName` and found them consistent, so this is a coverage gap and not a defect.

### [LOW] Flag merge into a resumed record is unpinned

**Location:** internal/adoptcli/here.go:53
**Issue:** The resume rows call with an empty request, so `mergeRequest` could drop the call's `NoOrch` unnoticed. `lym s` relies on that flag to keep the orch down until `beam start`.

### [LOW] Refusal paths of --here have no rows

**Location:** internal/adoptcli/caller_test.go:112
**Issue:** The refusals for `root`, `--all`, `--discard` and a failed `PullBolt` are D10 behavior with no row; only the subject refusal is covered.

### [LOW] ReadOwnerArchives branches without rows

**Location:** internal/fabricengine/ownerarchive_integration_test.go:158
**Issue:** A branch absent on origin, the non-prime slug path and the result ordering are not exercised.

### [LOW] Contained test pins what its covering test does not

**Location:** cmd/lym/shortcuts_test.go:63
**Issue:** The redundancy report lists the test as contained by `TestCLITree_EveryCommand`. That test checks Short text, bare and bogus invocation and root.Long, and never the help index rows of the shortcuts. The one-line change this diff makes asserts such a row, so the test is accepted and needs the keep marker.

### [LOW] Every RemoteURL error is read as no remote

**Location:** internal/fabricengine/ownerarchive.go:801
**Issue:** `return nil, nil` on any error hides a corrupt config or a read failure as an empty archive list, which the operator reads as no archives to look at.

### [LOW] Package doc flag list is stale

**Location:** internal/adoptcli/doc.go:4
**Issue:** The first lines now name the two new verbs, while the flag sentence below them still names four flags.

## Rejected

- Correctness lens, re-run `hereSubject` after the pull: `Owners.Effective` returns an empty owner for a slug the board no longer holds, so `--here` ends `owned_here` and does not error; D10 says nothing more of it.
- Tests lens, Effective-through-parent row for a beam with no own owner and no record: the same `Owners.Effective` path runs in the foreign-owned prime and beam rows, so a further row adds no behavior.
- Tests lens, duplicate subjects collapse and `placeErr` rows in `runArchives`: internal bookkeeping reached through the public rows already present; a row for each would be per-helper coverage.
- Tests lens, `TakesUp` for a done record: `Unfinished` is the record's own tested predicate and `TakesUp` only calls it.
- Holistic check of D10 and D12 against the diff: take-up rule, `owned_here`, local Bolt fallback, `no_subject` in other worktrees, the `s` shortcut order, the archive-owner prefix against the `archive/` prefix, the per-side branch comparison and git staying in `internal/fabricengine` all match the decision record; docs named by the repository land in the diff.
