---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:107,119,126,133
    summary: A changed Merge-Review merge-in starts a full generation without bumping the generation record, so the kept settle record stays live and every later Call of that generation re-archives and re-seeds it
    fix: In D6's Merge-Review sentence (line 133), state that case 2 for Merge-Review also records the next generation number in the generation record, with no amendment mark and no base, before its seed, so the settle record kept across the archive names an older generation and is no longer live. Add an acceptance criterion that a new merge-in after a settled Merge-Review reaches its round 1 judgment without a second archive.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:52-53,60,99,198
    summary: The settle record's settle time is the time the generation first settled, so the verify retry's round N+1 settle keeps a time earlier than the Publish failure record and D2's bypass never closes, contrary to D2 and the after-settle criterion
    fix: In D5 (line 99), give the settle record two times, or redefine the one time, so that a case 1 re-settle keeps the stored time and a settle on the round-by-round path (a round N+1 after the verify retry or the Webster reroute) writes its own settle time. In D2 (lines 52 and 60), name which of the record's times the failure record's write time is compared against.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:127,129,136,209
    summary: Excluding the generation_start seed Stuck from the count gives an amendment one more judged round than its allowance, so its soft-cap window opens at the hard cap, unlike any other generation and unlike D6's bound and its criterion
    fix: In D6 (line 129), make `episodeStuckCount` stop after counting the `generation_start` entry, so the amendment's seed consumes one unit as a normal generation's seed does, and say the count reads 1 once the seed is recorded rather than starting at 0. Restate D6's bound (line 136) and the criterion at line 209 to name the round numbers, so that with allowance A the soft-cap rule runs at round A and the hard cap escalates at round A+1. Alternatively keep the exclusion and move the hard cap to two rounds past the allowance, stating that the amendment gets A+1 judged rounds.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:99,104,130,205,207
    summary: The code-segment identity is named as the diff against the merge base with no stated form, so a Merge-In that changes a file the task also changed alters the hunk headers, context or post-image and reads as an amendment, contrary to the Merge-In criterion; the told base has the same gap
    fix: In D5's code-segment bullet (line 104), define the identity as a form that a parent-side change does not move, such as the stable patch-id of a zero-context diff of HEAD against the merge base, read through the told fabric seams, and say whether uncommitted changes count. Drop the `_lyx` restriction or restate it as the warp tree, since `_lyx` lives on the weft side and no warp diff holds it. In D6 (line 130), state what the code segment's amendment base is, so its reviewers diff the task's change since the settle without the parent's changes a Merge-In brought. Narrow the criterion at line 207 to the form chosen.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:128,135,141-142
    summary: D7 adds an off switch that approves every amendment unreviewed, a whole PR-Gate-reject replan included, but states neither its validation nor that it is not a guard key
    fix: In D7, state that `review_amend_max_bounces` joins `reviewbudget.CheckLocal`'s bounds with a minimum of 0 and no null, that it is not a guard key under reviewbudget's rule that a budget of 0 is the off switch, and that at 0 every amendment generation, a PR-Gate reject's rework and a rewrite after a goto to the writer included, settles with no review.
---

### [BLOCKING] Merge-Review's new generation leaves the old settle record live

**Location:** `_lyx/discussion/decision-record.md:107,119,126,133`

**Issue:** D5 calls a settle record live when it names the run directory's current generation and no round report past its settled round is on disk (line 107), and every archive keeps the settle record and the generation record in the new directory (line 119).
D6 records a new generation number only for an amendment: "in every segment but Merge-Review, starts an amendment generation, recording its number, its amendment mark and its base" (line 126).
For Merge-Review, case 2 "starts a full generation under the perch's own budget" (line 133) and nothing bumps the generation number.
So after the archive, the kept record still names the current generation, and the new directory holds no report past its settled round until the new generation outruns it.
At the second Call, after the round producer wrote round 1, the record is live, its identity (the old merge-in) differs from the live one, and case 2 archives and re-seeds again.
The generation never reaches a judgment.
This is the focus's first question for Merge-Review: the comparison archives a generation already under way.

### [MEDIUM] The verify-retry bypass never closes under D5's settle time

**Location:** `_lyx/discussion/decision-record.md:52-53,60,99,198`

