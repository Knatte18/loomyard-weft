## Goal

Code-Review lets result-level issues through to PR-Review: widened test limits with no measured cause, departures kept on obsolete measurements, set-up differences described as agreement, and claims of improvement with no before and after figures.
Close the gap in three places, as the board entry (issue #139) asks:

1. The code reviewer's brief checks results, generically, in every recipe that runs Code-Review.
2. Drift flags every assertion, tolerance or limit the change widens or removes in an existing test, and requires a stated measured cause for each.
3. When PR-Review findings are more than small, by count or by a severity, the run goes back to Code-Review instead of the PR-Review agent fixing them, and the miss is recorded.

## Scope

In: the three ideas above, in the tapestry, darn and sampler recipes, the ones with a Code-Review segment.

Out:

- Domain rules, such as "a kept departure from a reference model is measured on the current code".
  They belong in each target repository's PATTERN, which the rubric already makes reviewers apply.
- Any change to how Drift compares captures, or to the expect agent.
- Recipes with no Code-Review segment: their PR-Review keeps fixing findings in place.

## Decisions

### D1 — Result checks in the Code-Review rubric

Add three bullets under `## Also flag` in `contracts/stencils/loom/loom-rubric-code-review.md`, worded for any language and any hub:

- **Loosened checks.** An assertion, tolerance or limit the diff widens or removes in a test that exists at the review base is a finding unless a `Loosened-Check:` trailer (D2) on a commit of the review range names it and a measured cause.
  A cause that is not a measurement, such as "flaky" or "to make it pass", is a finding too.
- **Tests still check what they say.** A test whose limits changed still checks the behaviour its name and doc comment state;
  one whose new limits admit the failure its name rules out is a finding.
- **Claims of agreement or improvement.** A claim, in the change description or a commit message of the range, that a result agrees with a reference or improves carries before and after figures from the current code.
  A claim without them, a figure the change itself made obsolete, and an agreement claim across a set-up difference the claim does not name are each a finding.

The Code-Bouncer and Code-Burler-Review rows of every recipe read this one rubric.

Rationale: one edit reaches every recipe and hub; the checks stay generic while domain rules stay in the target's PATTERN.

### D2 — The `Loosened-Check:` commit trailer

A loosened check's cause is a commit trailer, one per check:
`Loosened-Check: <test> — <what was loosened> — <measured cause>`.
It may sit on any commit of the review range, so a session that forgot it adds it in a later commit, with no change if need be, instead of rewriting history.
Every session that commits code in a recipe with a Code-Review segment is told so in its brief, through one shared stencil fragment each such brief includes.
Where Go writes a role's commit, the role's handoff carries the trailer lines and Go appends them.

Bound:

- A commit made after the last Drift-Recheck, the PR-Review agent's fix-here commits (D4) and Finalize's merge-fixer commits, is checked by neither D1 nor D3.
  That is accepted, since D4 caps PR-Review's in-place fixes at `pr_review_small_max` between send-backs wherever its send-back edge can fire, with the gaps D4's Bound names, and the merge fixer resolves the parent merge.
- A trailer's measured cause is the committer's attestation: neither D1 nor D3 re-measures it, and any session that commits, Code-Burler-Fix included, can write one.
  Its bound is that D3's regression check section lists each loosened check with its cause, so the pull request body shows every one, and so does the parent's review where `pr_review_ask_parent` asks for one.

Rationale: the commit message is the one record every writer makes before Drift-Verify and Code-Review, and the change description does not exist yet at Code-Review in tapestry.
The trailer follows the existing `Pattern-Departure:` trailer, which reviewers already read.

### D3 — Drift rules on loosened checks

Drift-Verify and Drift-Recheck give the judge, beside the changed cells or the failing cases, in both diff modes:

- each file that exists at the review base and that the committed code changes, with its diff against that base;
- the commit messages of the review range.

The review base is the one the Code-Review rubric's review range uses, the last merged parent tip, so changes a Parent-Align merge brought in are no subject.

The judge rules every listed file once:

- `none` when its diff widens or removes no assertion, tolerance or limit in a test;
- otherwise one entry per loosened check, naming the test and what was loosened, ruled `caused` when a `Loosened-Check:` trailer of the range names a measured cause for it, and `uncaused` otherwise.

Every ruling carries a non-empty reason.
Drift stores each file's ruling keyed by a hash of the file's diff together with the range's `Loosened-Check:` trailer lines, and reuses it while both are unchanged.
So Drift-Recheck spawns no judge for a file already ruled, and a commit that only adds a trailer has the file ruled again.
The judge spawns when a listed file has no stored ruling, even with no changed cell.

An `uncaused` ruling makes the row Stuck for the parent, like a `does not match` cell.
Its way forward is a commit carrying the trailer, or one reverting the loosening, then `lym loom resume`.
The regression check section `driftengine.Verdict` renders lists each loosened check with its ruling and cause, so the pull request body and PR-Review's `drift_verdict` carry them.

Bound:

- Only files existing at the review base are listed, so a loosened check in a test the same change adds is not ruled; D1 covers it.
- The judge decides what counts as a test and as loosened, so no language or path pattern is hard-coded.
  A refactor that rewrites an assertion with the same bound may be ruled loosened, and is cleared by a trailer.
- The judge spawns on every Drift-Verify whose change touches an existing file not yet ruled; the cost is one judge session per such check.
  A commit that adds or changes a trailer re-rules every listed file, at the cost of one more judge session.
- The writers' own drift gates, at Darn-Write and Webster, run the check without the judge, so an uncaused loosening is first seen at Drift-Verify.
- The rows run only where `drift.enabled` turns them on.

Rationale: a limit widened in the same change passes on both sides of the capture, so the cell compare cannot see it; the judge already rules changes against stated causes, and the diff is its missing input.

### D4 — PR-Review sends large findings back to Code-Review

A new verb, `lym loom pr-findings <file>`, run by the PR-Review agent, takes a findings file in burler's review format with its verdict line, parsed by `internal/burlerengine`'s verdict parser.
Every finding counts, whatever its severity.
It decides:

- **Send back** when a finding is BLOCKING, or the set plus the fix-here count below exceeds `loom.pr_review_small_max` (D5).
  The verb copies the file into the loom scratch directory, beside the Parent-Align fixer report, as `pr-findings-<n>.md`, where `<n>` counts the run's send-backs.
  A new Code-Review generation archives the `_lym/reviews/code/` run directory, and this path lies outside it.
  The verb writes the friction note (D6) and a send-back mark naming that file in the pull-request record, and answers the agent to end its turn.
- **Fix here** otherwise, and also when the recipe's PR-Review row declares no `on_reroute`, when the edge's target row's entry gate does not hold, or when the target perch approves without review, as Code-Bouncer does once a Code-Review budget of 0 stamps `zero_rounds` on it.
  Each case is decided from the built rows and the run's settings, never the recipe name.
  The verb adds the set's findings to a fix-here count in the pull-request record and answers that the agent fixes them itself, as today.
  Only the row's entry after a persisted send-back reroute clears that count, with the mark, `parent_review_asked` and the session id.
  A re-step, a crash-resume, the resume after the dirty-tree Stuck and the resume after a reroute-cap halt keep it.

The verb refuses while the worktree holds uncommitted changes, naming commit or discard as the way forward.
It refuses a file the verdict parser refuses, naming the format contract as the way forward.

The agent can still change the tree after the verb answers, so the PR-Review row's wake that meets the send-back mark stops the session first and re-reads the tree, as it does for the continue flag.
On a clean tree it returns a Stuck verdict with `Reroute` set and the mark's file as its output, and clears nothing;
shed then persists the reroute history entry, whose output is that file.
On a dirty tree the agent, told to end its turn, is idle, so the row keeps the mark and halts Stuck for the parent, naming commit or discard, then `lym loom resume`, as the way forward;
the resumed row meets the mark on a clean tree and reroutes.

The row clears at the start of its `Call`, before it starts or resumes a session.
A producer's `Call` receives no shed history, so the landing wiring injects a reader of the run's shed status into the row.
When the record holds a mark and the row's newest history entry is a reroute whose output is the mark's file, that reroute persisted:
the row clears the mark, the fix-here count, the record's `parent_review_asked` flag and its session id, then runs as on any entry.
A mark met with any other newest entry keeps all four, and the row handles it as its wake does, rerouting on a clean tree.
That covers a re-step, a crash before shed persisted the reroute, the resume after the dirty-tree Stuck and the resume after a reroute-cap halt, whose entry is a Stuck one.
Matching the file, not only the outcome, keeps a crash after a later send-back from taking an earlier send-back's reroute for its own.

The tapestry, darn and sampler recipes give PR-Review `on_reroute: Code-Bouncer`.
The re-entered Code-Bouncer takes the send-back file through a mechanism new beside today's Parent-Align amendment:

- a send-back file written after the settle forces a new full generation, never an amendment, even on an unchanged subject, so the round reviews the whole review range;
- the `ReviewInput` seam returns every input written after the settle, the newest send-back file and the Parent-Align fixer report alike, and the generation record keeps each path;
- the reviewer's round prompt names each input of a full generation in a review-input note of its own, outside the amendment note;
  an amendment's note keeps naming the Parent-Align fixer report, as today.

After that episode the run goes down its usual rows: Drift-Recheck, Describe, then PR-Review again, which finds the open pull request.
With the session id cleared it starts a new review session, and with `parent_review_asked` cleared and `pr_review_ask_parent` on it asks the parent again.

The PR-Review brief, `contracts/stencils/landing/landing-template-pr-review.md`, tells the agent to run the verb on every set of findings it receives before fixing any of them: the parent's review, and a list of findings the operator gives.
A single change the operator asks for is no findings set and goes straight to a fix.

Bound:

- The verb sends a run back only from a PR-Review row that declares the edge, decided by the recipe's rows, never its name.
- Shed's reroute cap bounds the edge: past it the run halts for the parent with the cap's way forward.
- Where the edge can fire, the fix-here count bounds successive sets each under the threshold:
  at most `pr_review_small_max` findings are fixed in place between two send-backs, or before the first.
  A send-back the reroute cap halts keeps the mark and the count,
  so the resume the cap's way forward names reroutes that send-back within the reroutes the resume grants.
  A parent's `lym loom goto` back to Code-Review keeps the count,
  so the first set after it may send back once more, within the reroute cap.
  A goto keeps a pending mark too, since it writes no reroute entry,
  so the row it leads back to sends that file back once more, within the reroute cap;
  `lym loom resume` is the way forward a marked halt names.
  In the fix-here fallback cases nothing caps the in-place fixes, as in a recipe with no Code-Review segment.
- The agent can skip the verb, or transcribe fewer findings or lower severities than the parent's answer holds, since the row never reads that answer.
  This is an accepted risk.
  Its only limits are the brief's rule and that every fix lands as a commit on the open pull request, where a review before merge can see it;
  the row waits for no such review, and the parent is asked only with `pr_review_ask_parent` on, once until a send-back clears the ask.

Rationale: the reroute edge exists, spends no bounce budget, gives the target segment a fresh episode and is capped.
New are the mark, the verb, the stored file, the row's shed status reader, the full generation forced by a review input, the seam's list of inputs and the review-input note.

### D5 — The `loom.pr_review_small_max` setting

A new loom config key, `pr_review_small_max`, an integer, default 3, loaded like the other loom keys.
It is the most findings PR-Review fixes in place between two send-backs, or before the first, when none is BLOCKING.
A task may set it in its `run_settings`.

Rationale: what "small" means differs by hub; a count the operator can tune is the simplest bound the board entry names.

### D6 — The miss is recorded

On a send-back the verb writes a friction note through `internal/friction` for the PR-Review row, at a path of its own per send-back, so a later send-back or the agent's own note never overwrites it.
The note says that Code-Review approved the diff, gives the number of findings by severity and each finding's title, and names the findings file D4 stores in the loom scratch directory.
The rubric's `## Review input` section, which today reads an input only when the amendment note names one, also reads each input the review-input note (D4) names.
It says how the reviewer reads a PR-findings input: review the whole range as usual, and file each PR finding that holds against the code.

Rationale: the friction note is the run record Friction-Reflect already reads, so the miss reaches the operator without a new record type; reviewing the whole range lets the reviewer find the misses' siblings.

### D7 — Figures in the change description

The Describe brief, `contracts/stencils/landing/landing-template-describe.md`, and the darn writer's description rule state that a claim of agreement with a reference or of an improvement carries before and after figures from the current code, taken from the commits or the drift verdict, and that a claim with none is left out.

Rationale: Describe replaces the writer's description after Code-Review, so without this rule a claim D1 kept out of the commits could come back in the pull request's body.

### Added after Discussion (arbiter, 2026-11-18): F13 is answered by D4's deferred clear

Decision: D4 as fixed in round 3 stands: on a clean tree the PR-Review row returns Stuck with Reroute set and the mark's pr-findings-<n>.md file as its output and clears nothing; at the start of its next Call the row clears the send-back mark, the fix-here count, parent_review_asked and the session id only when its newest own shed history entry is an OutcomeReroute entry whose Output is the mark's file. Every other entry (re-step, crash before the persist, dirty-tree resume, reroute-cap resume, goto) keeps all four. No further change to D4 is needed for F13.

Rationale: Round 3's F13 found the clear ran inside Call before shed persisted the reroute. The round-3 fix moves the clear after the persist. Checked against the code: ShedProducer.Call takes only a context (internal/shedengine/producer.go:90), so an injected status reader is needed as D4 says; rerouteStuck writes Output: output.Path on the OutcomeReroute entry (internal/shedengine/reroute.go:59) and on the RerouteCapped Stuck entry (reroute.go:70); a crash before the persist leaves current_producer in place so the row is re-called with no matching reroute entry (run.go:405); foldsInto never folds a reroute entry (run.go:744); rerouteCount stops at a RerouteCapped entry, so the cap-halt resume can reroute. Distinct pr-findings-<n> paths keep a later send-back's crash from matching an earlier reroute. The round hit the hard cap with the fix in place answering the finding, so the segment is accepted.

## Constraints

- `PATTERN-stencil-ownership`: D2's fragment is one stencil included by each brief, never restated.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: D3's diffs and commit messages come from `internal/fabricengine`, never from git run in `internal/driftengine`.
- `PATTERN-shed-recipe-registry`: the `on_reroute` edge is a recipe row value; the send-back falls back by the recipe's rows, never its name.
- `PATTERN-config-layers`: D5's key loads through `configengine`; a new key changes no existing key's meaning, so it ships no reinterpretation.
- `PATTERN-run-records`: D6's note is written only through `internal/friction`.
- `PATTERN-refusal-way-forward`: the verb's refusals and the uncaused Stuck name their ways forward, and a test reaches each.
- `PATTERN-tolerant-records`: the judgement, ruling and pull-request records keep unknown keys, so a record without D3's or D4's keys from an older run still reads.
- `PATTERN-code-conventions`: no language, test-path pattern or engine is hard-coded.

## Auto-mode assumptions

- None beyond D4's own rule; the support log's question ledger records each auto-pick.

## Open risks

- A peer run, `drift-judge-and-line-range`, changes the drift judge's snapshot check and line-range matching in `internal/driftengine` at the same time; D3 touches the judge request and judgement in the same package, so the later run to land meets a merge there.

## Acceptance criteria

- Review: the Code-Review rubric carries the three result-check bullets of D1, worded for any language and hub, and accepts a trailer on any commit of the range.
- Review: every brief of a session that commits code in a recipe with a Code-Review segment includes the shared fragment stating D2's trailer and its shape.
- D2: a commit Go writes for a role carries the `Loosened-Check:` trailer lines that role's handoff gives.
- D3: a Drift-Verify run over a change that widens a limit in a test existing at the review base, with no trailer in the range, spawns the judge even with no changed cell, and the `uncaused` ruling makes the row Stuck with ways forward naming a trailer commit or a revert and `lym loom resume`.
- D3: after that Stuck, a commit adding only a `Loosened-Check:` trailer for the check, then `lym loom resume`, spawns the judge again, which rules the check `caused`, and Drift-Verify finishes Done.
- D3: a test loosened by a parent commit a Parent-Align merge brought in is not listed to the judge.
- D3: a judgement that leaves a listed file unruled, or carries an empty reason, is refused back to the judge.
- D3: Drift-Recheck over files whose diffs and the range's trailers are unchanged reuses the stored rulings and spawns no judge for them.
- D3: a judgement record written before this change, without loosened rulings, still reads.
- D3: the regression check section lists each loosened check with its ruling and cause.
- D4: `lym loom pr-findings` on a file with one BLOCKING finding reroutes PR-Review to Code-Bouncer, and on a file with more findings than `pr_review_small_max`, none BLOCKING, does the same.
- D4: a set within `pr_review_small_max`, none BLOCKING, sends back when the fix-here count lifts the total past it.
- D4: the re-entered Code-Bouncer, on a subject unchanged since its settle, starts a full generation whose review input names the send-back file in the loom scratch directory, and its reviewers can read that file.
- D4: the PR-Review row re-entered after a send-back starts a new review session, since its entry after the persisted reroute cleared the recorded session id, and with `pr_review_ask_parent` on posts a new review request to the parent.
- D4: on a file with at most `pr_review_small_max` findings and none BLOCKING, on a recipe whose PR-Review declares no `on_reroute`, with the target row's entry gate off, or with a Code-Review budget of 0 that stamps `zero_rounds` on Code-Bouncer, the verb answers fix-here and the row does not reroute.
- D4: the verb refuses a dirty worktree, and a file the verdict parser refuses, each naming its way forward.
- D4: a send-back mark the row's wake meets on a tree dirty after the stop halts the row Stuck with the way forward commit or discard, then `lym loom resume`, and the resumed row on a clean tree reroutes.
- D4: a PR-Review row whose newest history entry is the reroute of its marked send-back starts with no mark and a fix-here count of zero.
  A re-step of the row keeps the count.
  A resume after a crash between the row's reroute verdict and shed's persist keeps the mark and the count and reroutes that send-back;
  so does the resume after a reroute-cap halt.
- D4: the reroute cap halts PR-Review's send-back once exceeded, as for any reroute edge.
- D5: `pr_review_small_max` loads with default 3 and a task's `run_settings` overrides it.
- D6: each send-back writes its own friction note naming Code-Review's miss, the findings by severity and the findings file, and a second send-back leaves the first note in place.
- Review: the rubric's `## Review input` section reads each input the review-input note names, and says to review the whole range and file each PR finding that holds (D6).
- Review: the PR-Review brief tells the agent to run `lym loom pr-findings` with a verdict line on every findings set before fixing any of it (D4).
- Review: the Describe brief and the darn writer's description rule carry D7's figures rule.
