---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md:61
    summary: show reads through machinename.Names, which resolves the real machine file from the environment and not from Deps.FilePath, so the show rows read the developer's file against D8 and Names reports no source
    fix: "In card 3's Intent, replace the show paragraph's read with: show takes the path from deps.FilePath, reads it with machinename.ReadFile, and checks the name with machinename.CheckName; a file that exists reports source file, its name, and former as the non-empty names entries other than name; no file, or FilePath ok false, reports source hostname with deps.HostnameName's name and no former names; a FilePath error, a read or decode error, or a name CheckName refuses reports the error and the path in place of name and former. Under Uses, replace `internal/machinename#Names` with `plan:internal/machinename#HostnameName` (checked with validate-plan --overlay)."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/01-owner-replace-set.md:11
    summary: card 1 lists internal/boardengine/doc.go under Extends while its change makes doc.go:174 (the one commit subject) and doc.go:183 (without Takeover an entry owned elsewhere is untouched) false, and the Takeover field and SetOwners comments say the same
    fix: "In card 1, drop the `**Extends:**` group and add `internal/boardengine/doc.go` under `**Edit:**` (checked with validate-plan --overlay). Rewrite the Intent's doc paragraph: doc.go's Owners section gives both commit subjects, `board: owner <machine> for <slugs>` and `board: owner <machine> replacing <names>`, and excepts the replace set from line 183's rule, since a replace-set write rewrites a recorded owner in the set without Takeover. Add that the OwnerRequest.Takeover field comment and the SetOwners doc comment gain the same exception."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/04-machine-name-sweep.md:53
    summary: card 4 leaves how its tier1 test seeds a beam owner and the prime owner on a hubforge hub to be found out while it runs
    fix: "In card 4, replace the sentence `Read internal/hubforge's exports for the board fixture and how its other callers seed owners.` with the seeding, as internal/fronttask/hub_integration_test.go does it: on the first hub, boardengine.OpenHub(hub.Path) and Board.UpsertTask with a project entry (slug, kind project, title, labels), then boardengine.SetOwners(hub.BoardDir(), OwnerRequest{Machine: \"osl-1046\", Slugs: []string{slug}, Beam: true, Takeover: true}) and SetOwners with OwnerRequest{Machine: \"osl-1046\", Prime: true, Takeover: true}; on the second hub the same beam seeding under another machine's name; the live list hands in each hub's Path. Add `internal/boardengine#OpenHub` and `internal/boardengine#Board.UpsertTask` under Uses (checked with validate-plan --overlay)."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md:68
    summary: card 3 prescribes refusals with no reaching row, namely the FilePath error of a relative variable (set and show), the ReadOwners error naming the hub, and the hubsToSweep registry read error
    fix: "In card 3's TestName row list, add rows whose fakes return each error: FilePathErrorIsRefused (FilePath seam returns an error; set exits non-zero and writes nothing, show reports the error and exits 0), ReadOwnersErrorNamesTheHub (one hub's ReadOwners returns an error; the refusal names that hub and nothing is written) and LiveHubsErrorIsRefused (LiveHubs returns an error; nothing is written). Name each row at its step, as steps 1, 2 and 6 do."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md:77
    summary: the collision refusal's hand-write way forward is conditioned on a name this machine held under a lost file, which the verb cannot observe since an undecodable file yields no names, and no row reaches it
    fix: "In card 3 step 6, replace `For a name this machine held under a file lost to a decode failure` with `When the machine file did not decode`, and add the row UndecodableFileNameOwnedElsewhereNamesTheHandWrite: an undecodable file and a board recording the new name as an owner; the refusal names writing {\"name\": \"<name>\"} to the file's path, and the file's bytes are unchanged."
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/03-machine-name-verb.md:96
    summary: card 3 introduces lym machine name but its Intent states no bound and only points doc.go at the bounds D5 states
    fix: "In card 3's Intent, after the setName steps, add a Bound line: the verb writes only this machine's file; its collision check does not see an owner pushed to origin after its pull, or one only origin holds on a hub whose pull failed; and it passes a former name another host has since taken, a name a live machine holds under which no swept board records an owner, and a hostname rule's name another host shares. The way forward for a shared name is renaming the machine that holds it, as D5 states."
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/03-machine-name-verb.md:58
    summary: card 3 tells the implementer to read how hubgeom passes a hub's board directory, a research step for a fact the card already states, and the overview's shared decision defers to the same lookup
    fix: "In card 3, delete the sentence `Read how the existing owner callers (internal/hubgeom/dobbygeom.go) pass a hub's board directory and match them.` In 00-overview.md's Shared Decisions, replace the board-directory bullet with: A hub's owner calls take fabricengine.BoardDir(hub), as hubgeom.DobbyStart passes it."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-owner-replace-set.md:35
    summary: the replace subtest's empty name in Replace never meets an open beam with no recorded owner, so the row passes with or without the empty-name guard
    fix: "In card 1's subtest, have pushFromOther also add an open beam project with no recorded owner, and assert it stays unowned on origin after the write."
  - id: F9
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/01-owner-replace-set.md:29
    summary: Replace combined with Slugs or Prime has no caller and no row, and would append the prime or a slug to written twice when both selections match it
    fix: "In card 1's Intent, replace the sentence `Replace combines with Slugs and Prime as an added selection; the sweep sends it with neither.` with: an entry or the prime that both Slugs or Prime and the replace set select is written and appended to written once. Add to the subtest a second request with Prime true and the prime owner in the set, asserting Recorded names OwnerWritePrime once."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/03-machine-name-verb.md:69
    summary: step 3 does not tell machinename.ReadFile's read error from its decode error, so a file that exists but does not read could be treated as undecodable and reported as lost
    fix: "In card 3 step 3, add: a ReadFile error with exists false, a file that does not read, is returned as is and nothing is written; only an error with exists true is the undecodable file that is replaced. Add the row UnreadableFileIsAnError, whose file path is a directory, as TestResolveNames_UnreadableFileIsAnError does."
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-machine-name-sweep.md:47
    summary: branches the cards prescribe have no TestName row, among them a hostname error adding no name, a pull error checked locally, the cwd hub added, pull-before-read for the collision, plain nothing to sweep, a HasUnpushed error, the lym adopt addition and no recoveries on a same-name re-run
    fix: "Add as TestName rows: in card 3, a HostnameName error adding no name to this machine's names, a Pull error reported as checked locally only, a Cwd whose hub is not live being checked, and NameOwnedElsewhereIsRefused's owner placed in the fake's owners only by its Pull; in card 4, a hub with nothing recorded and HasUnpushed false reporting nothing to sweep, a HasUnpushed error reporting the hub by its write alone, a Lost entry whose standing owner is not one of this machine's names naming lym adopt, and a re-run with the name already set carrying no recovery lines."
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/03-machine-name-verb.md:87
    summary: the row osl46 over osl-1046 expects names osl-1046 then osl46, which the rewrite rule gives only when the hostname rule's name is osl-1046, and the row does not say so
    fix: "In card 3's row list, write the row as: `osl46` over a file naming `osl-1046`, with the fixed hostname name `osl-1046`, writing `names` `osl-1046` then `osl46`."
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-machine-name-sweep.md:51
    summary: TestName_SweepsRealHubs runs through runWith without saying which deps it builds, and it pins behaviour beyond TestSetOwners and the tier0 rows only when the Pull, ReadOwners, HasUnpushed and SetOwners seams are productionDeps' own
    fix: "In card 4, state that the test takes productionDeps(cwd) and replaces only FilePath (a file under t.TempDir()), HostnameName (a fixed name) and LiveHubs (the two hubs' paths), so the Bolt and owner seams are the production ones."
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-machine-name-sweep.md:48
    summary: the row asserting that a rename's envelope carries the recovery lines tests the wording a Review acceptance criterion leaves to code review
    fix: "In card 4, replace `A rename's envelope carries the recovery lines.` with: a rename's envelope carries a non-empty recoveries field and a same-name re-run's carries none; no row asserts the lines' wording."