**Issue:** D5 defines the record's settle time as "the time that generation first settled, which a re-settle in case 1 keeps" (line 99).
Loom's verify retry runs round N+1 of the same, already settled generation (line 53), so that generation's next settle still carries the first settle time, which is earlier than the Publish failure record's write time.
D2 then still admits the goto, though line 60 says the bypass holds "only until that generation settles again" and the criterion at line 198 says a goto onto the row is refused after that settle.
The record goes away only when a later Merge-In or Publish verify passes (`internal/landingshed/doc.go`: "Merge-In removes the record once its plan verify passes, and Publish removes it once its `publish_verify` passes"), so until then any goto onto `Webster-Burler-Review` re-runs a settled review.
This reopens round 1's bypass finding, whose fix keyed the bound on this time comparison.
Keeping the time across a case 1 re-settle is needed so a goto onto the Bouncer after a Publish failure does not close the bypass early; the defect is that the same rule also covers the round-by-round settle.

### [MEDIUM] Amendment budget is off by one against the perch's budget rule

**Location:** `_lyx/discussion/decision-record.md:127,129,136,209`

**Issue:** In a normal generation the seed Stuck consumes one unit: `NewBouncer`'s doc says "The seed call's unconditional Stuck permanently consumes one unit of that budget", and `budgetReached` compares the count over the history before this Call's Stuck (`internal/shedadapters/bouncer.go`).
So a budget of 1 opens the soft-cap window at round 1.
D6 has `episodeStuckCount` stop at the `generation_start` entry "and exclude it", starting the amendment at 0 (line 129).
With allowance 1, the count at round 1's judgment is 0, so round 1 is not at the soft cap; a CONTINUE spends the unit, and round 2 is the first round in the soft-cap window.
The hard cap is "one round past that allowance", round 2 (line 127), so the soft-cap window and the hard cap coincide.
D6's bound (line 136) and the criterion at line 209 expect the soft-cap rule at the allowance's round and the hard cap one round later, "as in any generation".

### [MEDIUM] The code identity's form is unstated, and a parent-side Merge-In can change it

**Location:** `_lyx/discussion/decision-record.md:99,104,130,205,207`

**Issue:** D5 names the code-segment identity as "the diff of the tree against its merge base with the parent branch" (line 104), and the criterion at line 207 says a Merge-In that changed only the parent's side leaves it unchanged.
A textual diff carries hunk line numbers and context lines, and a post-image hash carries the parent's lines in a file both sides touched, so a parent commit in a task-touched file changes either form and reads as case 2, an amendment review of an unchanged task change.
The decision does not say which form, nor whether "the tree" is HEAD or the working tree.
The `_lyx` restriction rests on a false premise: `git ls-files _lyx` in this worktree lists nothing, because `_lyx` is the weft side of the fabric, so no warp diff holds the status commits or review run directories the clause excludes.
D6 tells the amendment's reviewers "the base the settle record holds" (line 130); for a code segment after a Merge-In, a diff from that base also holds the parent's changes, and the decision does not say what the base is for a code segment.
On the focus's second question, the Discussion identity's exclusion of carry-over blocks and `Added after Discussion` entries holds, and the code identity holds only for a Merge-In that touches no file the task changed.

### [LOW] The amendment off switch has no stated bound

**Location:** `_lyx/discussion/decision-record.md:128,135,141-142`

**Issue:** D6 says an allowance of 0 settles an amendment with no review round, and D6's bound says an amendment of any size gets the allowance, so 0 lets a whole rewritten plan or a PR-Gate reject's rework through unreviewed.
D7 says only that 0 is the off switch and that the key is read through the config stack, `run_settings` included.
The existing policy that bounds such a value lives in `internal/reviewbudget/doc.go` ("No budget key is a guard: a budget of 0 is the off switch") and in `reviewbudget.CheckLocal`, which refuses each budget key below its minimum; D7 places the new key under neither, so the plan writer has no bound to implement or test.

## Focus verdicts

Each directive was reached; these are its verdicts beyond the findings above.

- Focus 1: an amendment generation's Call reaches the round path, since the bumped generation record makes the old record not live; a round N+1 after the verify retry or the Webster reroute reaches the round path, since its report sits past the settled round. Merge-Review is the exception (F1).
- Focus 3: after an interrupted settle, the amendment's count starts at its `generation_start` seed whatever the failed Commit left in history, subject to F3's off-by-one. The D3 driver rule reads a status stop's `error`, which is where Shed's blocked arm persists the `ReasonBounceBudgetExhausted` reason (`internal/shedengine/run.go`), so the hand-over rule cannot resume that halt.
- Focus 4: the settled-record refusal, the amendment hard cap and Merge-Review's exclusion from amendments state their bounds; the verify-retry bypass (F2) and the allowance of 0 (F5) do not.
