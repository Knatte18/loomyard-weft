---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:163
    summary: "D7 step 1 rests on a false premise: lym beam pr open does not refuse a beam that only lags its parent (internal/beamcli/pr.go judgeOpenable passes it, refusing only an open merge task or a conflict), so running merge-parent on every lag adds a needless merge cycle per drain and, on a moving parent, an unbounded drain-merge-drain loop that never reaches step 4."
    fix: "In D7 step 1, replace the lag trigger with the conflict refusal: run lym beam merge-parent <slug> only when lym beam pr open refuses because the beam conflicts with its parent (judgeOpenable in internal/beamcli/pr.go), and state that a beam only behind its parent goes straight to step 2. Update D7's Review acceptance bullet to match."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:57
    summary: "D2 claims a run and the orch its notices reach agree on one value, but orchcli.ParentStrand routes through orchroute.FromPair to the nearest live orch at or above the parent worktree, else the prime's; with beam B opted out (false) under a hub default true and B's orch down, the prime orch reads its own true and settles B's halts with no operator, bypassing D1's opt-out, and the reverse case leaves a forced non-interactive run with an orch that waits for the operator."
    fix: "In D2, make the orch judge a reaching halt by the run's value, not its own: the halt notice (or the orch's lookup from the slug) carries the run's resolved autonomy, read by D2's run reader, and the orch applies that value. State the down-orch case as a bound: a halt reaching an ancestor or prime orch is handled under the run's value. Replace the Rationale's agreement claim with this and add an acceptance bullet: with the run's worktree false and its orch down, the prime orch under true relays to the operator; with it true, the ancestor orch settles."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:96
    summary: "D4 routes every PR-Review ask and re-ask through the parent-review seat as code-review asks do, but shedadapters.ParentReviewSeat is producer-driven over burler round files (identity from the round-1 focus file) and records no head, and the loom parent-review group holds only answer and await, so the session agent has no way to send ask n+1 after its fixes and pr-continue has no recorded head to check; the needed new verb or edge is neither named nor bounded."
    fix: "In D4, name the mechanism: who sends each re-ask (the session agent through a named new verb, such as lym loom parent-review ask, or the PR-Review row on seeing a new pushed head), where each ask's head and number are recorded (the pull request record), and the new verb's bound (refused outside a PR-Review session under autonomy, records the head it read). Mark any new verb (new) and add it to the Constraints' shellguard line."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:96
    summary: "D4 leaves the existing PR-Review parent ask in force: askParentReview in internal/landingshed/prreview.go still posts the [pr-review request] line telling the orch to answer by lym creel post <strand>, and orch-role.md's Reviewing a pull request section prescribes that answer and says the operator ends the review; an orch that answers that way records nothing, so D4's pr-continue refuses."
    fix: "In D4, state that under autonomy the row's [pr-review request] line names lym loom parent-review answer <slug> --ask <n> as the answer, and add orch-role.md's Reviewing a pull request section to D9's list of rules that name role-autonomy.md as their exception; add a Review acceptance bullet for both."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:98
    summary: "D4 treats pr-continue as the review's ending alone, but it also clears MailedHead so the row gates a head again after a gate failure from outside the change (gateRetryHint, internal/landingshed/prreview.go:716; internal/loomcli/prcontinue.go:39-43) and serves the closed-PR cleanup session; under D4's refusal that re-gate needs a satisfied parent answer at the current head, and one call both re-gates and ends the review."
    fix: "In D4, split the two uses: under autonomy pr-continue with a mailed head standing clears it and re-gates without the satisfied-answer check, ending the review only when an answer satisfies D4's condition; or name a separate re-gate path. State the closed-PR case. Add an acceptance bullet for the re-gate under autonomy."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:95
    summary: "D4 fixes the PR-Review session's Interactive when its spec is built (internal/landingshed/prsession.go sessionSpec) while pr-continue's autonomy check reads at call time; a toggle mid-review leaves an operator in an interactive pane whom pr-continue refuses, or a non-interactive session with no operator that ends the review with no answer from anyone."
    fix: "In D4, state that the autonomy value read when the session starts governs the review to its end: record it in the pull request record at session start, and have pr-continue read that recorded value rather than the live one. Add an acceptance bullet for a toggle while the session runs."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:74
    summary: "D3 forces judge_interactive false, removing the operator's seat that decides each ruling, and leans on judge_ask_parent; but past judge_parent_timeout_min a parent answer is refused and the judge's rulings stand as written, so a busy or down orch lets every non-stopping ruling pass with neither operator nor parent, and D3 states no bound for it."
    fix: "In D3, add a Bound paragraph stating what forcing judge_interactive false lets through: a matches ruling the parent does not answer by judge_parent_timeout_min stands, and a does-not-match still stops the run. Either accept that bound explicitly, or have autonomy also make an expired parent ask stop the run for the orch instead of letting the rulings stand."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:78
    summary: "D3's forcing on the module's resolved config misses raddle_auto_approve's reader, the run's setting lookup a profile's raddle block can set (internal/loomcli/raddlewiring.go, internal/loomshed/raddlewrite.go AutoApprove), and Raddle-Write records the mode in the run's state on its first call, so D3's restores each key's own value at its next load is false for a raddle run already started."
    fix: "In D3, state that the forcing applies at the run's setting lookup as well as the resolved module config, and that Raddle-Write's mode is fixed at its first call, so a change of autonomy reaches only a raddle run not yet at Raddle-Write. Adjust the restores-at-next-load sentence to name that exception."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:30
    summary: "Scope Out says making tasks stays the operator's under autonomy, but D3 forces raddle_auto_approve true, so a raddle files draft tasks with no operator approval (internal/loomshed/raddlefile.go: every filed task is draft), and D7 step 1 has the orch file a released merge task on its own go-ahead, against orch-role.md's rule that a step starting work needs the operator's explicit go-ahead."
    fix: "Reword Scope Out's bullet to what holds: releasing draft tasks, operator-filed tasks and promoting notes stay the operator's, while a raddle under autonomy files drafts (D3) and the orch files a beam's merge task (D7). Add to D7's Bound that merge-parent is the one work-starting step the drained line licenses, and name orch-role.md's go-ahead rule among D9's exceptions."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:129
    summary: "D5's class 5 counts only --by parent decision entries, but many settlements the orch role prescribes record none (lym loom resume on a spent bounce budget, lym batten run, goto plus resume, circling continue or accept), and lym loom decision replace folds added entries away, so a row can halt and be re-settled indefinitely and the Bound's two per row of a run does not hold."
    fix: "In D5, make every settlement of a halt under autonomy record one lym loom decision add <slug> --by parent entry naming the row, resume and circling verbs included, and count class 5 from those entries and the main decisions a replace folds them into (each folded entry keeps the row in its text). Or count from a record that every settling verb writes. State the chosen count source in the Bound and add an acceptance bullet."
  - id: F11
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:136
    summary: "The board entry's first case is a severe halt the orch cannot settle, but D5's Bound forbids relaying any halt outside the five classes as severe; a halt outside them that the orch has no action for (no verb settles it, and it is no repeat yet because the orch has made no settlement to count) has no route and stalls silently."
    fix: "In D5, add a class or a rule for a halt the orch finds no settling action for: it is relayed as a severe halt naming what it tried, or it counts toward class 5 at once. State it in the Bound beside the closed list, and update the Review acceptance bullet's class count."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:148
    summary: "D6 leaves where the drain marker lives to the implementer: the beam run's existing await marker is rewritten whole by writeMarker on clearDownNotice and recheckStanding and removed by landOrRefuse (internal/pawlengine/beamawait.go, landrequest.go), and pawl's markers sit in the untracked .lym/beam/ directory that Beam-Up clears and a takeover leaves behind, so a marker kept there posts a second [beam drained] line."
    fix: "In D6, name the store: a drain marker file of its own beside the pawl markers that no existing writer rewrites, with Beam-Await alone writing and removing it, and state that a takeover or new pawl run may post one further line; or a tracked record with its writer named. Add the dobby-down notice cycle to D6's no-second-line acceptance bullet."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:192
    summary: "D9 mails an orch only on a lym config beam --set write, but the resolved value also changes by the editor save of lym config beam, lym config reconcile, a hand-edited committed file and a default-layer change pulled from another machine; the orch keeps a stale reading while D2, D3 and D8 read at call time, and with D8 an orch that believes off writes unmarked replies the hook drops from Slack, severe-halt relays included."
    fix: "In D9, either have the orch read its autonomy at call time (a lym verb it runs on each halt, or the value carried on each notice per F2's fix) instead of trusting the launch line, or name every writer that mails it. Make D8's hook and the orch read one source, so a disagreement cannot drop replies. Add an acceptance bullet for a value change made outside lym config beam --set."
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:184
    summary: "D8's Bound names no backstop for a reply meant for the operator that lacks [operator] at its opening: a forgotten marker, or a relay after other text in the same turn, drops the whole reply from Slack while the operator is away, and the record is silent on the orch's direct creel posts to <shortname>:SLACK that the presence line asks for and that the filter does not govern."
    fix: "In D8, have the orch also post each severe-halt relay and merge-ready notice to <shortname>:SLACK with lym creel post, as the presence line already directs, so the hook's filter governs only turn replies; state that a relay opens its own reply, one relay per turn. Add a Review acceptance bullet for role-autonomy.md carrying both rules."
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:66
    summary: "D2's Rationale says a broken beam.yaml already stops the worktree's dobby, which the orch hears of, but internal/dobbyengine/loop.go posts the stop to the orch only after a re-exec and otherwise only logs it."
    fix: "In D2's Rationale, drop the clause about the orch hearing of it, keeping that a failed load reads false so the operator is asked."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:190
    summary: "D9 puts the autonomy line on every orch pointer on the path that appends the presence line, but WithPresence wraps only the start, resume and reload pointers; the handoff and soft-handoff bodies and the compact focus are rendered without it."
    fix: "In D9, name the pointers that carry the line (start, resume, reload and the pending resume the watcher types after a handoff) and drop the claim that the presence path reaches every pointer, or extend the line to the handoff bodies and compact focus explicitly."
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:195
    summary: "D7 has the orch run lym beam pr open and lym beam describe on a drained line, but those verbs' own help in internal/beamcli/cli.go says the orch runs them when the operator says so, and D9 names only orch-role.md and role-slack.md as taking the autonomy exception."
    fix: "In D9, add the lym beam pr and describe help texts in internal/beamcli/cli.go to the texts that name the autonomy exception, and add them to D9's Review acceptance bullet."
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:248
    summary: "D9's acceptance bullet covers a write that changes one worktree's value, not D9's own case of a --layer default write that changes several worktrees' values and mails each of their live orchs."
    fix: "Extend D9's acceptance bullet: a --layer default write mails every live orch whose resolved value changed and none whose worktree layer overrides it."
  - id: F19
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:132
    summary: "D5 says the orch files the bug a repeating stop shows, without naming the channel; PATTERN-github-auth lets only lym selfreport create file an issue, and D5 class 2 says only the operator makes a task."
    fix: "In D5, name the channel: the orch files the bug with lym selfreport create, or as a board note, never as a task."
  - id: F20
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:42
    summary: "D1 states that a worktree's beam.yaml is inherited by a worktree forked from it, as every per-worktree file is, without the observed result or its pin, though D1's nested-beam opt-out rests on it."
    fix: "In D1, pin the inheritance: the per-worktree file is tracked under the worktree's _lym/config/ on its code branch, so a branch forked from it carries the file; cite the test or code that shows it, such as Finalize's configChangeNotice comparing the file's fork-point and tip values."
  - id: F21
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:222
    summary: "The Auto-mode assumption that draft members keep a beam open restates D6's drafts included and Q10."
    fix: "Cut the bullet from Auto-mode assumptions and write None, since D6 states the rule."