---

### [BLOCKING] show reads the developer's machine file instead of the seam's

**Location:** `_lym/plan/03-machine-name-verb.md:61`, with the show rows at line 87.

**Issue:** `machinename.Names` calls `resolveNames(machineFile, os.Hostname)`, and `machineFile` calls `FilePath()` from the process environment (`internal/machinename/machinename.go:83-85`, `119-122`).
`show` reading through it ignores `Deps.FilePath`, so the rows "show with a good file, with no file, and with a bad file" read whatever file the developer's machine holds and cannot be made to pass.
That breaks D8, which has every test of the verb hand in the file's path through `machinecli`'s deps.
`Names` also returns no source, which D5's show needs.
`setName` already reads through `ReadFile`, so the fix keeps the name read inside `internal/machinename`'s functions, as `PATTERN-owner-machine` requires.

### [MEDIUM] Card 1 extends a doc whose lines it falsifies

**Location:** `_lym/plan/01-owner-replace-set.md:11` and its doc paragraph at line 39.

**Issue:** `internal/boardengine/doc.go:174` gives one commit subject, `board: owner <machine> for <slugs>`, and line 183 says that without Takeover, SetOwners leaves an entry owned elsewhere untouched.
A replace-set write commits under a new subject and rewrites such entries without Takeover.
The rubric reserves `Extends` for adding to a ref without changing what is there, so the ref belongs under `**Edit:**`.
The `OwnerRequest.Takeover` field comment (`ownerset.go:43`) and the `SetOwners` doc comment (`ownerset.go:72-73`) state the same rule, and the card's Intent leaves them as they read.
The affected-set and repo-invariants lenses both reported this defect.

