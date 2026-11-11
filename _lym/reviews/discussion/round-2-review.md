---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:73
    summary: D2 puts a loom-shaped read, parsing the Lym-Record-Replaced value and cutting history by the record's hash, inside internal/fabricengine, against PATTERN-fabric-boundary's Fabric isolation rule
    fix: In D2, replace "The read lives in internal/fabricengine, beside fabricengine.CommittedAnchoredFileHistory" with a split. internal/fabricengine gains only general seams any module can take, for example every first-parent weft commit carrying a given trailer key with its value, newest first, and an anchored file's history returned with each version's commit SHA. internal/loomcli owns the Lym-Record-Replaced key, parses its by, path and sha256 fields, and does the cut in recordHistory. Restate the PATTERN-fabric-boundary constraint line to match.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:124
    summary: A raddle run that adopts the motif pair's still-live weft keeps the motif run's committed record, because Record-Write never overwrites a file, so the --record-file record never reaches raddle; Open risks bullet 4 misstates both the effect and the remedy, and the live window is wider than D4 says
    fix: Decide the adopted-weft case in D4. Either refuse --record-file while the slug's weft branch is live (local, or on origin with no archive/<slug>/* tag covering its tip), with a way forward to wait for or finish the teardown, or have Record-Write re-seed the record from the board's stored record when the two differ before the run's first record commit of its own. Count the detached teardown child's in-flight window, between the local delete and origin's delete, as live. Then rewrite Open risks bullet 4 to the real outcome, since the stale motif record is read and nothing flags it, or drop the bullet once the chosen rule closes the case.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:91
    summary: D3 copies the record to the board whenever the entry carries a stored record, so a motif or tapestry run sent back into its Discussion segment after an earlier approval copies an in-review record, breaking the rule that a call made while Discussion-Bouncer settles is not copied
    fix: In D3, limit the stored-record condition, and store's matching admission, to a run whose recipe has no Discussion approval row. Keep the past-approval gate for every recipe that has one. Add an acceptance criterion that add on a run re-entered into its Discussion segment, whose entry carries a stored record, leaves the stored record unchanged.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:32
    summary: D1 refuses replace only while the run stands at a Webster row, but a fold made after plan approval and before Webster, or before a goto back into Webster, removes an added entry from every Master spawned later, since Master renders only Added after Discussion entries
    fix: Extend D1's Webster refusal to every row from the plan's approval through the recipe's last Webster row, keeping the way forward that names add now and a later replace once the run has left its Webster rows. Change the acceptance criterion at line 200 to cover a run standing at a row between plan approval and Webster as well.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:74
    summary: D2 cuts at the newest version whose bytes match the trailer, so a sweep that later commits the replaced bytes again, after an added entry was dropped, moves the cut past that drop and the drop is never flagged, against the D2 acceptance criterion at line 209
    fix: In D2, cut at the newest version that matches the trailer's sha256 and was committed at or before the trailer commit, not the newest overall. Add an acceptance criterion where a replace writes X, add makes Y, a writer restores X and a sweep commits it, and the check still reports added-decision-dropped.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:157
    summary: D6 tells the parent to fold every added entry into the main decisions, but on a Record-Write record such as darn's or pawl's the scope is the fenced Goal body with added entries over it, and darn and drift give precedence only to added entries, so a folded D-entry loses its precedence over the body
    fix: In D6, state how a fold is written on a Record-Write record. Either the fold rewrites the Goal's fenced body to carry the folded call, or replace on a Record-Write record keeps the added entries it would otherwise fold. Then name in D1 which of the two replace enforces or permits on such a record.
  - id: F7
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:96
    summary: FileRaddle saves board.json and the project's record before it writes the raddle store's done marker, so a crash or failed store write leaves raddleRunFiling reporting not filed, and a D3 copy in that window overwrites the family project's record, which the retry does not repair
    fix: In D3, key the filed refusal also on the board, and refuse add, replace and store when the run's entry is no longer a task, or when the board already holds the filing as boardengine.Board.HoldsRaddleFiling judges it, not on raddleRunFiling alone. Add an acceptance criterion for a family filing whose store write failed.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:61
    summary: D1's bound names only dropping added entries and appended bullets, but the file may also rewrite an added entry or insert a new one, which IdentityText strips so no re-review sees it, while dropping an appended PATTERN bullet does change reviewed text
    fix: In D1, either refuse a file whose Added after Discussion entries are not a subset, byte for byte, of the current record's entries, or state in the Bound that replace may rewrite or insert added entries unreviewed. Correct the IdentityText sentence so it names appended PATTERN bullets as reviewed text.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:85
    summary: D2's bound says only a lym verb writes a records-branch commit message, and D4's says it excludes a run's agent, but no shell guard denies git commit in _lym and the strand check reads an environment variable the caller can unset, so both bounds overstate their guards
    fix: Restate D2's Bound so the trailer guards against a mistaken writer, not a forged commit, since a commit made outside lym can carry the trailer. Restate D4's Bound so the run-agent refusal guards against a mistaken call, not a forged or unset strand name, as decision.go states for its own guard.
  - id: F10
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:157
    summary: The run report counts post-Discussion decisions per --by author from Added after Discussion entries, so a fold makes the report undercount, and the record is silent on whether that is accepted
    fix: Add one line to D6 or Open risks saying the run report counts only the added entries a record still holds, so a folded call drops out of its decision count. Alternatively decide that the report also reads the Lym-Record-Replaced trailers, and add that to D2.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:171
    summary: The PATTERN-refusal-way-forward constraint names the refusals of D1 and D4, but D3 adds new loom refusals the entry binds, while D4's refusals sit in internal/boardcli, which the entry does not bind
    fix: Change the constraint line to every new refusal of D1 and D3, which lands with a test that reaches it, and say that D4's boardcli refusals name their way forward too though the entry does not bind boardcli.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:228
    summary: A note for the plan writer asks Plan to confirm whether git records an empty commit limited to the record's path while another path is staged, a fact D1's trailer commit rests on, so it is a deferred research question
    fix: Delete the note at line 228, and add to D1's empty-commit bullet the established fact. With git 2.53.0, git commit --allow-empty -m <msg> -- <record path>, run while another path is staged and the record is unchanged, lands an empty commit and leaves the other path staged (the experiment this review ran).
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md:88
    summary: The support log's Q7 and its Turn 3 bound-breaker 5 line say D4 refuses a slug with a live batten run, but D4 reads no batten liveness and refuses a slug with a task worktree in the hub
    fix: In support-log.md, change Q7 to refused while a run holds the entry or the slug has a task worktree in the hub, and refused for a run's task session. Change the bound-breaker 5 line at line 54 to say D4 refuses an entry whose slug has a task worktree in the hub.
  - id: F14
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:49
    summary: The decision record carries deliberation that belongs in the support log, namely the rejected gitrepo.Repo.CommitEmpty in D1, why D4 reads no batten liveness, and the mechanics of batten teardown in D4's weft bullet
    fix: Move to support-log.md's Rejected alternatives the D1 sentence at line 49, which rejects gitrepo.Repo.CommitEmpty. Move to the support log the explanation at lines 115-116, which covers boardcli's imports and where batten liveness is read, and keep the D4 lines It reads no batten liveness and The run lock and the worktree check are its run guards. In D4's weft bullet, keep the rule for when the motif pair's weft branch is still live and adopted. Move the teardown mechanics at lines 126-127 to the support log.
  - id: F15
    severity: NIT
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:139
    summary: An older binary on another machine that stores a record keeps record_body_sha256 as an unknown key without restamping it, so the promote warning can fire spuriously or stay silent, a bound D5 does not state
    fix: Add to D5's Bound that a record stored by a binary older than this task leaves record_body_sha256 as it was, so the warning can be wrong until a newer binary stores the record again.
