---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md — D7, Constraints (PATTERN-durable-vs-ephemeral-state), Open risks bullet 2
    summary: The run's routing depends on the never-tracked amendment record, which the cited constraint forbids, and its loss on adoption is wider than the Open risk says
    fix: In D7 make the amendment record tracked content, for example `_lyx/loom/amendment.yaml` beside the decision record's tree, written by `lyx loom decision add --amends-plan` and staged in the same `commitDiscussion` commit as the decision entry, so the two land in one commit; have Plan-Bouncer's approved settle remove it inside its commit seam and PR-Rework's archive remove it with the generation. Rewrite the Constraints bullet to name the tracked path, and replace Open risks bullet 2 (adoption loss and re-recording) with nothing, since a tracked record survives adoption and needs no duplicate decision entry.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md — D7, guard admission and removal sentences
    summary: The start of the round that settle judged has no recorded source, and a round-0 skip-seam settle or a zero usage start leaves both the admission and the removal undefined
    fix: In D7 name the time source and its fallback, researched against the code. For example, the settle record gains a `round_started_at` field written by the Bouncer from the round's review-half start in `round-<N>-usage.yaml`, and the skip seam's round-0 settle writes its own settle time there. Then state that a record with no start reads as the settle time, so an amendment written after that settle is admitted and the next approved settle with a later start removes it. Add an acceptance criterion for an amendment on a rework-exempt (round-0) generation.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md — D7, failure-cleanup sentence; Acceptance criteria, the --amends-plan bullet
    summary: A failed second --amends-plan call removes the whole amendment record, dropping an earlier committed amendment that is still pending
    fix: In D7 replace "the verb removes the amendment record" with "the verb restores the amendment record to its bytes before the call, removing it only when the call created it". Extend the acceptance criterion so that a failed second call leaves the first call's title in the record. Under F1's tracked record this follows from restoring the working copy before the one commit.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md — D7, commit-failure restore sentence
    summary: A commit error after git commit has landed would restore the record to pre-append bytes while HEAD holds the entry, so the next discussion commit silently drops a committed decision
    fix: In D7 state that on a commit error the verb first reads the record as committed at HEAD (the existing `committedHoldsEntry`). When HEAD holds the entry it keeps the record and the amendment record and reports success with the error noted; only when HEAD lacks the entry does it restore and remove. Add the landed-commit case to the acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md — D7, Plan-Burler-Review answer-key instruction; D8, last paragraph
    summary: D7 makes each amending entry a finding for every card that does not carry it, while D8 words the entry as a change to named cards, so the instruction raises findings on every unnamed card
    fix: "In D7 reword the instruction to: when the amendment record exists, each decision-record entry it names is a finding for each card that entry names and that does not yet carry it, or, when the entry names no card, a finding until some card carries it; fixed by editing only cards Webster's auto-rebaseline accepts or by inserting a card after the last begun batch."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md — D9 steps 2 and 3, and the untraced definition
    summary: A location quarry answers not_found (outside every member, such as an unused-import build error) is neither matched nor untraced
    fix: "In D9 step 3 add: a not_found answer matches through its Unit, the file's self glyph, every live card whose targets hold that file or its unit; with no such card it goes into untraced. Add an acceptance criterion: an unused-import build error in a file a card targets names that card."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md — D9 step 3, the overlap rule
    summary: The overlap rule compares found glyphs with flat targets as written, so a test or member a card creates under a plan handle never matches its own failure
    fix: In D9 step 3 state that each target is read through `internal/planparser`'s ref-to-member mapping (`RefMember`), so a `plan:` handle compares as the glyph it becomes, before the overlap rule applies. Add an acceptance criterion where a failing test a card creates under a `plan:` handle names that card.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md — D9 step 3 and the PublishFailureNote sentence
    summary: The premise that no per-card commit exists is false, and the batch range misses the commits that can cause the failure
    fix: "In D9 drop the claim that no per-card commit exists: Webster commits each card but records only the batch head. Define the range for every batch state: start SHA to recorded head for a done batch, and start SHA to the record's Head for a failed or in-flight batch with no recorded head. Have the note also name the range from the latest matched batch head to the record's Head as later commits touching the same members (verify-gate fixer `fix:` commits, Webster-Review fixes, the merge-in), so the fixer is pointed at both."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md — D5, the additive guess
    summary: The wire guess matches member glyphs whose name starts with wire, which misses the method glyphs loomCLI.wire and battenCLI.wire the estimate counted
    fix: "In D5 change the guess to: a member glyph whose last dotted segment starts with `wire` or is `addWeftVerbs`, so `internal/loomcli#loomCLI.wire` is guessed additive as the orch's estimate treated it."
  - id: F10
    severity: LOW
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md — D7, the latest write time in whole-second UTC
    summary: A whole-second write time compared with a sub-second round start reads an amendment written in the round's first second as before the round
    fix: In D7 state that the write time and the round start are compared at the same precision, truncating both to whole seconds with an amendment at an equal second read as written after the start; or record the write time at full RFC 3339 nanosecond precision.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md — D1 paragraph 1 against D4
    summary: D1's later-card-wins rule for two cards fencing one entry ID needs an order that D4 removes for two Extends on PATTERN.md
    fix: In D1 add that two cards fencing a line for one entry ID make a dependency edge whatever their labels, decided by D4's pair rule, so the later card still runs later once batches run in parallel; add the pair to D4's rule text.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md — D4 Rationale
    summary: The claim that one function keeps the rule from drifting covers only the pair verdict, while Webster and plandag still detect overlap differently
    fix: In D4's Rationale narrow the sentence to say the function decides the edge for a pair its caller already found overlapping, and that overlap detection (Webster's exact-string match, plandag's member containment) stays each caller's.
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md — Acceptance criteria, the orch stencil bullet; D8
    summary: The orch stencil already forbids editing _lyx/plan in a task pair, so no longer allows editing a card describes no change
    fix: "Reword the acceptance criterion to: The orch stencil names `--amends-plan`, then `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume`, as the route for a design call that amends the plan."
