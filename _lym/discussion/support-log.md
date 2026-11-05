# Support log — pawl-orch-landing

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Four explore subagents read the pawl recipe and Beam-Await, Finalize and the pull-request code, the `lym beam` verbs, dobby and the push lock, and how shed treats a status naming a removed row.
Findings that shaped the design:

- Beam-Await does not read the ready record; the dobby turns it into a land request once nothing works under the beam, and Beam-Await accepts it, writes the Parent-Align status as the hold and re-checks.
- Beam-Describe is pawl-only; Parent-Align and PR-Review are shared with tapestry and darn.
- The pull-request record PR-Review and Finalize use sits in the run's scratch directory; `openAtOnce` edits the title and body of a pull request it finds.
- Shed refuses a status naming a row the recipe lacks and has no row-migration mechanism; `hubreconcile` never visits runs.
- The push lock is a flock; Go opens files close-on-exec, so a dead holder frees it.
- `lym beam start` refuses a held beam through `beamUpRefusal`.
- `internal/pullrequest`'s `Status` reads no merge state; GitHub's REST `mergeable_state` carries the values D6 needs.

### Turn 2 — design picks

Settled by auto-pick: the record's home (D4), the verb set (D6), the ready refusals (D3), Finalize's decision for a beam (D8), the text rule (D9), the in-flight move (D13), the start rule (D12), and the push-lock risk left open.

### Turn 3 — lens review

Six forks returned reports: interactions, existing-paths, bound-breaker, walkthrough, board-scope, leanness. Findings, merged across lenses:

- Finalize halts named `lym beam pr open`, which refuses at Finalize (interactions F1, walkthrough 1): fixed; Finalize opens or finds the pull request itself when one is required, its halts name `lym beam pr status` and `lym loom resume`, and `status` answers at Finalize (D6, D8).
- A member filed after ready runs and lands (bound-breaker 2, board-scope 1, interactions F2): fixed; the dobby starts no member once the ready record stands and names the cancel (D2), and the beam verbs refuse once ready (D5, D6).
- A nested beam inherits the pull-request record and the description (existing-paths 1 and 2, bound-breaker 1): fixed; the landing record names its beam and stamps the description, read as `ReadyFor` reads (D3, D4).
- Finalize wrote the tracked record without a commit (existing-paths 4): fixed; Finalize reads D4 through a seam and drops the pull request committed, keeping its scratch record for what it opens itself (D4, D8).
- Record removed before the parent-side landing (bound-breaker 7): fixed; ordering stated (D4, D8).
- Recorded pull request squash depended on `atOnce` (walkthrough 2): fixed; a recorded pull request always squashes (D8).
- D7 had no head field to announce from (interactions F3, existing-paths 5): fixed; D4 carries the announce head, set by `open` and by D13's carry (D4, D6, D13).
- D11's policy alone may not resume a pawl run, and the repair cap (existing-paths 3, walkthrough 5): fixed; the arbiter's eligibility extends to pawl and a new loop exit is a new stop (D11).
- Crash after the accept's removals left the dobby down (interactions F4): fixed; the entry starts the dobby when the marker stands (D11).
- Old-binary crash leaves a Parent-Align board hold (interactions F5): fixed; the entry check reads retired names too (D11).
- D13's halt may be taken by the arbiter (walkthrough 3): fixed; judged first as the parent's (D13).
- D13 left the dobby-stop marker and the scratch record (walkthrough 4, existing-paths 5): fixed (D13).
- A live old-binary driver never moves (bound-breaker 6): fixed as a stated bound (D13).
- Two resumes after a move (bound-breaker 9): fixed; the notice says so (D13).
- Halted run at Beam-Await ambiguous (walkthrough 6, bound-breaker 10): fixed; running or halted counts (D3, D5, D6).
- Stale pull-request head at ready (bound-breaker 3): fixed; refusal 6 compares heads and names `pr update`, and `behind` is named (D3, D6).
- Closed or merged recorded pull request (bound-breaker 4, interactions F6): fixed; `open` reopens, `close` skips GitHub for closed or merged, ready passes merged, Finalize's closed halt names the operator on GitHub (D3, D6, D8).
- `lym loom goto` back to Beam-Await (bound-breaker 5): fixed as D10's stated bound.
- Parent with no upstream (bound-breaker 8): fixed (D3).
- Pending checks after Finalize's merge-in (existing-paths 7): fixed; the halt names the check and resume (D8).
- D9 widened to every recipe (board-scope 3, leanness 7): fixed; D9 is pawl-only and its open risk dropped.
- D7 and `notify_on_land` double notice (existing-paths 8): fixed as D7's bound.
- `refuse` restarting the dobby right after the hold lifts (existing-paths 6): rejected; the hold is gone then, so it is correct, and D12's bound now names `lym beam start` only.
- Push-lock goal point (board-scope 2): kept in Open risks as a question for the parent; the duplicated fact is cut there (leanness 3).
- Dropped `merge` verb (board-scope 4): rejected; no finding, the entry says the orch never merges.
- Restating PATTERN constraints, assumptions, D1's doc list, D2's duplicate clause, thin rationales (leanness 1, 2, 4, 5, 6): fixed by cutting.
- Thin Turn 2 narrative (leanness 9): rejected; the ledger carries each pick.