---

### [BLOCKING] D2's history read is a loom-shaped seam inside fabric

**Location:** `_lym/discussion/decision-record.md:73`, with the constraint at line 172.

**Issue:** `PATTERN-fabric-boundary`'s Fabric isolation rule says a seam fabric offers is general: "never a call, field or type shaped for one consumer", and "a seam only one module could use belongs in that module".
D2 has the read find the newest commit whose `Lym-Record-Replaced` trailer names this record's path.
That read parses the trailer's `<by> <path> <sha256>` value and returns versions from the one whose bytes hash to that sha256.
D2 places all of it in `internal/fabricengine`.
That knowledge belongs to loom's replace verb alone.
The existing seams are general: `fabricengine.FindTrailerCommit` matches a key, and `fabricengine.CommittedAnchoredFileHistory` reads a path.
The constraint line asserts compliance with the entry, so the record contradicts its own constraint.
Every violation of a listed constraint is BLOCKING under this round's rules.

### [MEDIUM] An adopted live motif weft keeps the stale record, and the Open risk misstates it

**Location:** `_lym/discussion/decision-record.md:124`, with Open risks at lines 188-190.

**Issue:** `Topology.Add` adopts a live weft branch with its run records (`internal/fabricengine/add.go:150`).
The motif and raddle runs share `_lym/discussion/decision-record.md` (`internal/loomcli/projectrecord.go:51-57`).
Record-Write writes the record only when the file is missing (`internal/loomshed/recordwrite.go:120-124`, `:153`).
So the raddle run reads the motif run's unfolded record, the board record set with `--record-file` never reaches it, and the first D3 copy overwrites the board record with the stale one.
Open risks bullet 4 assumes the working record is the folded one, so its consequence (the dropped-entry check flags folded entries) and its remedy (a self-replace) are both wrong.
The detached teardown child also leaves origin's branch untagged until it finishes, so the live window covers that in-flight span too, not only a standing pair or a failed remote step.
The interactions, walkthrough and bound-breaker lenses each raised this.