---

### [BLOCKING] The amendment record breaks the durable-vs-ephemeral constraint the record cites

**Location:** D7; Constraints, the `PATTERN-durable-vs-ephemeral-state` bullet; Open risks bullet 2.

**Issue:** The settled-segment guard admits the goto only when the amendment record exists. The Plan-Burler-Review instruction also reads it, and Plan-Bouncer's settle removes it, so the run's routing depends on it.

`pattern/PATTERN-durable-vs-ephemeral-state.md` says: "Nothing a run, the orch or a landing depends on lives only under `.lyx`".

The record places the file under `.lyx/loom/` and cites the constraint as met. Its own second Open risk shows the dependency:
- On adoption the goto is refused.
- The stated recovery re-runs `--amends-plan`, which appends a duplicate `Added after Discussion` entry to the answer key.

The loss is also wider than "before the goto". A run adopted after the goto but before the approved settle re-runs the round without the record, so the amendment is no longer forced as a finding.

Making the record tracked removes the constraint break, the Open risk and the duplicate entry together. Committing it in the decision entry's own commit also gives D7's "land together or not at all" by construction.

### [MEDIUM] The round start the guard and removal compare against has no recorded source

**Location:** D7, guard admission and removal.

**Issue:** The settle record holds only `Round` and `SettledAt` (`internal/shedadapters/settlerecord.go`). The only round start on disk is the review half's `start` in `round-<N>-usage.yaml` (`internal/shedadapters/roundusage.go`), which has these gaps:
- It is written after the half ends.
- A failed write of it only logs a Warn.
- It is zero for a half with no terminal result (`internal/burlerengine/engine.go:74`).

A rework-exempt generation settles at round 0 through `approveWithoutReview`, which judges no round at all.

The record leaves this fact for the implementer to find, which the rubric's deferred-research rule flags. A zero start also never compares as after a write, so the approved settle would never remove the record. The guard would then keep admitting gotos onto a settled Plan-Review, which breaks D7's stated bound.

### [MEDIUM] A failed second `--amends-plan` call erases the first amendment

**Location:** D7, the failure-cleanup sentence.

**Issue:** D7 appends later titles to one record, but on an append or commit failure it "removes the amendment record". Two calls show the loss:
1. Call 1 commits decision A, so the record lists A.
2. Call 2 fails at `commitDiscussion`, for example on the weft write lock (`internal/fabricengine/commitweftpaths.go:69-75`), and removes the whole record.

A stays committed but no longer routable. The goto is refused, and nothing tells the caller that A needs re-recording.

### [MEDIUM] A commit error after the commit landed undoes a committed decision

**Location:** D7, the commit-failure restore.

**Issue:** `gitrepo.StageAndCommit` can return an error after `git commit` has landed, for example when the follow-up SHA read fails. D7 then restores the working copy to its pre-append bytes while HEAD holds the entry.

The next commit of the record commits the removal. Both `commitDiscussion` and `writeCarryOver` (`internal/loomcli/wiring.go:379-400`) commit the decision record, and a loom run's discussion check reads no history (the discussion spec, "Parts no writer authors"). So the entry vanishes unflagged.

The verb already has `committedHoldsEntry` for the darn path.

### [MEDIUM] The answer-key instruction fires on every unnamed card