## Rejected alternatives

- Keep the beam pull-request record in the run's scratch directory: lost on adopt or a fresh machine, while Finalize and the dobby act on it.
- A `lym beam pr merge` verb: the entry says the orch never merges; Finalize is the only merger.
- Let `lym beam describe` publish to the pull request itself: separate verbs keep a describe run from touching GitHub and let the orch review the file first.
- Notify member landings by setting `notify_on_land` on each member: the entry rules it out, and members filed later would miss it.
- Migrate in-flight runs in `hubreconcile`'s walk: it never visits runs, and a lazy move in shed's step covers every path that steps the run.
- Break a push lock whose holder record names a dead pid: the flock is already free then; breaking a live holder's lock is unsafe.
- Squash commit text from the pull request's current GitHub text: needs a further GitHub read; the orch keeps the pull request in step with the description through `lym beam pr update`.
- Finalize never editing a found pull request in every recipe: task landings change with no ask on the board; kept to pawl.
- Finalize writing the tracked landing record through its scratch record path: those writes carry no commit; Finalize reads the tracked record through a seam instead.
- Halts naming `lym beam pr open` at Finalize: the verb refuses past Beam-Await; Finalize opens the pull request itself instead.
- A board refusal of new members under a ready beam: the board cannot read the tracked ready record; the dobby's start hold and the member fork's precondition cover it.
- The dobby writing the land request again after a crash in the accept: its land episode stays recorded across restarts, so Beam-Await's entry re-checks from the ready record itself (D11).
- Finalize halting on a recorded pull request merged at another head: the beam verbs refuse once ready, so no way forward works there; Finalize lands the rest as with none recorded instead (D8).
- Carrying a scratch-recorded pull request into the landing record at Finalize: the scratch record does not name its writer, so the carry would also fire on a pull request a halted Finalize opened; Finalize reads the scratch record instead (D8).

## Review rounds

### Round 1

Fixed: the crash and goto recovery rests on Beam-Await's entry reading the ready record, not on the dobby (D10, D11); a hand member run under a ready beam is refused at the fork (D2); the repair-cap reset has a ten-minute bound (D11); a pull request whose merge lacks beam commits is refused at ready, and Finalize lands the rest of one merged at another head (D3, D8); Finalize's GitHub refusals halt naming GitHub's message (D8); a run at Finalize lands through a pull request in its scratch record (D8); D13's tracked changes commit and remove the reopen request; `close` on a `merge` parent, describe owning `summary.md`, the arbiter's rule, the paused hand-back, both retry windows, and cuts (D2, D4, D5, D6, D13, assumptions).
Open for the parent: the push-lock point (Open risks).

## Question ledger

- Q1 Where does the open pull request record live? Auto-pick: tracked, beside the ready record (D4). Resolved.
- Q2 Which `lym beam pr` verbs? Auto-pick: open, update, status, close; no merge (D6). Resolved.
- Q3 Does `lym beam ready` refuse more than the two causes the entry lists? Auto-pick: yes, run row, open work, missing description and a required pull request with no record (D3). Resolved.
- Q4 How does a beam's Finalize decide pull request or not? Auto-pick: the record when it stands; otherwise beam.yaml `merge` and required rules as today, and `on` or `off` lands directly (D8). Resolved.
- Q5 Does D9 apply to every recipe? Auto-pick: no, pawl only, after the lens review (D9). Resolved.
- Q10 How is a ready beam held before the status hold? Auto-pick: the dobby starts no member once the ready record stands (D2). Resolved.
- Q11 How does a nested beam avoid its parent beam's records? Auto-pick: the landing record names its beam and stamps the description (D4). Resolved.
- Q6 How is an in-flight run moved? Auto-pick: a recipe-declared retired-row map applied by shed's step (D13). Resolved.
- Q7 Does `lym beam start` on a held beam start the dobby? Auto-pick: the orch only (D12). Resolved.
- Q8 Issue #47's stale push lock. Open: the lock already frees on holder death; asked of the parent through Open risks.
- Q9 Squash commit text source. Auto-pick: the description (D9). Resolved.