### [MEDIUM] Card 4 leaves its owner seeding to be researched

**Location:** `_lym/plan/04-machine-name-sweep.md:53`.

**Issue:** The test needs a beam project and the prime recorded under `osl-1046` on a real hub, and the card says to find out how while it runs.
The repository already holds the answer: `internal/fronttask/hub_integration_test.go` seeds a beam with `boardengine.OpenHub` and `Board.UpsertTask`, then records owners with `SetOwners(hub.BoardDir(), …)` with `Beam` and `Takeover` set, or `Prime` and `Takeover` set for the prime.
Neither seeding call is in the card's Uses.

### [MEDIUM] Card 3 refusals with no reaching row

**Location:** `_lym/plan/03-machine-name-verb.md:68`, `:74` and `:59`.

**Issue:** A relative config variable makes `FilePath` return an error (`machinename.go:42-46`), a corrupt `board.json` makes `ReadOwners` fail through `ParseOwners` (`ownerset.go:295-307`), and an unreadable `hubs.json` makes `hubregistry.Live` fail (`hubregistry.go:98-124`).
Each halts the verb with a message, none is a propagated raw I/O error that `refusal-spec.md` exempts, and the row list names no test that reaches any of them.

### [MEDIUM] The lost-file way forward rests on a state the verb cannot see

**Location:** `_lym/plan/03-machine-name-verb.md:77`.

**Issue:** An undecodable file leaves `ReadFile` returning an empty `File` (`machinename.go:63-65`), so the verb cannot know which name it "held under a file lost to a decode failure".
The observable condition is that the file did not decode, and D5 attaches the hand-write way forward to exactly that case.
No row reaches the branch.

### [MEDIUM] Card 3 states no bound for its new verb

**Location:** `_lym/plan/03-machine-name-verb.md:96`.

**Issue:** The rubric's attack-surface rule wants the card that introduces a verb to state in its Intent what the verb can let through and what bounds it.
Card 3 introduces `lym machine name` and its collision check, which D5 bounds explicitly, yet its Intent only tells `doc.go` to document "the bounds D5 states".
Card 4 states its own sweep bound; card 3 states none.

### [LOW] Card 3 carries a research step for a fact it already states

**Location:** `_lym/plan/03-machine-name-verb.md:58` and `_lym/plan/00-overview.md:23`.

**Issue:** Line 57 already prescribes `boardengine.ReadOwners(fabricengine.BoardDir(hub))`, which matches `hubgeom.DobbyStart` (`internal/hubgeom/dobbygeom.go:101-109`).
The next line still sends the implementer to research it, and the overview's shared decision names the fact only by where to look it up.

### [LOW] The replace subtest does not reach the empty-name guard

**Location:** `_lym/plan/01-owner-replace-set.md:35`.