### [MEDIUM] D3's stored-record copy fires during a re-entered Discussion review

**Location:** `_lym/discussion/decision-record.md:91`.

**Issue:** A run of a recipe with an approval row that is sent back into its Discussion segment reads as not past approval (`internal/loomrecipe/discussionapproval.go`).
Its entry still carries the record stored at the earlier approval.
So D3's second condition copies an unapproved, in-review record onto the board.
That breaks the settling-window rule at `internal/loomcli/decision.go:5-6`.
The walkthrough and bound-breaker lenses both raised it.

### [MEDIUM] A fold before Webster takes an added entry out of Master's view

**Location:** `_lym/discussion/decision-record.md:32`.

**Issue:** Master reads no record and renders only the record's `Added after Discussion` entries into `parent_decisions` at each fresh spawn (`internal/websterengine/doc.go:338-342`).
Suppose `add` runs without `--amends-plan` after plan approval, at a Drift or Plan-Review row, and a `replace` folds that entry before Webster starts.
Then no Master sees the call, and the same holds after a goto back into Webster.
D1 refuses only while the run stands at a Webster row.
The existing-paths and bound-breaker lenses both raised it.

### [MEDIUM] D2's cut can move past a later committed drop

**Location:** `_lym/discussion/decision-record.md:74`.

**Issue:** Take a replace that writes X, then an `add` that makes Y.
A writer then deletes the new entry, which leaves exactly X, and a status sweep commits it with no gate.
The history is now X (sweep), Y, X (replace), and the newest version matching the trailer is the sweep.
The cut keeps only that sweep, so `discussionparser.CheckAddedDecisionsKept` never sees Y.
That breaks the D2 criterion at line 209, and `checkDarnRecord` then admits the record.

### [MEDIUM] D6's fold drops the precedence Record-Write readers give added entries

**Location:** `_lym/discussion/decision-record.md:157`.

**Issue:** On a Record-Write record, the Goal is the fenced board body and the Decisions section starts empty.
Darn's scope is "the board entry's body under `## Goal`, with the record's `Added after Discussion` entries over it" (`internal/darnengine/doc.go:64`).
Darn's gate lets only such an entry override a finding (`internal/loomshed/darnwrite.go:47`).
Drift's seats give added entries precedence over the rest (`internal/driftengine/seats.go:104`).
A call folded into a D-entry that contradicts the fenced body loses that precedence, and D6 does not say how a fold is written on such a record.

### [LOW] D3's filed refusal misses the crash window inside FileRaddle

**Location:** `_lym/discussion/decision-record.md:96`.

**Issue:** `boardengine.Board.FileRaddle` saves `board.json` and the project's record file, then writes `store.json` last (`internal/boardengine/raddlefile.go:146-150`, `:337-361`).
`raddleRunFiling` reports done only from the store (`internal/loomcli/raddlewiring.go:82-97`).
A failed store write leaves a converted project with a "not filed" run.
A D3 copy then overwrites the project's record, and the retry takes the held-filing path and writes only the store.
Today no copy reaches a raddle run, since the raddle recipe has no approval row; D3 opens this path.

### [LOW] D1's bound misdescribes what replace changes unreviewed