**Location:** D7's Plan-Burler-Review instruction; D8.

**Issue:** D7 makes each named entry "a finding for every card that does not yet carry it". D8 requires the entry to be worded "as a concrete change to named cards".

Read literally, every card the entry does not name is a finding. That contradicts the rubric's "carried by some card" fidelity rule (`contracts/stencils/loom/loom-rubric-plan-review.md:78-79`), and the fixer would chase findings it cannot satisfy.

### [MEDIUM] Locations outside every member fall through the trace

**Location:** D9 steps 2 and 3, and `untraced`.

**Issue:** quarry v0.4.0 answers a range inside no member as `not_found`, with `Unit` set to the file's self glyph (`internal/engine/enclose.go:31-40`). That answer is neither a rejection nor a found glyph.

D9 matches only found glyphs, and `untraced` covers only rejected or unheld locations. A Go build error such as an unused import sits on an import line, so it goes neither to a card nor into `untraced`.

### [MEDIUM] Targets declared as `plan:` handles never match a traced glyph

**Location:** D9 step 3.

**Issue:** The overlap rule compares a found glyph with flat targets as written. A test a card creates is declared as a `Create` `plan:` handle, which is the form `edit-test-file`'s way forward prescribes (`internal/planparser/edittestfile.go`).

The test's failure resolves to `internal/x#TestFoo`, which equals no target string. So the card's own new test, the most common trace target, names no card.

`planparser.RefMember` already maps a handle to the glyph it becomes, but D9 does not route the matching through it.

### [MEDIUM] The batch commit range rests on a false premise and misses later commits

**Location:** D9 step 3; the `PublishFailureNote` sentence.

**Issue:** `internal/websterengine/doc.go:70` says each card lands as its own commit; Webster only records the batch head (`recordbatch.go:585`, `CardSHAs = []string{report.HeadSHA}`). So "no per-card commit exists" is false.

The range has three further gaps:
- It is undefined for a failed or in-flight batch, which has a start SHA but no recorded head.
- It omits the commits that rewrite members after the batch: the verify-gate fixer's `fix:` commits, Webster-Review fixes and the merge-in.
- A failure caused by one of those commits points the fixer at a range that does not contain the cause.

### [MEDIUM] The wire guess misses the method glyphs it is meant to count

**Location:** D5.

**Issue:** The board entry's hot spots `loomCLI.wire` and `battenCLI.wire` are methods. Their member glyph is `T.M`, as in `internal/loomcli#loomCLI.wire`, and does not start with `wire`.

So `-additive-guess` does not reproduce the orch's "plus wire additive" column. Repeating that estimate is D5's stated purpose.

### [LOW] The whole-second write time mis-orders an amendment in the round's first second

**Location:** D7.

**Issue:** The round start is a sub-second `time.Time` (`halfUsage.Start`), while D7 records the amendment's write time in whole seconds.

Suppose an amendment is written 0.3 s after the round starts. Truncated, it reads as before the start, so two things go wrong:
- The guard refuses the goto.
- The approved settle removes the record, although the round never saw the amendment.

### [LOW] D1's card-order rule leans on an order D4 removes

**Location:** D1 against D4.

**Issue:** D1 lets the later card win "since the plan runs in card order". D4 makes two `Extends` on `PATTERN.md` no dependency, and it is placed for parallel batches, where that order no longer holds.

This is bounded today by card-order execution.

### [LOW] D4 overstates what the shared function unifies

**Location:** D4 Rationale.

**Issue:** Overlap detection still differs between the two callers:
- Webster's `refsIntersect` compares exact strings (`internal/websterengine/sequence.go`).
- plandag's `columnsOverlap` also matches a member of a type (`tools/plandag/graph.go`).

For example, `Extends` on `pkg#T` against `Edit` on `pkg#T.M` is an edge in plandag and nothing in Webster. So "keeps the rule from drifting" overstates what D4 unifies.

### [NIT] The orch-stencil criterion describes no change

**Location:** Acceptance criteria, orch stencil bullet.

**Issue:** `contracts/stencils/orch/orch-template-role.md:109` already says "Never edit or commit in either worktree of a task pair, `_lyx/plan` and the decision record included". Only the hub orch rule, which is out of scope, allows a hand edit.

## Rejected

- D3's "every reader of an Edit group reads Extends alike", flagged as a call-site gap. Enumerating call sites is excluded by the rubric's Do-not-flag list, and the Notes already flag that `deriveEdges` needs group types.
- D9's citation of `PATTERN-tolerant-records` for a YAML record. That entry's scan covers JSON only, but the YAML decode is already non-strict, so the stated guarantee holds and the citation is harmless.
