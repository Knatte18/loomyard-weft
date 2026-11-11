---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/04-machine-name-set.md:14-15
    summary: Card 4 plans TestNextFile and TestHubOfDir against the unexported helpers nextFile and hubOfDir, and half of TestNextFile's rows repeat TestName_Set rows, breaking PATTERN-test-speed's test economy
    fix: In _lym/plan/04-machine-name-set.md, delete the Create lines `plan:internal/machinecli#TestNextFile` and `plan:internal/machinecli#TestHubOfDir`, and delete the TestNextFile and TestHubOfDir bullets from the Tests list. Add these rows to the TestName_Set bullet. `NoFileTakesTheHostnameNameAsOld`, with no file and host `osl-1046`; `osl46` writes `{"name":"osl46","names":["osl-1046","osl46"]}`. `AlreadySetWritesNothing`, where the file's bytes are unchanged and the result's `written` is false. `NoHostnameAddsNone`. `NewNameMovesToTheEnd`. `CwdContainerDirectoryIsSwept` sets `cwdHub` to `hubOfDir` itself and passes as cwd a `t.TempDir()` whose `fabricengine.BoltDir` holds a shortname written with `fabricengine.WriteShortname`; the request reaches that hub. `PlainCwdAddsNoHub` passes a bare `t.TempDir()` with `cwdHub` set to `hubOfDir` and adds no hub. State that `OwnFormerNamePassesTheCollisionCheck`, `UndecodableFileLosesItsNamesAndSaysSo`, `HandWrittenNameSweepsTheHostnameName` and `EmptyEntryDropIsReported` each assert the written file's `names`, which carries the cut TestNextFile rows of the same cases. Keep the note that the worktree-subdirectory case is TestName_SweepsHubs' cwd. validate-plan --overlay with this target drop reports no blocking finding beyond the baseline.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/01-owner-replace-set.md:49
    summary: Card 1's subtest ReplaceSetLosesToACompetingOriginWrite pins behaviour TestSetOwners' CompetingWriteLosesToOrigins already covers, which breaks PATTERN-test-speed's no-redundant-test rule
    fix: In _lym/plan/01-owner-replace-set.md, delete the Tests bullet `ReplaceSetLosesToACompetingOriginWrite`. Keep the Intent sentence that the re-read after the push judges each selected slug as it judges a named one, and add that `CompetingWriteLosesToOrigins` (internal/boardengine/ownerset_integration_test.go:453) pins the Lost report on that shared re-read, while `ReplaceSetSelectsAfterThePull` pins that replace-selected slugs enter it.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-owner-replace-set.md:36-37
    summary: Card 1 leaves three doc sentences saying that without Takeover an owner that reads as another machine is never written, which the replace set makes false
    fix: In card 1's Intent, add that these three sentences are reworded to except the replace set. The `OwnerRequest.Takeover` field doc (internal/boardengine/ownerset.go:43) becomes "without it, or a replace set naming that recorded owner, such an entry is left alone and reported in OwnerWrite.Lost". The `SetOwners` doc sentence "Without req.Takeover, an entry whose effective owner is another machine is neither written nor claimed" (ownerset.go:72-73) gains "unless req.Replace names its recorded owner". internal/boardengine/doc.go:183, "Without Takeover, SetOwners leaves an entry whose effective owner is another machine untouched", gains the same exception. All three are already among card 1's Edit targets.
  - id: F4
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-machine-name-set.md:85
    summary: Card 4's step 4 reads the unpushed seam but prescribes no outcome for its error
    fix: In card 4's step 4, after "read `unpushed` (kept for step 7)", add that an `unpushed` error is recorded in that hub's checked-locally detail and reads as not ahead, so the hub is still checked and swept, and a hub that records nothing reports `nothing to sweep` with that detail. Add a TestName_Set row `UnpushedReadErrorIsNotedAndSwept`, in which one hub's `unpushed` errors, the hub is swept and its outcome carries the error.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/04-machine-name-set.md:98
    summary: Card 4's failed-hub way forward depends on whether the error names its own way forward, a condition sweepHub has no way to test
    fix: In card 4's step 7, replace "its way forward is the error's own way forward when the error names one (as a dirty or diverged Bolt's or a deferred push's does), then the re-run" with "the outcome carries the error's text, which names its own way forward where SetOwners gives one, and its way forward field is the re-run `lym machine name <name>`". In the `FailedHubIsReportedAndOthersSweep` row, replace "a write error naming its own way forward reports it before the re-run" with "the outcome carries the write error's text and the re-run as its way forward".
  - id: F6
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/04-machine-name-set.md:117
    summary: Card 4 says fakeHubs keys five per-hub seams, but deps has four per-hub seams
    fix: In card 4's TestName_Set bullet, replace "keys the five per-hub seams by board directory" with "keys the per-hub seams `pull`, `unpushed`, `readOwners` and `setOwners` by board directory".
---

### [BLOCKING] Card 4 tests unexported helpers directly and repeats TestName_Set's cases

**Location:** `_lym/plan/04-machine-name-set.md:14-15` and the Tests bullets at `:114-116`

**Issue:** `PATTERN-test-speed` says an unexported helper is tested through the surface that uses it, and that a new test function must cover a behaviour no existing test covers.
`TestNextFile` targets `nextFile` and `TestHubOfDir` targets `hubOfDir`, both private.
Four `TestNextFile` rows repeat `TestName_Set` rows at the verb's surface:

- `UndecodableKeepsOnlyTheHost` repeats `UndecodableFileLosesItsNamesAndSaysSo`.
- `HandWrittenNameOnlyRecordsTheHost` repeats `HandWrittenNameSweepsTheHostnameName`.
- `EmptyEntryIsDroppedAndCounted` repeats `EmptyEntryDropIsReported`.
- `InvalidNameKeepsNamesAndAddsTheHost` repeats the repair `OwnFormerNamePassesTheCollisionCheck` drives.

