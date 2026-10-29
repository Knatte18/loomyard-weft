---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D10
    summary: D10 keeps a wait marker of a kind the quiet check already reads, but none of the four kinds readAgentActivity counts fits PR-Review, so the record defers which marker exists to Plan and the quiet notice the operator ruled out goes off
    fix: In D10, name the mechanism and its lifetime, settled from the code. The four live-wait kinds in `internal/battencli/wire.go` readAgentActivity are a non-paused verify marker (verifytree, live only during a gate), a parent-lock wait record (contradicts D9's no lock), a child status running at pawl's Beam-Await row (matched by row name in childAtBeamAwait) and a non-held shuttle wait marker (written only inside a shuttle Run's Wait, which D7's un-awaited session has not). State the chosen one, for example that readAgentActivity also reads a child status running at the PR-Review row, beside childAtBeamAwait, live for as long as the status reads running at PR-Review, and drop the words of a kind battenshed's quiet check already reads. Update the support log's Turn 4 and Q5 to match, and the D10 acceptance criterion to name the mechanism.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 Bound, D9 first merged bullet
    summary: D8's bound says D9's Done reads only the record's gated head, but D9's merged ending compares the merged head with the task head, so an ungated agent commit carried by the detached records push and merged on GitHub ends Done and lands through Finalize's already-landed path ungated
    fix: Settle the merged-on-GitHub case and make D8 and D9 agree. Either make D9's merged Done require the merged pull request's head to equal the record's gated head and add a merged-with-an-ungated-head halt naming the head and the way forward, or rewrite D8's bound to say truthfully that a merge on GitHub lands whatever head the pull request holds, gated or not, and what bounds it. Add a D8 or D9 acceptance criterion for an ungated head merged on GitHub.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7
    summary: The session's limit never pushes, never approves, merges or closes the pull request and never calls GitHub is only a stencil instruction; the session runs in the task worktree with git push and gh available, and the Decision states no enforced bound or the residual it lets through
    fix: In D7, state what bounds the session. Either deny the escape on the launch spec, for example settings denying `git push` and `gh` through the shell-guard table, and say so as the bound, or state the residual plainly as a bound, that the session can push or merge on GitHub and what then catches it (F2's gated-head check). Add a D7 acceptance criterion for the chosen bound.
  - id: F4
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks, first bullet
    summary: With pr_account empty or the operator's own account, GitHub refuses the operator's approval, so the board entry's first ending, the operator approves on GitHub, cannot happen; whether lym must require another pr_account, accept a merge on GitHub as the operator's ending, or offer a lym-side approval is a question only the parent can settle
    fix: Raise to the run's parent the question the Open risks bullet asks, how the operator approves a pull request opened under their own account, with the three options it names. Record the answer through lym loom decision add, then drop the Open risks bullet and the open Q13 in the support log.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D11, fourth bullet
    summary: D11 replaces the local parent-side Fabric.Merge with a GitHub merge plus a fetch and fast-forward of the local parent pair, but does not say what replaces the records-side work Fabric.Merge does under the weft write lock, its merge record and conclude commit with the warp correspondence
    fix: In D11's fourth bullet, after the GitHub merge and the fetch, route the landing through Finalize's existing already-landed path, the parent-side Fabric.Merge that finds the code side already holding the task's work, instead of a bare fast-forward of the pair, so the records side and its correspondence are handled as for a pull request merged on GitHub by someone else. Name the path, not its mechanism.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D7, D6
    summary: D7 finds the session live, stops it through shuttle's stop verb and resumes it by its session id, but Runner.Open writes no run record and returns the strand GUID and session id only to its caller, the D6 record holds neither, and no shuttle verb resumes an opened session
    fix: In D6, add the session's strand GUID, session id and engine to the pull-request record's fields. In D7, state that shuttle gains, beside the start that carries a prompt, a resume of an opened session by its session id, and that the row reads both identifiers from the D6 record. Extend the D7 acceptance criterion to a re-entry after a crash resuming the recorded session.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D9 merged bullets
    summary: The merged bullets read the head and tree with the session still live, unlike the approval bullet, and no bullet covers merged with the task head equal to the merged head and a dirty tree, so a late agent commit can land outside the pull request and a dirty tree leaves the loop with no rule
    fix: In D9, give the merged state the approval bullet's sequence. On merged, stop the session, then re-read the head and tree. A merged head equal to the task head on a clean tree is Done. Task commits the merged head lacks are the existing halt. A dirty tree is a Stuck with a Halt naming the uncommitted files, with committing or discarding them in the task worktree, then `lym loom resume`, as the way forward. Add the dirty-tree case to the D9 acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D11, last bullet
    summary: The refused-merge halt names re-approval or settling a check as its one way forward, but a repository that disallows the configured squash or merge method refuses every merge, which neither step fixes, so resume halts again
    fix: In D11's last bullet, give the halt a way forward per refusal class. A disallowed merge method names setting landing.yaml's `squash` to the method the repository allows through `lym config landing`, or allowing it in the repository settings, then `lym loom resume`. Keep re-approval or settling the check for the moved-head, dismissed-approval and required-check refusals.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3, D11 first bullet, D12
    summary: The row's no closes an open pull request at once, but Finalize's no lands locally and leaves it to the teardown, which then closes it with a teardown comment, contradicting D12's a landed run's pull request is merged and left as it is
    fix: In D11's first bullet, have a no close a D6 record's open pull request with a comment naming the resolver's reason and remove the record, as D3's no does. In D12, keep a landed run's pull request merged and left as it is, true now on both paths.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7, D9 approval bullet
    summary: D7 says the wait loop starts no session, so an operator who closes the window ends the agent until the next entry, yet D9's approval bullet resumes the session from inside the wait loop whenever the approved head moved or the tree is dirty
    fix: In D9's approval bullet, resume the session only when the row itself stopped a live session for that read, and leave a session the operator closed ended. Alternatively, narrow D7's sentence to name the approval bullet's resume as the one exception.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4
    summary: In D4's halt way forward, the trailing then lym loom resume also attaches to the cancel-and-rerun branch, after which no run is left to resume
    fix: In D4, split the way forward into two alternatives. One gives the hub a GitHub origin and an upstream for the parent, then `lym loom resume`. The other, for a field `on` under a parent that is not `required`, runs `lym batten cancel <slug>` and starts a new run with the field changed.
  - id: F12
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D12
    summary: D12 claims every teardown caller closes the open pull request, but the teardown also runs for a task worktree already gone, where the D6 record under the run directory cannot be read
    fix: In D12, state the gone-worktree case. Either the teardown finds the open pull request by its head branch and the parent base through the D5 client when the record is unreadable, or it says the bound, that a teardown of a gone worktree leaves the pull request open and logs a Warn naming the task branch.
  - id: F13
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D9, second bullet
    summary: The merged-lacking-commits halt names filing them as a follow-up without saying the follow-up is a board note, which leaves room for halt text that tells an agent to create a task
    fix: In D9's second bullet, change filing them as a follow-up to filing them as a board note, and keep `lym batten cancel <slug>` as the rest of the way forward.
---

### [BLOCKING] D10 leaves the wait marker's kind for Plan to find

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** D10 says the row keeps a wait marker "of a kind battenshed's quiet check already reads" and names none.
`readAgentActivity` in `internal/battencli/wire.go` counts four live waits, and none fits a row that waits on GitHub.
A non-paused verify marker belongs to verifytree and is live only during a gate.
A parent-lock wait record contradicts D9's "takes no parent lock at any point".
`childAtBeamAwait` matches the pawl row `pawlengine.NameBeamAwait` by name.
A non-held shuttle `wait.yaml` is written only inside a shuttle Run's Wait (`internal/shuttleengine/waitmark.go`), and D7's session is started through an `Runner.Open`-like start that writes no run directory and is not awaited.
So the fact the operator's creel #20 decision rests on is a deferred research question, and as written the quiet notice goes off on every long wait.

### [BLOCKING] D9's merged ending lands a head D8 never gated

**Location:** `_lym/discussion/decision-record.md` D8 Bound, D9 first merged bullet

**Issue:** D8's bound promises that an ungated commit the detached records push carries to the pull request "lands nothing until the row has gated it", because D9's Done reads only the gated head.
D9's merged bullet compares the merged head with the task head instead.
If anyone merges that pull request on GitHub, the run is Done, and Finalize's already-landed path (`internal/landingshed/finalize.go`) lands the ungated head with no gate.

### [BLOCKING] D7's no-push, no-GitHub limit is only a prompt instruction

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** The session is told it never pushes and never calls GitHub, but D7 names no setting or guard that enforces it.
The session runs in the task worktree, where `git push` and `gh` work, and D5's rationale itself says the session's environment carries a token.
An agent that runs `gh pr merge` reaches the parent through F2's path, outside Finalize and its lock.
The rubric's attack-surface rule requires the Decision to state what bounds this new route, and it states none.

### [BLOCKING] The operator cannot give the approval the entry's first ending needs

**Location:** `_lym/discussion/decision-record.md` Open risks, first bullet

**Issue:** The board entry names three endings, the first being the operator approving on GitHub.
The record's own Constraint says GitHub refuses an approval from the pull request's author, so with `pr_account` empty or the operator's own account, that ending cannot happen.
The writer correctly put the question under Open risks.
Choosing among requiring another account, accepting a merge on GitHub, which skips Finalize's gate and trailer, and a lym-side approval changes what the task is for, so only the parent can settle it.
This finding raises it.

### [MEDIUM] D11's GitHub merge skips Fabric.Merge's records-side work

**Location:** `_lym/discussion/decision-record.md` D11, fourth bullet

**Issue:** D11 replaces the parent-side `Fabric.Merge` with a GitHub merge, then a fetch and fast-forward of the local parent pair.
`Fabric.Merge` runs under the weft write lock and writes a merge record and conclude commit carrying the warp correspondence (`internal/fabricengine/merge.go`, `internal/fabricengine/doc.go` "The merge surface").
D11 says nothing about what replaces that work.
Finalize's existing already-landed path already handles a pull request merged on GitHub by someone else, so routing through it is the smaller design.

### [MEDIUM] D7's session identifiers have no stated home

**Location:** `_lym/discussion/decision-record.md` D7, D6

**Issue:** `Runner.Open` writes no run record and returns `OpenResult` (strand GUID, session id) only to its caller (`internal/shuttleengine/open.go`).
The D6 record holds the number, URL, base and gated head only.
D7 still stops the session through shuttle's stop verb, which needs the GUID, and resumes it by its session id after a crash, a re-entry or D9's stop-then-resume.
No stated file holds either identifier, and D7 names no shuttle resume for an opened session.

### [MEDIUM] D9's merged bullets read with the session live and miss the dirty tree

**Location:** `_lym/discussion/decision-record.md` D9 merged bullets

**Issue:** The approval bullet stops the session before its last read, so a commit made after the approval stays out of the landing.
The merged bullets read with the session still live.
A commit made after that read and before the stop is a task commit the merge lacks, and Finalize's local merge lands it outside the pull request.
A merged pull request whose head equals the task head, on a dirty tree, matches none of the bullets.

### [MEDIUM] D11's refused-merge halt has no way forward for a disallowed merge method

**Location:** `_lym/discussion/decision-record.md` D11, last bullet

**Issue:** A repository that disallows the method `squash` selects refuses every merge.
The halt's way forward, re-approval or settling a check, does not reach a landing, which `PATTERN-refusal-way-forward` requires.

### [LOW] A no at Finalize leaves the pull request to a teardown comment

**Location:** `_lym/discussion/decision-record.md` D3, D11 first bullet, D12

**Issue:** D3's no closes the pull request with the resolver's reason.
D11's no lands locally and leaves it open, so the teardown closes it as if the run were abandoned.
That contradicts D12's "a landed run's pull request is merged and left as it is".

### [LOW] D9 resumes a session D7 says the wait loop never restarts

**Location:** `_lym/discussion/decision-record.md` D7, D9 approval bullet

**Issue:** D7 says the wait loop starts no session, so a window the operator closed stays closed.
D9's approval bullet resumes the session from inside that loop whenever the approved head moved or the tree is dirty.

### [LOW] D4's way forward ends the cancel branch in a resume

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** The trailing "then `lym loom resume`" also follows `lym batten cancel <slug>`, after which there is no run to resume.

### [LOW] D12 misses a teardown whose task worktree is gone

**Location:** `_lym/discussion/decision-record.md` D12

**Issue:** `internal/pairteardown` also tears down a pair whose task worktree is gone.
In that case the D6 record under the run directory cannot be read, so "every other teardown caller" does not hold.

### [LOW] D9's follow-up does not say it is a board note

**Location:** `_lym/discussion/decision-record.md` D9, second bullet

**Issue:** The halt's way forward says "filing them as a follow-up".
The rubric and the writer's stencil require a follow-up a decision files to be a board note, never a task.
The wording should name the note.

## Rejected

- bound-breaker F3 and walkthrough F0, existing-paths finding 1 in their stated form: they rest on the task's records side being merged into the parent on a local landing, but `internal/fabricengine/merge.go` says neither merge verb makes the weft a merge participant; the narrower, corrected defect is kept as F5.
- bound-breaker F4, an edit reverted mid-gate: the tree after the gate equals the gated head, and a lym merge still runs the `finalize` gate over the merged whole tree, so the residual is below the bar of a finding.
- Focus premises for D1 and D2 were checked and hold: per-worktree config lives on the records side and a landing does not carry it (`internal/landingshed/finalize.go` configChangeNotice), and the run lock refuses a held entry's change to any field other than status outside its named exceptions (`internal/boardengine/doc.go` "Run lock").
- No Decision edits this task's own board entry, and none files or promotes a task. D4's way forward changes the field only after the cancel releases the lock.
- The support log's Rejected alternatives and Question ledger hold no requirement the record lacks, and no Decision argues for a rejected option.