**Issue:** After the seed and `pushFromOther`, no open beam has an empty `owner_machine` (`internal/boardengine/ownerset_integration_test.go:55-77`), so the `""` in `Replace` matches nothing whether or not `writeOwners` skips it.
The guard-reach lens rated this MEDIUM.
It is kept at LOW because card 4 also leaves empty entries out of the set it sends, so the acceptance criterion holds at the caller as well.

### [LOW] Replace combined with Slugs or Prime can write an entry twice

**Location:** `_lym/plan/01-owner-replace-set.md:29`.

**Issue:** The existing `Prime` branch appends `OwnerWritePrime` to `written` (`ownerset.go:178-184`).
A replace selection that also matches the prime would append it again, and likewise for a slug named in `Slugs`.
No caller sends the combination and no row reaches it.

### [LOW] Step 3 conflates a read error with a decode error

**Location:** `_lym/plan/03-machine-name-verb.md:69`.

**Issue:** `ReadFile` returns `exists` false with an error for a file that does not read, and `exists` true for one that does not decode (`machinename.go:55-67`).
The step names only the decode case, so a file that was never read could be replaced and reported as lost.

### [LOW] Prescribed branches with no row

**Location:** `_lym/plan/03-machine-name-verb.md:87-88` and `_lym/plan/04-machine-name-sweep.md:46-48`.

**Issue:** Several branches the cards prescribe have no row, so their behaviour is not pinned:

- a hostname error adding no name;
- a pull error checked locally;
- the cwd's hub added to the sweep;
- plain nothing to sweep;
- a `HasUnpushed` error;
- the `lym adopt` addition for a `Lost` entry under another name;
- no recovery lines on a same-name re-run.

The collision row also does not say that its owner reaches the fake board only through `Pull`.
Without that, it does not pin the acceptance criterion's owner that only origin held before the pull.

### [LOW] The osl46 row's expected names omit the hostname rule's name

**Location:** `_lym/plan/03-machine-name-verb.md:87`.

**Issue:** Step 7 appends the hostname rule's name to `names` when it is absent.
With a fixed hostname name other than `osl-1046`, the row would expect three names, not the two the acceptance criterion and the row state.

### [LOW] The tier1 sweep test's deps are unstated

**Location:** `_lym/plan/04-machine-name-sweep.md:51`.

**Issue:** `TestSetOwners` already pins the replace selection against a real origin, and the tier0 rows pin the sweep's reporting.
What this test adds is `productionDeps`' Bolt and owner wiring.
It adds that only when those seams are the production ones, and the card does not say how its deps are built.

### [LOW] A row tests the wording of a Review criterion

**Location:** `_lym/plan/04-machine-name-sweep.md:48`.

**Issue:** The acceptance criterion on the recovery lines opens with `Review:`, so the rubric leaves their wording to code review.
Whether the envelope carries them after a rename, and not after a same-name re-run, is behaviour a row may pin.

## Rejected

- None of the forks' findings was rejected. The guard-reach lens's empty-name finding is kept at LOW rather than MEDIUM, for the reason in its block.

## Focus verdicts

- Card 3's and card 4's private helpers and types each carry a `plan:` handle, but they all sit on the card of the surface they serve, so neither card breaks the per-card granularity rule. Not a defect.
- Card 1's `PATTERN.md` target makes no textual change, because the declared `changes` text equals the base line. `pattern-line-unprescribed` still requires a card that edits `PATTERN.md` to fence the line, so the target stands. The constraint that the background names the machine file before the hostname is already met at `pattern/PATTERN-owner-machine.md:41-42`, so no card needs to carry it. Not a defect.
- Card 3's edits to `docs/overview.md` and `PATTERN-cli-cobra` are licensed. They document the new module, and the cli-cobra Package-naming deviation is what D5's wiring per `PATTERN-cli-cobra` requires of a cli package with no engine. Not a defect.
- Card order and Uses hold. `machinename.FilePath` and `ReadFile` exist (`internal/machinename/machinename.go:42`, `55`). Card 4's `plan:` edits follow card 3's creates, and its `Replace` use follows card 1. `Deps` is a struct, so it takes no re-sign arrow. `setName` and `productionDeps` keep their signatures.
- Card 2's `TestFromHostname` assertion is the only coverage of `HostnameName` reading `os.Hostname`, so it duplicates no test. On card 4's tier1 test, see F13.