`TestName_Set` can reach `hubOfDir`'s container and plain-directory cases through the verb, because `hubOfDir` is offline.
`lymcwd.ResolveWorktree` resolves the root in-process through `dotgit.FindRoot` (`internal/lymcwd/lymcwd.go:152-162`), and the shortname read is a file read.
A row then sets the `cwdHub` field to `hubOfDir`.

### [BLOCKING] Card 1's competing-write subtest repeats CompetingWriteLosesToOrigins

**Location:** `_lym/plan/01-owner-replace-set.md:49`

**Issue:** After the push, `SetOwners` judges every slug in `written` through one re-read loop (`internal/boardengine/ownerset.go:141-160`), whether a named slug or a replace-selected one put it there.
`CompetingWriteLosesToOrigins` (`internal/boardengine/ownerset_integration_test.go:453`) already pins that a `BeforePush` competing origin write lands a slug in `Lost` with origin's owner standing.
`ReplaceSetSelectsAfterThePull` already pins that replace-selected slugs enter `written`, since it asserts them in `Recorded`.
`ReplaceSetLosesToACompetingOriginWrite` reaches no other code, and `PATTERN-test-speed` makes a redundant test a finding exactly as a missing one is.

### [MEDIUM] Card 1 leaves the Takeover docs false

**Location:** `_lym/plan/01-owner-replace-set.md:36-37`

**Issue:** The replace set writes over an owner that reads as another machine with `Takeover` unset.
Three docs then go false: `internal/boardengine/ownerset.go:43`, `ownerset.go:72-73` and `internal/boardengine/doc.go:183`.
Each says that without `Takeover` such an entry is left alone.
Card 1 lists each of their symbols or files as an Edit target, but its Intent directs other doc edits only.

### [LOW] Card 4 gives the unpushed read error no outcome

**Location:** `_lym/plan/04-machine-name-set.md:85`

**Issue:** `Bolt.HasUnpushed` returns an error (`internal/fabricengine/bolt.go:171`).
Card 4 reads it before each hub's pull and uses it only to tell `pushed` from `nothing to sweep`, but no step says what its error does.

### [LOW] Card 4's failed-hub way forward rests on an untestable condition

**Location:** `_lym/plan/04-machine-name-set.md:98`

**Issue:** `SetOwners` embeds its way forward in the error text only (`internal/boardengine/ownerset.go:122`, `:278`, `:288`).
`sweepHub` has nothing to test for whether an error "names one".
The branch and the `FailedHubIsReportedAndOthersSweep` row both depend on that condition.

### [NIT] Card 4 miscounts fakeHubs' per-hub seams

**Location:** `_lym/plan/04-machine-name-set.md:117`

**Issue:** `deps`' per-hub seams are `pull`, `unpushed`, `readOwners` and `setOwners`, which is four, not five.
`hubs` and `cwdHub` are not per hub.
A count here also goes stale whenever a seam is added.

## Focus verdicts

- Card 4 test economy: settled as F1 for `TestNextFile` and `TestHubOfDir`.
  The `TestName_Set` rows otherwise each cover a distinct branch.
  `TestName_SweepsHubs`' subtests cover what tier 0 cannot: a real pull before the collision check, the worktree-subdirectory cwd and a real re-run.
- Card 4 refusal reachability: each refusal has a named input and a named row.
  Those are the invalid name, no directory, relative variable, unreadable file, registry failure, unreadable board and collision.
  `CheckName` runs first and passes the valid names those rows use.
  `readCurrent` keeps a read error apart from a decode error, so the unreadable-file row reaches its refusal and an undecodable file reaches the repair.
  Not a defect.
- Card 1's replace selection: the Intent states what it skips (the `OwnedElsewhere` check) and its bound (recorded owners named in the set, on open beam projects and the prime only, with `Takeover` still the only general takeover).
  The refusal of `Replace` combined with `Slugs`, `Prime`, `Claim` or `Takeover` is a fail-loud wiring guard in `contracts/specs/refusal-spec.md`'s sense: no on-disk state reaches it.
  Not a defect.
- Cards 3 and 4 granularity and typing: test helpers and private types are bundled on the cards that own their surface.
  The Edit targets `deps`, `productionDeps`, `newCommand` and `cmd/lym#newRoot` change no signature, so no arrow is missing.
  `docs/overview.md` and `pattern/PATTERN-cli-cobra.md` take only additions in card 3, so `Extends` fits, and card 4 rewrites the overview bullet under `Edit`.
  Not a defect.
- Fidelity: every Decision of D5, D6 and D8 and every Constraint is carried, or already holds in the tree.
  The `PATTERN-leaf-packages` background already admits `internal/standalonestate`, and `PATTERN-owner-machine` already reads the machine file first.
  `machinename.FromThisHost` is licensed by `PATTERN-owner-machine`'s sole-source scan (`internal/machinename/machinename_test.go:234`).
  The own hub lookup is licensed because `preflight.ResolveMode` refuses a wired hub worktree's subdirectory (`internal/preflight/predicates.go:96-125`), and `lym machine name` is no cliwire standalone module.
- Card order and Verify lines: card 4 uses only cards 1 and 3, and card 3 uses card 2.
  No card has a `**Verify:**` line.

## Rejected

- lens:repo-invariants finding 2 (TestName_SweepsHubs' file tier): the card prescribes no file placement that breaks tier purity, and `cmd/lym/tierpurity_test.go` and `tiertags_test.go` refuse a mixed file at the card's own gate, so naming test files is left to the implementer.