---

### [BLOCKING] D7 step 1 runs merge-parent on a lag that `lym beam pr open` accepts

**Location:** `_lym/discussion/decision-record.md:163`

**Issue:** `judgeOpenable` (`internal/beamcli/pr.go:236-258`) says "A beam only behind its parent passes, since GitHub merges it". It refuses only an open merge task or a conflict.
Step 1 instead triggers on any lag.
The merge task is a released member (`internal/beamcli/mergeparent.go:23-24`), so its landing drains the beam again (D6), and a parent that moved in the meantime triggers step 1 again.
On a busy parent the loop is unbounded and never reaches the operator notice.
All four lenses found this.

### [BLOCKING] D2's run/orch agreement breaks when the parent worktree's orch is down

**Location:** `_lym/discussion/decision-record.md:57`

**Issue:** `orchroute.FromPair` (`internal/orchroute/walk.go:104-117`) walks up from the pair's parent to the nearest worktree whose orch is live, falling back to the prime.
D5 has the orch apply its own worktree's value to every halt that reaches it.
So a beam that opted out under a hub-wide `true` has its halts settled by the prime's orch, with no operator, whenever its own orch is down.
D1's opt-out is the operator's guard, and this bypasses it.
The reverse case strands a forced non-interactive run with an orch that waits for the operator.

