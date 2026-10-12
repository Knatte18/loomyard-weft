---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D13 (lines 177-182)
    summary: D13 says Shed records a reason at each paused persist, but shedengine.Status has no reason field, so no persist records one, pause_before's included; the goto is not a step stop at all, and under D12's status re-read every routine goto would still read as an empty-reason paused stop and file a note.
    fix: Rewrite D13's Decision. Its premise becomes that the reason lives only on the step result and envelope, and a status file carries none. Name where the new reason goes. Either (a) StepResult.Reason is set on the bare pause_requested pause (pause requested) and on the cancelled-context pause in both stepLocked branches (cancelled), and D12's status re-read classifies a paused status by its last history entry (an OutcomeGoto entry is a moved run, never an empty-reason paused stop) rather than by a reason; or (b) a new tolerant reason field on shedengine.Status is written by every persist that records StatePaused, Goto and Relaunch included. Recommend (a), since it adds no status schema. Drop the goto from the list of persists that yield an empty-reason paused stop. Restate the rationale so it no longer claims the goto produces such a stop. Update the D13 acceptance criterion at line 251 to match.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (line 90)
    summary: D4 has the Go driver, running in the task worktree, write batten's publish-retry marker, a file private to internal/battenshed under the batten prime run's .lyx scratch directory in another worktree, but it never says how the Go driver locates it or what happens when no batten prime run exists.
    fix: Reverse the direction in D4. Batten's publishRetryDecision treats the run's one retry as spent when the child's durable Go settle records (D9, in the task worktree's friction directory, which batten already reaches through the child's location) show a Go verify retry. The Go driver reads batten's publish-retry marker only through a battenshed-exported read that takes the prime's scratch directory, and only when the batten-watched marker names a prime. Remove the sentence saying the Go driver writes batten's marker. State that the count lives in two places, each written by its own actor and read by the other. Update the line 239 acceptance criterion to say the Go driver's spent retry is read by batten from the settle records.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D7 (line 123) against D12 (lines 172-175)
    summary: D7 reads any stop-key or uncommitted-path change during a tackler's life as hand-back with cause tampered, while D12 says a run another actor moved is a new stop; an orch goto or an operator edit made while the tackler runs meets both rules, and D7 blames the tackler for it.
    fix: In D7, order the checks. After the tackler ends, D12's re-read runs first. A stop key that changed and whose latest history entry or status write is not attributable to the tackler is a moved run, and D12 handles it as a new stop. Cause tampered is kept for a change the run's records attribute to no other actor. If attribution is not decidable, say so instead and send every change to D12 as a new stop, with the tackler's verdict discarded and a stop-report line noting the change. Add an acceptance criterion that an orch goto during a tackler wait is handled as a new stop, not as tampered.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (line 123)
    summary: The only Go-side bound on a tackler's mutations is that the stop key and the set of uncommitted paths are unchanged, so a tackler that commits its edits, moves HEAD, or edits a path that was already dirty passes the check.
    fix: In D7's sentence at line 123, extend what is recorded at spawn and compared after the tackler ends. The comparison covers the stop key, the worktree's HEAD, the checked-out branch, and the uncommitted paths with their content hashes (the existing dirty-paths read plus a hash per path). A difference in any of them reads as hand-back with cause tampered, subject to the ordering of F3. Add the HEAD and content cases to the tampered acceptance criterion at line 240.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3 binary row (line 70) against D5 (line 96)
    summary: D3's hand-over row keys on the envelope's binary_changed, which shedverbs/steprecord.go computes against the calling lyx's own build, while D5 says the Go driver judges by the on-disk build and never its in-memory one; read in-process by an old Go driver after a deploy, binary_changed is false.
    fix: Change D3's row at line 70 from "binary-eligible stop with binary_changed true" to "binary-eligible stop whose last step's recorded build differs from the on-disk lyx build (D5)". In D5, state that the Go driver reads the on-disk build identity itself and never uses the binary_changed field. Keep the acceptance criterion at line 250 as written.
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D7 (lines 124, 132)
    summary: A tackler goto is bounded only by CheckGotoTarget's at-or-before-the-current-row rule, so a cheap model's verdict can rewind a run to its first row and re-run every earlier LLM row, which costs more than the arbiter spawn the tackler exists to avoid.
    fix: Make the choice in D7. The recommended choice is that a tackler goto may name only the current row or the row the halting history entry routed from. Any target further back is not admitted, and the verdict reads as hand-back with cause precondition. State the bound in D7 at line 132 and add a malformed-case row to the acceptance criterion at line 241.
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md Question ledger Q10
    summary: Q10 records a Go friction note at every sighting of an empty-reason paused stop, while D13 writes one only for a paused stop whose reason is still empty after the shed change.
    fix: Reword Q10 to match D13. Shed records a reason where the stop arises, and the Go driver writes a friction note only for a paused stop whose reason is still empty.
