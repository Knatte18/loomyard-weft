---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:185
    summary: D5 puts the draft rule where the store commits an entry but names no function, and the rule needs the pre-write entries that only boardCriticalSection holds
    fix: >-
      In D5, replace "applied where `boardengine`'s store commits an entry" with "applied in `Board.boardCriticalSection` (`internal/boardengine/board.go`), beside `checkRunLocks` and `checkTaskCreation`, over the entries as loaded and as the mutation left them".
      Add one sentence that the other two writers of `board.json`, `writeOwners` in `ownerset.go` and the replay in `replay.go`, run no draft rule:
      the owner write never makes an entry a task and only writes claim strings, and the replay copies entries a `boardCriticalSection` write already normalised.
      Add a Constraint that the rule is not placed in `Store.Save`, which holds no pre-write snapshot to tell "already a task before the write" from a create.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:116
    summary: The owned-elsewhere exit code is defined by listing three refusal sites, which misses the start check a full pass of a found status at Worktree-Create reaches
    fix: >-
      In D4, define the owned-elsewhere exit code by the refusal, not by its sites:
      "every refusal of a pass that reads another machine as the run's owner exits with it, through one typed error that `ownedElsewhereRefusal` and the owned arm of `refuseResumeUnreachable` both return".
      Keep the three named sites as examples, and add the fourth, `refuseStartOwnedElsewhere`, which `ownerPreflight` reaches for a found status still at Worktree-Create with no pair on this machine.
      Add that a fresh start's refusal from `claimBoardEntry` or `refuseStartOwnedElsewhere` stays a refused start under D5.
      Extend the acceptance bullet on the owned-elsewhere code to read "any refusal that names another machine as owner".
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:63
    summary: D2 decides the adopt refusals before the owner write but not before takeOverSlug drops local commits and adds pairs, so a refused call can leave those mutations behind
    fix: >-
      In D2, state that both refusals are decided before any mutation of the call, using only the read-only part of step 1 (`ownedElsewhere`, `stoppedBranches` and `LivePair`) to find the entries that pass,
      so a refused call drops no local commits and adds no pair.
      Name the position as ahead of the `takeOverSlug` loop in `internal/adoptcli/adopt.go`, and reword "the entries that passed step 1" to "the entries step 1 does not stop".
      Extend the acceptance bullet on the adopt refusals with "and mutates nothing".
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:118
    summary: D4 re-judges the owner before a retried owner pause but leaves the window between the refused pass and the first pause, in which lyx adopt can land and a stale pause then strands the run
    fix: >-
      In D4, after the sentence that opens the owned-elsewhere handling, add one of two things.
      Either an accepted bound, "an adopt that lands between the refusal and the pause leaves the run paused under the owner caller with its processes ended, and `lyx adopt <slug>` again lifts it",
      or a step, "dobby reads the effective owner from the local Bolt once the pause stands, and when it no longer reads another machine it lifts its own owner pause entry and runs a full pass".
      The retried pause on a probe tick needs no change.
---

### [MEDIUM] D5 names no commit point for the draft rule

**Location:** `_lyx/discussion/decision-record.md:185`, D5.

**Issue:** D5 states the statusless-task guard as one rule, which settles the "list of writes" half of the focus question, and its text covers a note or project upserted to a task and an empty-string status.
But it places the rule "where `boardengine`'s store commits an entry", and the store has no such single place.
`Store` mutates in place through `UpsertTask`, `SetStatus`, `Promote`, `UpsertTasksBatch` and `MergeTasks`, and persists through `Store.Save`, which holds no pre-write snapshot.
The rule is conditional on "the entry was already a task before the write", so it needs both the entries as loaded and as left.
The one place that has both is `boardCriticalSection` (`internal/boardengine/board.go`), which already runs `checkRunLocks` and `checkTaskCreation` over `loaded` and `store.Tasks()` and then calls `Save`.
Two writers bypass it: `writeOwners` (`ownerset.go`) saves on its own, and the replay (`replay.go`) saves a replayed board.
Left unnamed, an implementer can put the rule in each of the five mutators, which is the list of writes the rule was meant to replace, or in `Save`, which cannot tell a release from a create.
This is a fact the design rests on, found in minutes, so Plan should not have to find it.

