---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lym/plan/02-hostname-name.md:12
    summary: FromThisHost is a new export that hands any caller the hostname rule's name past the machine file, and its Intent states no bound on who may use it
    fix: >-
      In card 2's Intent, state the bound beside the doc-comment sentence:
      `FromThisHost` serves the machine name verb's own name set alone,
      and no owner, address or channel name is ever taken from it,
      since `Name` is the one source of this machine's name and fails rather than fall back to the hostname.
      Reword the planned doc comment so it stands alone and names no other symbol:
      replace "it ignores the machine file, unlike `Name`" with
      "FromThisHost returns the hostname rule's name of this host and never reads the machine file; it is for callers that need that name beside the file's names, and never as this machine's name".
  - id: F2
    severity: LOW
    class: design
    location: _lym/plan/04-machine-name-set.md:121
    summary: The row NewNameMovesToTheEnd names no input and repeats the final-names assertion OwnFormerNamePassesTheCollisionCheck already makes
    fix: >-
      In card 4's `TestName_Set` row list, give `NewNameMovesToTheEnd` the one input that no sibling row covers, or drop it.
      The input is the name already set with `names` out of order and nothing else to change:
      file `{"name":"osl46","names":["osl46","neve"]}`, host `osl46`, run `lym machine name osl46`, which writes `names` `["neve","osl46"]` and so shows that a re-run writes the file when the rewrite changes `names`.
      `AlreadySetWritesNothing` covers the no-write half of that rule,
      and `OwnFormerNamePassesTheCollisionCheck` (expected `names` `["neve","box","osl46"]`) already pins the move of a name that sits in `names` to the end.
  - id: F3
    severity: LOW
    class: design
    location: _lym/plan/04-machine-name-set.md:142
    summary: The tier 1 subtest FailedHubIsSweptByTheReRun builds two real hubs and a second clone for behavior the tier 0 rows and boardengine's own refusal row already pin
    fix: >-
      Cut the `FailedHubIsSweptByTheReRun` subtest from card 4's `TestName_SweepsHubs` and the sentence that lists it,
      and fold its criterion into the tier 0 row `FailedHubIsReportedAndOthersSweep`:
      run the command a second time with the same `fakeHubs`, the failing hub's `setOwners` now succeeding,
      and assert the second run's request for that hub carries the same `Machine` and `Replace` and the machine file's bytes are unchanged.
      The verb treats every write error alike, so the real unreachable origin pins no branch of the verb;
      `internal/boardengine`'s `TestSetOwners` row `Refusal_UnreachableOriginAtPull` already pins that a refused write leaves the Bolt as it was.
      The remaining subtests keep the real-Bolt pins, the origin-only collision and the sweep through the cwd subdirectory.
---

### [LOW] FromThisHost carries no stated bound on its callers

**Location:** `_lym/plan/02-hostname-name.md:12`, card 2 `Intent`.

**Issue:**
Card 2 adds `machinename.FromThisHost`, which returns the hostname rule's name whatever the machine file holds.
`PATTERN-owner-machine`'s background says a machine file that yields no valid name is an error and never a fallback to the hostname, and the sole-source scan (`TestSoleSource_NoOtherProductionFileReadsTheHostname`, which I read in `internal/machinename/machinename_test.go`) bars other packages from reading the hostname.
The new export lets any package take the hostname's name without reading the hostname itself, so the scan does not see it.
The Intent names the one intended caller, `internal/machinecli`, but states neither what the export can now skip nor what bounds its use.
The planned doc comment also describes the function by comparison, "unlike `Name`", which the repo's comment rule bars: a comment states what its symbol does and why, never in terms of another symbol.

### [LOW] Row NewNameMovesToTheEnd repeats a sibling row's assertion

**Location:** `_lym/plan/04-machine-name-set.md:121`, card 4 `TestName_Set` rows.

**Issue:**
The row names no input.
The move-to-end behavior is already pinned by `OwnFormerNamePassesTheCollisionCheck`: its file holds `["neve","osl46"]`, the hostname rule's name `box` lands after `osl46`, and the expected `["neve","box","osl46"]` requires the name last.
The append branch is pinned by `NoFileTakesTheHostnameNameAsOld` and `HandWrittenNameSweepsTheHostnameName`.
A row that names no input invites the implementer to copy the sibling's case, and a redundant row is a defect as a missing one is.
Round 1's fix added this row from its own proposal list without an input; none of round 1's findings was disputed or deferred, and its F1 to F6 fixes hold in the current plan.
The one case no row covers is the name already set with its `names` out of order, which writes only because the rewrite changes `names` (D5: a run with the name already set writes the file only when that rewrite changes `names`);
`AlreadySetWritesNothing` covers the other half of that rule.