---

### [BLOCKING] D13 rests on a reason field the status file does not have

**Location:** `_lyx/discussion/decision-record.md` D13, lines 177-182.

**Issue:** `shedengine.Status` (`internal/shedengine/status.go`) has no reason field.
`persistTransient` (`internal/shedengine/run.go:731`) writes state, error, transient, parent notice and history, never a reason.
The `pause_before` path at `run.go:224-228` puts its reason only on the returned `StepResult`, not on disk.
So "a reason at each of those persists" names nothing that exists, and the decision does not say whether it means a new status field or `StepResult.Reason`.
The goto premise is also wrong for the step envelope.
`Goto` (`internal/shedengine/goto.go:89-97`) persists `StatePaused`, but a later step on that run passes the pause check and goes straight to the resume write at step 3b, so a goto never produces a `paused` step stop.
The only paths that do are the bare `pause_requested` pause (`run.go:230-238`) and the cancelled context (`run.go:317-321`).
The goto reaches the Go driver only through D12's status re-read, and a status read has no reason to check.
So under D13 as written, every orch goto made while the Go driver waits reads as an empty-reason `paused` stop and files a friction note.
D13's own rationale rules that out.

### [MEDIUM] D4's shared retry count writes a marker the Go driver cannot locate

**Location:** `_lyx/discussion/decision-record.md` D4, line 90.

**Issue:** batten's publish-retry marker is `<scratchDir>/InnerRun-publish-retry` (`internal/battenshed/publishretry.go:73-76, 99-108`).
`scratchDir` is the batten prime run's `.lyx` scratch directory, in the prime's worktree, not the task's.
The marker format and its states (`started`, `goto-done`, `resumed`, `capped`) are unexported battenshed internals.
D4 has the Go driver, running in the task worktree, write that file "so neither a restart nor batten adopting the run later retries again".
It never says how the driver finds the path.
In the unwatched case D4 is about, often no batten prime run exists, so there is no directory to write into.
The child's own settle records (D9) are durable, local to the task, and already one of the two sources D4 names, so having batten read them closes the same race without a cross-worktree write.

### [MEDIUM] D7's tampered check and D12's moved-run rule claim the same event

**Location:** `_lyx/discussion/decision-record.md` D7 line 123, D12 lines 172-175.

**Issue:** D15 tells the orch to reach a go-driven run through `lyx shed goto` and `lyx loom` verbs.
D2 keeps the run lock free during a settle wait so that the orch, the operator and batten can move the run.
An orch goto during a tackler wait changes the stop key, which `loomengine.StopKeyOf` defines as producer, history length and state.
D7 reads that change as `hand-back` with cause `tampered`, blaming the tackler.
D12 says the same event is a new stop.
The two rules disagree on the action (hand back or act afresh) and on the friction it leaves.

### [MEDIUM] The tackler's tamper bound misses commits and edits to already-dirty files

**Location:** `_lyx/discussion/decision-record.md` D7, line 123.