### [MEDIUM] D4's parent-review seat path does not exist for PR-Review

**Location:** `_lym/discussion/decision-record.md:96`

**Issue:** `ParentReviewSeat` (`internal/shedadapters/parentseat.go`) is run by a review producer or Bouncer per burler round.
Its generation is the round-1 focus file, and it records no head.
The `parent-review` group (`internal/loomcli/parentreview.go:349,385`) holds `answer` and `await` only.
Nothing lets the session agent send ask n+1 after a fix, and nothing records the head that D4's `pr-continue` check reads.
This is a new verb or edge with no stated bound.

### [MEDIUM] D4 leaves the creel-answer PR-Review ask and the orch role's rule in force

**Location:** `_lym/discussion/decision-record.md:96`

**Issue:** `askParentReview` (`internal/landingshed/prreview.go:356-393`) still tells the orch to answer by `lym creel post <strand>`.
`contracts/orchrole/orch-role.md`'s "Reviewing a pull request" says the same and adds "the operator ends the review".
An answer sent that way records nothing, so D4's `pr-continue` refuses.
D9's exception list omits this section.

### [MEDIUM] D4's `pr-continue` refusal blocks the verb's re-gate use

**Location:** `_lym/discussion/decision-record.md:98`

**Issue:** `prContinueVerb` clears `MailedHead` (`internal/loomcli/prcontinue.go:39-43`).
`gateRetryHint` (`internal/landingshed/prreview.go:716`) tells the session to run it so the row re-gates a head after a gate failure from outside the change.
The closed-PR session also works under the continue flag (`prreview.go:317-325`).
D4 treats the verb as the ending alone.