### [LOW] Tier 1 subtest FailedHubIsSweptByTheReRun duplicates tier 0 and boardengine coverage

**Location:** `_lym/plan/04-machine-name-set.md:142`, card 4 `TestName_SweepsHubs`.

**Issue:**
The card justifies each subtest as asserting only what the tier 0 rows cannot.
For this subtest the real substrate adds an unreachable origin, but `sweepHub` branches on the error's presence and never on its class, so the real error pins no branch of the verb.
What the subtest pins is covered elsewhere.
That a failed hub is reported with the re-run as its way forward while the others sweep is `FailedHubIsReportedAndOthersSweep`.
That a re-run with the name set derives the same replace set and writes nothing is `AlreadySetWritesNothing`, whose request assertion pins `Replace` as the former names.
That a refused write leaves the Bolt untouched is `TestSetOwners`' `Refusal_UnreachableOriginAtPull`, which I read in `internal/boardengine/ownerset_integration_test.go`.
The subtest costs two hubs and a second clone to repeat them; the PATTERN-test-speed economy section keeps real-hub tests the exception.
Round 1's focus verdict counted "a real re-run" among what tier 0 cannot cover; that holds for the pull before the collision check and the cwd subdirectory, but not for the re-run, since the re-run's replace set and unchanged file are verb logic over the file's `names` and run the same against a fake.
The acceptance criterion it carries (a re-run with the same name sweeps the failed hub) is met by the folded tier 0 assertion in the fix.

## Rejected

- Card 1, row `ReplaceSetWithNoMatchingOwnerWritesNothing`: sibling `AbsentEntryWritesNothing` shares the `len(written) == 0` return, but the row alone reaches the prime-owner-not-in-the-set branch of the replace selection, which `ReplaceSetSelectsAfterThePull` never reaches (its prime matches).
  It is a table row, not a new function.
- Card 1, subtest `ReplaceSetSelectsAfterThePull`: its sibling `CompetingWriteLosesToOrigins` pins the `Lost` report on the shared re-read; the subtest asserts the selection after the pull, the commit subject and the untouched owners, none of which that row reaches.
- Card 2, `TestFromThisHost`: no other test reaches `FromThisHost`, and the error path it skips is pinned by the `UnreadableHostnameNamesItsWayForward` row of `TestResolveNames`, which I read and which asserts the hostname error and the way forward.
- Card 3, `TestName_Show` rows against card 4's `TestName_Set` rows on the same bad-file classes: each pair is one input through two verbs with different outputs, a printed error against a refusal, so no row repeats another.
- Card 4, subtest `RenameSweepsEveryHub` against card 1's `ReplaceSetSelectsAfterThePull`: the acceptance criteria state the done-entry and other-owner retention at the verb's level, and the subtest alone reaches the real `liveHubs`, `hubOfDir` and production seams across two hubs.
- Card 4, `CollisionWithAnOriginOnlyOwnerIsRefused` against the tier 0 `CollisionIsRefusedNamingHubAndOwner`: only the real pull shows the check reading an owner origin alone held.
- Focus item 2, per-hub sweep seams: every seam call that returns an error has a stated outcome and a named row.
  `unpushed` error is `UnpushedReadErrorIsNotedAndSwept`, `pull` error or skip is `PullSkipOrErrorIsCheckedLocally`, `readOwners` error is `UnreadableBoardIsRefusedBeforeAnyWrite`, `setOwners` error is `FailedHubIsReportedAndOthersSweep`, and `hubs` error is `RegistryReadFailureIsRefusedBeforeAnyWrite`.
  `cwdHub` returns no error: `fabricengine.ReadShortname` returns a found flag, which I confirmed in `internal/fabricengine/shortnamebinding.go`.
  A failed machine-file write is a propagated raw I/O error and is exempt.
- Focus item 3, card 1's replace-set doc coverage: the sentences in `internal/boardengine` that state when an owner reading as another machine is left alone are the `OwnerRequest.Takeover` field doc, the `SetOwners` doc sentence and the `internal/boardengine/doc.go` sentence, and the card's Intent rewrites all three.
  The `writeOwners` doc line about an entry left alone for an owner elsewhere stays true, since a replace write reports nothing in `Lost` for an entry it does not select.
  A search of the Go and markdown files outside `docs/research` found no other sentence on the subject; `internal/adoptcli`'s doc concerns adopt's own takeover.
- Fidelity: each of D5, D6 and D8 and each acceptance criterion of the decision record is carried by a card; the `PATTERN-leaf-packages` background already holds the `internal/standalonestate` clause the constraint names.