**Issue:** the rubric's attack-surface question asks what bounds the tackler beyond stencil text.
D7's answer is the post-run comparison of the stop key and the worktree's uncommitted paths.
A tackler that edits and commits leaves the uncommitted-path set unchanged, and so does one that moves HEAD with a reset or checkout.
Editing a file that was already dirty leaves the set unchanged too, because the set holds paths and not content.
D7 states that the read-only rule is stencil instruction only, so this comparison is the whole mechanical bound, and it lets all three through.
The consume side does hold: the tackler `resume` goes through `shedengine.Relaunch` with an Admit under the status lock (`internal/loomcli/arbiterconsume.go:53-62`), as the focus directive asks.

### [MEDIUM] D3 keys the hand-over on a field D5 says not to trust

**Location:** `_lyx/discussion/decision-record.md` D3 line 70, D5 line 96.

**Issue:** `internal/shedverbs/steprecord.go:39,135,184` computes `binary_changed` against "the build identity of the calling lyx".
A Go driver that reads status in-process after a deploy is still the old build, so the field reads false.
D5 says the judgement is by the on-disk build and never by the driver's own in-memory build.
D3's row names `binary_changed`, so a plan that follows the table would miss exactly the deploy D5 exists to catch.

### [LOW] A tackler goto can rewind the run to its first row

**Location:** `_lyx/discussion/decision-record.md` D7, lines 124 and 132.

**Issue:** `CheckGotoTarget` and `gotoAdmitted` (`internal/shedengine/goto.go:110-126, 214-254`) admit every row before the reference row.
D7 does state a bound ("at or before the current row", one move per attempt, `tackler_max_spawns` per run), so this is not an unbounded edge.
That bound still lets a cheap model send a Finalize halt back to Plan-Write or Discussion, which re-runs every later LLM row.
The tier's purpose is to avoid opus cost, and an unrestricted goto defeats it.
How far back a tackler may move the run is a decision the record leaves open.

### [NIT] Q10 disagrees with D13

**Location:** `_lyx/discussion/support-log.md`, Question ledger Q10.

**Issue:** Q10 says "a Go friction note at every sighting".
D13 narrows that to a stop whose reason is still empty after the shed change.

## Focus verdicts

- D7 (tackler resume and goto): a stale `resume` is bounded mechanically.
  It goes through `shedengine.Relaunch`, under the run lock and one status-lock hold, with an Admit that compares the stop key and the pause conditions recorded at spawn, as `arbiterconsume.go` does.
  A refusal becomes a new stop (D12).
  A wrong `goto` is bounded by `CheckGotoTarget` to rows at or before the current one (F6).
  What the tackler does to the worktree itself is bounded only by the tamper comparison, which leaks (F4) and collides with D12 (F3).
- D4 and D3 (verify failure goto): the match test D4 states is `recordMatchesHalt`'s.
  It requires the target, the write time, the head, the last entry being a Stuck of the record's row, and the time window, and it is as strict as batten's.
  A `blocked` stop with a non-matching record falls through D3 to the binary row if one applies, and otherwise to the last row (tackler).
  Where the one-retry count lives is the defect in F2.
- D5 (hand-over): D5 defines the clean-tree guard itself, at line 97.
  A crash or failed spawn after the lock release leaves a parked run with a stop report.
  When watched, batten's InnerRun re-step condition holds, because the last step's recorded build still differs from the on-disk one.
  When unwatched, the hand-back line has already been sent before the release.
  If batten's re-step starts a second `drive` while the fresh one is still spawning, D2's held-lock refusal settles it.
  Not a defect.
- D1 (typed `--child-driver` against a seed recording none): refused, naming both ways forward.
  This is the typed-against-recorded carve-out `PATTERN-driver-choice-single-site` allows, because a typed value is present to compare.
  Not a defect.
- D13 (empty-reason paused stop): its premise fails against `internal/shedengine` (F1).