### [MEDIUM] D4 is silent on autonomy changing while a PR-Review session runs

**Location:** `_lym/discussion/decision-record.md:95`

**Issue:** `sessionSpec` (`internal/landingshed/prsession.go:159-170`) sets `Interactive` once, when the spec is built.
D2 reads at call time, so `pr-continue` can see a different value than the session started with.

### [MEDIUM] D3 forces `judge_interactive` false with no bound on the parent timeout

**Location:** `_lym/discussion/decision-record.md:74`

**Issue:** under `judge_ask_parent`, the orch role says "a late answer is refused, and the judge's rulings stand as written".
`judge_parent_timeout_min` sets that deadline (`internal/driftengine/config.go:93-94`).
With the operator's seat forced off, an unanswered ask lets the rulings stand unreviewed.
PATTERN-config-layers counts a timeout whose expiry lets a gate pass as hollowing it, and the rubric's attack-surface rule needs the bound stated.

### [MEDIUM] D3's forcing misses `raddle_auto_approve`'s reader and the raddle mode stored at its first call

**Location:** `_lym/discussion/decision-record.md:78`

**Issue:** `raddle_auto_approve` is read through the run's setting lookup (`internal/loomshed/raddlewrite.go:33-35`), not the resolved loom config alone.
Raddle-Write records the mode on its first call and reads the state after that (`raddlewrite.go:64-67`).
So D3's claim that turning autonomy off restores each key at its next load is false for a raddle run already under way.