The release command is settled: `lyx board set-status '{"slug":"<slug>","status":null}'` passes `resolveLookup` (`internal/boardcli/cli.go`), because `status` is a required key and an explicit null is present in the decoded map, and `SetStatus` receives a nil status.
It runs as typed, and the start guard's refusal can name it.

### [LOW] The owned-elsewhere exit code is defined by three sites

**Location:** `_lyx/discussion/decision-record.md:116`, D4.

**Issue:** D4 gives the owned-elsewhere refusal its own exit code and lists `resumeRefusal`'s `EntryOwnedElsewhere` arm, `resumeOwnerClaim`'s lost claim and `refuseResumeUnreachable`.
`ownerPreflight` (`internal/battencli/owner.go`) has a fourth: a full pass of a found status still at Worktree-Create, with no pair on this machine, runs `refuseStartOwnedElsewhere` and returns `ownedElsewhereRefusal`.
That exits non-zero without the code, so dobby reads it as a failing pass, retries it every tick and posts the failing-pass notice after three, the outcome D4 and D7 rule out for an owned-elsewhere refusal.
The cause is the method: the code is defined by a list of sites, where it should follow from the refusal.
`refuseResumeUnreachable` builds its own error, so a shared constructor alone would miss it too.

### [LOW] D2's refusal can follow mutations

**Location:** `_lyx/discussion/decision-record.md:63`, D2.

**Issue:** D2 decides both refusals "before the owner write, against the entries that passed step 1 and go into it".
In `internal/adoptcli/adopt.go` the entries that go into the owner write are the ones `takeOverSlug` returns true for, and `takeOverSlug` also runs `DropLocal` and `AddPair`.
A refusal placed after that loop leaves a call with `--drop-local` having moved branches under archive tags, and pairs added, with the envelope saying nothing was resumed.
The stops the refusal needs, a diverged or live pair, are read-only, so the refusal can come first.
The record leaves the position open.

### [LOW] D4 leaves the stale-verdict window before the first owner pause

**Location:** `_lyx/discussion/decision-record.md:118`, D4.

**Issue:** The focus question on a failed owner pause is settled for the retry.
D4 runs a full pass on each probe tick while the mark stands and no pause covers the slug, and that pass re-judges the owner through `ownerPreflight` before anything pauses, so a run adopted back is resumed and the mark clears.
The same refusal that starts the handling is a verdict too, and `PauseOwnedElsewhere` (`internal/tendercli/deps.go`) waits `ownerPauseGrace`, ten seconds, before it interrupts, and then ends the run's registered processes without re-reading the owner.
An adopt that lands in that window records the owner here and lifts the owner pause before it exists, then dobby's pause lands: the mark stands, the pause covers the slug, D4 runs no pass until the covering pause lifts, and nobody lifts it.
`InnerRun`'s own owner check has the same window today, so this may be an accepted bound, but D4 states bounds elsewhere for far smaller windows and states none here.

## Settled focus questions

- Pass under a covering pause: D4 states how dobby treats a covered run (continue passes, no full pass bar the lift's, a pre-run pause refusal with its own exit code read as covered).
  The spawn-hold yield and the owner re-check are reachable only by a continue pass or a pause that begins mid-pass, and the acceptance bullets describe those; `check()` still runs `lockStep` and `noticeStep` under a pause, so the lock-release claim holds.
  A refused covered pass is counted toward neither the three-failures notice nor D7's marker, in D4, D5 and D7 alike.
  Not a defect.
- Owed full pass after an origin-unreachable resume: the pass writes the per-run marker itself, so a hand pass and the detached first pass of `lyx beam land` or `lyx adopt` leave it, and every pass outcome dobby reads is an exit code or a marker file, not output text.
  Not a defect.
- Failed owner pause, retry: re-judged first by a full pass, see F4 for the part this leaves open.
- `lyx adopt` refusals: D2 covers every entry the call records an owner for, a task or a beam, by the parent rule, and the acceptance bullet names both; the one gap is the position in F3.
- Draft status: see F1 for the commit point; the release command works as typed.
