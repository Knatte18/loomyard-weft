---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1, commit bullet, and D2
    summary: D1 claims its trailer commit is made even when the replaced bytes already landed, but the seam it names makes no commit when nothing is staged, so the D2 cut has no trailer to key on
    fix: In D1, replace the claim that `fabricengine.CommitAnchoredPathsWithTrailers` carries the trailer in every case. State that the commit stages the record through that seam, and that when it reports committed false (HEAD already holds the written bytes), replace lands an empty commit carrying the same trailers through a new fabricengine seam beside `commitEmptySnapshot`, marked (new), that wraps `gitrepo.Repo.CommitEmpty` with caller trailers. State that seam's behaviour when the weft index holds staged content (`CommitEmpty` refuses then) - recommended - refuse with a way forward naming re-running replace. Keep the D2 acceptance criterion for the swept-bytes case, and add a D1 criterion that a replace whose bytes HEAD already holds still leaves a trailer commit at HEAD.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1, commit-failure sentence
    summary: Restoring the prior bytes after a failed commit loops when HEAD already holds the written bytes, from a sweep or from a commit that landed before the error, since the named way forward fails D1's own base and working-copy checks
    fix: In D1, replace the failure sentence with - when the commit returns an error, replace reads HEAD; when HEAD holds the written bytes with the trailer, the call succeeds and reports the commit error in the envelope, as add does with commit_error; when HEAD holds the written bytes without the trailer, it lands the trailer commit as F1's fix describes; only when HEAD holds neither does it restore the prior bytes and refuse, naming re-running replace. Add a D1 acceptance criterion for the HEAD-already-holds case.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1, lock bullet and carry-over bullet
    summary: The per-run decision lock covers add, replace and store only, while the Plan-Review and Code-Review Bouncer settles write and commit carry-over blocks into the same record past approval, so a replace can be overwritten or can drop a block written between its check and its write
    fix: In D1's lock bullet, add `writeCarryOver` in `internal/loomcli/wiring.go` (the Bouncer carry-over writer) to the writers that take the per-run decision lock, so the lock covers every Go writer of the record past approval. Name that writer in the bullet, and add a D1 acceptance criterion that a carry-over write started while a replace holds the lock runs after it and its block is in the record at HEAD.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2, first two sentences
    summary: The trailer names no record, and a pair's weft inherits its parent's commits, so a member pair can find a beam's project-record trailer whose hash matches none of its own record's versions, and D2 does not say what the history then returns
    fix: In D2, make the trailer value carry the record's anchor-relative path beside the --by value and the sha256, and have the history read take the newest trailer commit whose path is this record's. State that a trailer whose sha256 matches no version of the record leaves the history whole. Add a D2 acceptance criterion that a trailer for another record path does not bound this record's history.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3, first two sentences
    summary: After a raddle family filing turns the run's entry into the project and writes the project's own record, D3's stored-record trigger makes a later add or replace on that non-perch run copy the Sketch-level run record over the filed project's record
    fix: In D3, state that the copy is skipped once the run's raddle filing is done (the state `raddleRunFiling` reports), so a filed entry keeps the record Raddle-File wrote, or refuse add and replace on a raddle run past its filing in every mode, as `perchFilingRefusal` does for perch. Recommended - extend the existing filed refusal to every mode, naming add or replace on the filed part. Add a D3 acceptance criterion for that case.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D1, notice bullet, and D6
    summary: The replace notice names no opening, while a spawned role acts on LOOM mail only when it opens with the add notice's exact text, and Merriam, which may not read the record, is told to re-read it whole and so never gets the replaced design
    fix: In D1's notice bullet, name the replace notice's opening line, marked (new), and in D6 add that `contracts/stencils/parent/parent-directive-parent.md` treats a LOOM line with that opening as the parent's instruction, as it does for add's. State what a strand that reads no record gets - recommended - a Merriam strand is held back with a reason in notice_held_back, as `heldBackReason` does for a halted run, and the parent re-enters Webster so the fresh session carries the record. Add D1 acceptance criteria for both.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4, refusal bullet naming a live batten run
    summary: D4's live-batten-run refusal names no seam a board verb can read, cannot be atomic with a batten start that seeds the old record first, and cannot see a batten run on another machine, which leaves a fact the guard rests on for Plan
    fix: In D4, name the seam the refusal reads and the package that owns it (the registered batten processes `pairteardown` reads, or the batten status file), and state its bound - a batten start after the check, and a batten run on another machine, are not seen. Either drop the live-batten clause and rely on the run status alone with that bound stated, or keep it and add an Open risk naming both windows. Record the evidence the choice rests on.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4, refusal bullet, and Rationale
    summary: Once a halted raddle run's status is cleared with lym batten status, D4 admits --record-file, but the run's worktree keeps its own record, which Record-Write never overwrites, so a resumed run still splits by the stale design
    fix: In D4, add a refusal for an entry whose slug has a task worktree in the hub, with a way forward naming `lym loom decision replace <slug>` in that worktree or ending the run with `lym batten cancel <slug>` first. Add the case to D4's refusal acceptance criterion.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4, one-write bullet
    summary: Writing the record file before board.json as StoreDecisionRecord does has no rollback, so a payload the store refuses after the file write leaves the new record beside the old body, against the one-write and nothing-written claims
    fix: In D4's one-write bullet, state that the payload is validated before the record file is written, and that a failed save writes the entry's former record file back, as `boardengine.Board.FileRaddle` does. Add a D4 acceptance criterion that a payload the store refuses leaves the stored record unchanged.
  - id: F10
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 and D2
    summary: A board-side fold with --record-file writes no trailer, and the record does not say whether a raddle run of the Sketch starts on a fresh weft or adopts the motif pair's weft branch, whose history holds the folded entries
    fix: Research whether pair teardown after a motif run archives or removes the weft branch that `fabricengine.Add` would otherwise adopt for the same slug, and record the answer with its evidence in D4. If the branch can be adopted, state in D2 that the seed of a stored record bounds the history too, for example by making `seedDiscussedRecord`'s commit carry the replace trailer.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1, carry-over sentence
    summary: The carry-over check refuses a file that lacks a current block but not one that adds a block, so a replace can write a carry-over block, which only Go may write
    fix: In D1, change the carry-over sentence so the file's carry-over blocks equal the current record's blocks, byte for byte, with none added or dropped, and extend the D1 refusal acceptance criterion to a file that adds a carry-over block.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1, Bound paragraph, last sentence
    summary: The bound says a replace changes IdentityText, but IdentityText strips added entries, so a replace that only drops or rewrites added entries leaves it unchanged
    fix: Reword the sentence to - a replace that changes any text outside added entries and carry-over blocks changes `discussionparser.IdentityText` of the record, so a later Bouncer re-entry whose subject holds the record reviews it again; a replace that only drops added entries changes no reviewed text.
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D1, first bullet
    summary: The caller list says only the parent, its arbiter or the operator's unnamed shell replaces, but add's sampler rule, which D1 takes unchanged, also admits the sampler strand with --by operator
    fix: Add the run's sampler strand, with --by operator only, to the caller list in D1's first bullet.
  - id: F14
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Notes for the plan writer, second bullet
    summary: The note that bounding recordHistory reaches every caller repeats D2's last sentence
    fix: Delete the second bullet of Notes for the plan writer.