### [MEDIUM] Scope Out keeps task-making with the operator, which D3 and D7 contradict

**Location:** `_lym/discussion/decision-record.md:30`

**Issue:** Scope Out: "Releasing draft tasks, making tasks or promoting notes: these stay the operator's under autonomy."
D3's forced `raddle_auto_approve` files draft tasks with no approval.
D7 step 1 files a released merge task.
`contracts/orchrole/orch-role.md:423` lets a step that starts work run only on the operator's explicit go-ahead, and D9 does not name that rule among its exceptions.

### [MEDIUM] D5's class 5 does not count settlements that record no decision

**Location:** `_lym/discussion/decision-record.md:129`

**Issue:** the role's settlements include `lym loom resume` on a spent budget, `lym batten run`, goto plus resume, and `circling continue` or `accept` (`contracts/orchrole/orch-role.md:55-56`, `:160-169`).
None of them writes a `--by parent` entry.
A `lym loom decision replace` also folds added entries into the main decisions (`loom-discussion-spec.md:150-152`).
The Bound's "two per row of a run" therefore does not hold.

### [MEDIUM] A halt the orch cannot settle has no route under D5's closed list

**Location:** `_lym/discussion/decision-record.md:136`

**Issue:** the board entry's first operator case is "A severe halt the orch cannot settle".
D5 replaces that with five closed classes and forbids relaying any other halt as severe.
Take a halt outside the classes for which the orch finds no settling action.
It never reaches class 5, because no settlement is recorded to count.
It is never relayed, so it stalls without a word to the operator.

### [MEDIUM] D6 leaves the drain marker's store unnamed, and the existing marker writers would wipe it

**Location:** `_lym/discussion/decision-record.md:148`

**Issue:** the await marker is rewritten whole by `writeMarker` on `clearDownNotice` (`internal/pawlengine/beamawait.go:268-276`) and on `recheckStanding` (`:299-303`).
`landOrRefuse` removes it (`:322-326`).
Focus item 4 asks how the marker is stored, and the record defers that to the implementer.
Kept in the existing file, the marker is lost after one dobby-down cycle, and a second line posts.

### [MEDIUM] D9's change mail misses every writer other than `lym config beam --set`

**Location:** `_lym/discussion/decision-record.md:192`

**Issue:** the other writers are the editor save (`internal/configcli/doc.go:50-60`), a reconcile, a committed hand edit, and a default-layer change pulled from origin.
D8's hook reads at call time (`internal/slackcli/hook.go:162-197`).
An orch whose belief is stale at `false` drops every Slack reply, including the severe-halt relays that are the point of the mode.

### [MEDIUM] D8 has no backstop for an unmarked operator reply

**Location:** `_lym/discussion/decision-record.md:184`

**Issue:** `replyHook` forwards a reply whole or not at all.
A forgotten marker, or a relay after other text, loses the relay to Slack while the operator is away.
The presence line (`internal/orchengine/prompt.go:29`) and `contracts/orchrole/role-slack.md` already give a direct creel route to `<shortname>:SLACK`, and D8 does not use it.