**Location:** `_lym/discussion/decision-record.md:61`.

**Issue:** D1 lets the file "drop or rewrite" added entries and forbids neither rewriting nor inserting one.
`discussionparser.IdentityText` strips every added entry, so a rewritten or new one changes no reviewed text (`internal/discussionparser/identity.go:11-18`).
It keeps appended PATTERN bullets, so dropping one does change reviewed text.
The Bound states only the drop case.

### [LOW] D2 and D4 bounds overstate their guards

**Location:** `_lym/discussion/decision-record.md:85` and `:132`.

**Issue:** `internal/shellguard` holds no `git commit` deny, so a session or the operator can commit in `_lym` with a forged `Lym-Record-Replaced` trailer.
That is the forgery the support log rejected the content marker for (`_lym/discussion/support-log.md:68`).
D4's run-agent refusal reads the strand name from the environment (`internal/boardengine/taskguard.go:14-17`), which an agent can unset.
`internal/loomcli/decision.go:9` already states that limit for `add`'s guard.
Round 1's handler note held the forgery question because `lym fabric commit` writes the fixed message `weft sync`.
That covers lym's own verb only, not a raw `git commit` in `_lym`, which no guard refuses.

### [LOW] A fold makes the run report undercount decisions

**Location:** `_lym/discussion/decision-record.md:157`.

**Issue:** `internal/reportengine/rounds.go` counts post-Discussion decisions per `--by` author from the record's `Added after Discussion` entries.
After a fold the count drops, and the record does not say whether that is accepted.

### [LOW] The refusal-way-forward constraint names the wrong decisions

**Location:** `_lym/discussion/decision-record.md:171`.

**Issue:** D3 adds new refusals in `internal/loomcli`: `add`, `replace` and `store` once the raddle filing is done.
The entry binds `internal/loom*` (`pattern/PATTERN-refusal-way-forward.md:4`), but the constraint omits them.
It names D4's refusals in `internal/boardcli`, which the entry does not bind.

### [LOW] A note leaves the empty-commit fact for Plan to confirm

**Location:** `_lym/discussion/decision-record.md:228`.

**Issue:** D1's trailer commit, made while another path stays staged, rests on git landing an empty commit limited to one path.
The note hands that check to Plan, which is a deferred research question.
This review ran it on git 2.53.0 in a scratch repository.
With `rec.txt` committed and unchanged and `other.txt` modified and staged, `git commit --allow-empty -m "trailer test" -- rec.txt` exited 0 and landed a commit with no file change.
`other.txt` stayed staged.
The fact holds, so only the record's text needs to change.

### [LOW] The support log still says D4 checks batten liveness

**Location:** `_lym/discussion/support-log.md:88`, and line 54.

**Issue:** Q7 and the Turn 3 bound-breaker 5 line say D4 refuses a slug with a live batten run.
D4 now reads no batten liveness and refuses a slug with a task worktree in the hub (`_lym/discussion/decision-record.md:108`, `:114`).

### [LOW] Deliberation sits in the decision record

**Location:** `_lym/discussion/decision-record.md:49`, `:114-117` and `:124-129`.

**Issue:** D1's sentence about `gitrepo.Repo.CommitEmpty` not fitting is a rejected alternative.
D4's account of `internal/boardcli`'s imports and of where batten liveness is read is deliberation.
So are the teardown mechanics in D4's weft bullet.
Each of these explains a choice and gives the plan writer no requirement.
Under completeness before leanness, the requirement lines stay in the record: no batten liveness read, the run lock and worktree check as run guards, and the live-weft rule.
Only the narrative around them moves.

### [NIT] D5 does not bound the stamp under mixed binaries

**Location:** `_lym/discussion/decision-record.md:139`.

**Issue:** A binary older than this task stores a record through `boardengine.Board.StoreDecisionRecord` without restamping `record_body_sha256`.
It keeps the stale key, so the advisory warning can be wrong until a newer binary stores the record.

## Rejected

- existing-paths F2, a replace on a non-perch raddle run halted at Raddle-File after staging. That gap belongs to `add` today: a decision recorded at that row in non-perch mode already reaches no filed part, and D1 admits `replace` only where `add` is admitted, so the record adds no new path to it.
- bound-breaker F7 is the same defect as existing-paths F3 and is merged into F7. bound-breaker F9 is merged into F2. bound-breaker F6 is merged into F9.