---

### [BLOCKING] The trailer commit is not made when the bytes already landed

**Location:** `_lym/discussion/decision-record.md` D1 commit bullet; D2.

**Issue:** `fabricengine.CommitAnchoredPathsWithTrailers` hands off to `CommitAnchoredPaths`, which reaches `gitrepo.Repo.StageAndCommit`.
That function returns `committed=false` and makes no commit when `git diff --cached --quiet` finds nothing staged (`internal/gitrepo/gitrepo.go:146-151`; `internal/fabricengine/trailercommit.go:77`).
So when a sweep already landed the replaced bytes, no trailer commit exists, D2 reads the whole history, and every folded entry is reported as `added-decision-dropped`.
On a Record-Write run, `checkDarnRecord` then refuses every later `add`.
The only empty-commit primitive is `gitrepo.Repo.CommitEmpty`, reached only through `commitEmptySnapshot` (`internal/fabricengine/recordsgit.go:178-188`), and it takes no caller trailers.
All four lenses found this independently.

### [MEDIUM] A failed commit's restore leaves a refusal loop

**Location:** `_lym/discussion/decision-record.md` D1 commit-failure sentence.

**Issue:** HEAD can already hold the written bytes when the commit errors.
That happens through a sweep, or through `StageAndCommit` failing in `CurrentSHA` after `git commit` landed (`internal/gitrepo/gitrepo.go:157-166`).
Restoring the prior bytes then makes the working copy differ from HEAD, so every re-run of the named way forward fails D1's own base and working-copy checks.
`add` already handles this case by reading HEAD after a failed commit (`internal/loomcli/decision.go:650-680`).
The existing-paths, interactions, walkthrough and bound-breaker reports are merged here.

### [MEDIUM] The decision lock leaves out the carry-over writer

**Location:** `_lym/discussion/decision-record.md` D1 lock and carry-over bullets.

**Issue:** `writeCarryOver` (`internal/loomcli/wiring.go:512-519`) writes and commits the record from the Plan-Review and Code-Review Bouncer settles, past approval, and takes no decision lock.
A settle that read the record before a replace can write the old record back, plus its block, after the replace succeeded.
A block written between the replace's check and its write is overwritten.
The base-hash check catches a carry-over only once it is committed, so the window is narrow.
`add` shares that exposure today, which is why the bound-breaker report's BLOCKING became MEDIUM here.
It still falsifies D1's kept-carry-over bound.

### [MEDIUM] The trailer names no record

**Location:** `_lym/discussion/decision-record.md` D2.