### [LOW] D2's Rationale misstates what the orch hears of a broken `beam.yaml`

**Location:** `_lym/discussion/decision-record.md:66`

**Issue:** `internal/dobbyengine/loop.go:236-241` posts the stop only after a re-exec, and otherwise only logs it.
The fallback to `false` is safe whatever the orch hears.

### [LOW] D9's "every orch pointer" claim overreaches the presence path

**Location:** `_lym/discussion/decision-record.md:190`

**Issue:** `WithPresence`'s callers (`internal/orchengine/watcher.go`, `internal/orchcli/start.go`, `internal/orchengine/resumecontext.go`) wrap start, resume and reload pointers only.

### [LOW] The `lym beam pr` verbs' help contradicts D7 under autonomy

**Location:** `_lym/discussion/decision-record.md:195`

**Issue:** `prOpenCmd` and its siblings' Long text in `internal/beamcli/cli.go` says the orch runs them when the operator says so.
D9 does not name that help text among the texts that take the autonomy exception.

### [LOW] D9's acceptance bullet misses the default-layer write

**Location:** `_lym/discussion/decision-record.md:248`

**Issue:** D9's own case of one write that changes several worktrees has no criterion.

### [LOW] D5's "files the bug" names no channel

**Location:** `_lym/discussion/decision-record.md:132`

**Issue:** under PATTERN-github-auth, only `lym selfreport create` files an issue, and D5's own class 2 keeps task-making with the operator.

### [LOW] D1's fork inheritance is asserted, not pinned

**Location:** `_lym/discussion/decision-record.md:42`

**Issue:** focus item 1 asks for the observed result and its pin.
The support log credits an explore fork with it, but the record gives no evidence.
D1's nested-beam opt-out depends on it.

### [NIT] The Auto-mode assumption repeats D6

**Location:** `_lym/discussion/decision-record.md:222`

**Issue:** D6's "drafts included" and Q10 already state that draft members keep a beam open.

## Focus settlements

- Focus 1: the record pins D2's routing to `orchcli.ParentStrand` and D6's wakes and `Notify` to `internal/pawlengine/beamawait.go`, but the routing claim is false (F2).
  D1's inheritance is only asserted (F20).
  The Guard refusal holds (`internal/configengine/layers.go:166-168`).
  D2's strict-load claim is partly false (F15).
- Focus 2: D3 states no bound for `judge_interactive` (F7).
  For `raddle_auto_approve`, the Rationale bounds release, but Scope Out contradicts it (F9).
  D4's bound covers an answer at a stale head, since `pr-continue` refuses an ask not sent at the current head.
  It still has no recorded head to check against (F3), and it blocks the re-gate use (F5).
- Focus 3: a halt that fits no class has no route (F11), and the class 5 count leaks (F10).
  A reply the orch forgets to mark is lost to Slack, with no backstop (F14, F13).
- Focus 4: the marker's store is left to the implementer (F12).
  The merge task starts released, which holds (`internal/beamcli/mergeparent.go:23-24`), but step 1's trigger is false (F1).
- Focus 5: no Decision edits this task's own board entry, and none files or promotes a task as a run step.
  Open risks reads None, which agrees with the Decisions.
  The Auto-mode assumption is a duplicate (F21).

## Rejected

- Bound-breaker finding 12 (Beam-Await's `Deps` lacks seams for member tasks, child pairs and commits beyond the parent): seam and interface enumeration is Plan's work under the writer's exploration fence, and D6 states the conditions.
- The interactions lens's LOW on the mid-reply marker (its F6) and the existing-paths lens's LOW on the direct SLACK posts (its 9): merged into F14 at MEDIUM rather than kept apart. They are one defect: no backstop for an operator reply the hook drops.