**Issue:** `FindTrailerCommit` matches on the key alone (`internal/fabricengine/trailercommit.go:120-146`).
A pair's weft forks from its parent's records branch, so a member under a beam can find the beam's project-record trailer.
No version of the member's own record hashes to that trailer, and D2 does not say whether the history is then empty, which turns the check off, or whole.

### [MEDIUM] D3 overwrites a filed family project's record

**Location:** `_lym/discussion/decision-record.md` D3.

**Issue:** `convertRunEntry` turns the run's entry into the project and writes the project's own record (`internal/boardengine/raddlefile.go:204-228`).
`perchFilingRefusal` refuses only perch mode (`internal/loomcli/decision.go:942-959`).
So once D3 copies to every entry that carries a stored record, an `add` or `replace` on a non-perch raddle run after its filing copies the Sketch-level record over the project's record.

### [MEDIUM] The replace notice reaches neither the directive's filter nor Merriam

**Location:** `_lym/discussion/decision-record.md` D1 notice bullet; D6.

**Issue:** The parent directive acts on LOOM mail only when it opens `Parent decision recorded with lym loom decision add` (`contracts/stencils/parent/parent-directive-parent.md:23`; `internal/loomcli/decision.go:311`).
`add` sends Merriam inline text because Merriam may not read the record (`internal/loomcli/decision.go:313-321,354-359`).
A replace notice that says "re-read it whole" therefore either goes unacted on or reaches a role that cannot read the record.

### [MEDIUM] The live-batten-run refusal rests on an unnamed seam

**Location:** `_lym/discussion/decision-record.md` D4 refusal bullet.

**Issue:** `internal/boardcli` imports nothing from batten (`internal/boardcli/cli.go:16-38`).
The only liveness reads are in `pairteardown` (`registeredBattenAlive`) and `internal/loomcli/driverprompt.go:107`.
The record leaves the seam, its atomicity with a batten start and its blindness to other machines for Plan to find, which makes this a deferred research question.

### [MEDIUM] A cleared run status admits a fold that the run's worktree never reads

**Location:** `_lym/discussion/decision-record.md` D4 refusal bullet and Rationale.

**Issue:** Record-Write never overwrites an existing record file (`internal/loomshed/recordwrite.go:124,153-157`).
After `lym batten status` clears a halted raddle run's status, D4 admits `--record-file`, but resuming that run keeps the stale worktree record.
That is the issue #116 situation the task exists to fix.

### [MEDIUM] The one-write claim has no rollback

**Location:** `_lym/discussion/decision-record.md` D4 one-write bullet.

**Issue:** `StoreDecisionRecord` writes the file, then flags the entry, and never restores the file (`internal/boardengine/decision.go:57-61`).
A payload the store refuses after the file write leaves the new record with the old body.
`FileRaddle` shows the existing restore pattern (`internal/boardengine/raddlefile.go:151-159`).

### [LOW] Which weft history a raddle run starts from after a board fold

**Location:** `_lym/discussion/decision-record.md` D4; D2.

**Issue:** `fabricengine.Add` adopts a live weft branch rather than forking a new one (`internal/fabricengine/add.go:150-153`).
`seedDiscussedRecord` writes the folded record over that history without a trailer (`internal/loomcli/discussionrecord.go:71-81`).
Whether the motif pair's branch survives to be adopted is unverified, so it is LOW until researched.

### [LOW] Added carry-over blocks are not refused

**Location:** `_lym/discussion/decision-record.md` D1 carry-over sentence.

**Issue:** The check holds the current blocks but not their absence elsewhere.
`IdentityText` strips carry-over blocks (`internal/discussionparser/identity.go:35-51`), so a block a replace adds is never re-reviewed.

### [LOW] The IdentityText bound is overstated

**Location:** `_lym/discussion/decision-record.md` D1 Bound paragraph.

**Issue:** `IdentityText` drops every added entry (`internal/discussionparser/identity.go:53-62`), so a replace that only drops or rewrites added entries does not change it.
The behaviour is fine; the sentence is false as written.
The bound-breaker report raised it as MEDIUM, lowered here because no reviewed text goes unreviewed.

### [NIT] The caller list omits the sampler strand

**Location:** `_lym/discussion/decision-record.md` D1 first bullet.

**Issue:** `add`'s sampler rule admits the sampler strand with `--by operator` (`internal/loomcli/decision.go:736-740`), and D1 takes that rule unchanged.

### [NIT] A note repeats D2

**Location:** `_lym/discussion/decision-record.md` Notes for the plan writer, second bullet.

**Issue:** D2 already says every caller of the history seam reads the bounded history.

## Handler notes

The focus questions on board entries are settled with no finding.
No Decision edits this task's own board entry, and none files or promotes a task.
Every D1 to D6 is named in an acceptance criterion, and no Open risk is settled by a Decision.
`seedDiscussedRecord` does overwrite the record (`internal/loomcli/discussionrecord.go:71`), so Open risk 3 stands.
The trailer-forgery question holds: `lym fabric commit` writes the fixed message `weft sync`.
Leanness holds apart from F14: D6's departure reason was moved into the record on purpose in the writer's lens round, and the plan writer needs it.
