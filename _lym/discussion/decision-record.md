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

Bound: a commit made after the last Drift-Recheck, the PR-Review agent's fix-here commits (D4) and Finalize's merge-fixer commits, is checked by neither D1 nor D3.
That is accepted, since D4 keeps PR-Review's fixes small and the merge fixer resolves the parent merge.

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
Drift stores each file's ruling keyed by a hash of the file's diff, and reuses it while that diff is unchanged, so Drift-Recheck spawns no judge for a file already ruled.
The judge spawns when a listed file has no stored ruling, even with no changed cell.

An `uncaused` ruling makes the row Stuck for the parent, like a `does not match` cell.
Its way forward is a commit carrying the trailer, or one reverting the loosening, then `lym loom resume`.
The regression check section `driftengine.Verdict` renders lists each loosened check with its ruling and cause, so the pull request body and PR-Review's `drift_verdict` carry them.

Bound:

- Only files existing at the review base are listed, so a loosened check in a test the same change adds is not ruled; D1 covers it.
- The judge decides what counts as a test and as loosened, so no language or path pattern is hard-coded.
  A refactor that rewrites an assertion with the same bound may be ruled loosened, and is cleared by a trailer.
- The judge spawns on every Drift-Verify whose change touches an existing file not yet ruled; the cost is one judge session per such check.
- The writers' own drift gates, at Darn-Write and Webster, run the check without the judge, so an uncaused loosening is first seen at Drift-Verify.
- The rows run only where `drift.enabled` turns them on.

Rationale: a limit widened in the same change passes on both sides of the capture, so the cell compare cannot see it; the judge already rules changes against stated causes, and the diff is its missing input.

### D4 — PR-Review sends large findings back to Code-Review

A new verb, `lym loom pr-findings <file>`, run by the PR-Review agent, takes a findings file in burler's review format with its verdict line, parsed by `internal/burlerengine`'s verdict parser.
Every finding counts, whatever its severity.
It decides:

- **Send back** when any finding is BLOCKING or the findings number more than `loom.pr_review_small_max` (D5).
  The verb stores the file in the run's `_lym/reviews/code/` directory under a name of its own per send-back, writes the friction note (D6), and writes a send-back mark in the pull-request record.
  It answers the agent to end its turn.
  The PR-Review row's wait wakes on the mark, clears it, and returns a Stuck verdict with `Reroute` set; the row's existing stop ends the session.
- **Fix here** otherwise, and also when the recipe's PR-Review row declares no `on_reroute` or the edge's target row's entry gate does not hold: the verb answers that the agent fixes the findings itself, as today.

The verb refuses while the worktree holds uncommitted changes, naming commit or discard as the way forward, so the row's wake meets a clean tree.
It refuses a file the verdict parser refuses, naming the format contract as the way forward.

The tapestry, darn and sampler recipes give PR-Review `on_reroute: Code-Bouncer`.
The Code-Bouncer's review input reports the newest send-back file as well as the Parent-Align fixer report.
A PR-findings input starts a full generation, never an amendment, so the next round reviews the whole review range, not only the change since the settle.
After that episode the run goes down its usual rows: Drift-Recheck, Describe, then PR-Review again, which finds the open pull request and, with `pr_review_ask_parent` on, asks the parent again.

The PR-Review brief, `contracts/stencils/landing/landing-template-pr-review.md`, tells the agent to run the verb on every set of findings it receives before fixing any of them: the parent's review, and a list of findings the operator gives.
A single change the operator asks for is no findings set and goes straight to a fix.

Bound: the verb sends a run back only from a PR-Review row that declares the edge, decided by the recipe's rows, never its name.
Shed's reroute cap bounds the edge: past it the run halts for the parent with the cap's way forward.

Rationale: the reroute edge exists, spends no bounce budget, gives the target segment a fresh episode and is capped; the Bouncer's review input and full generation exist, so only the mark, the verb and the second input source are new.

### D5 — The `loom.pr_review_small_max` setting

A new loom config key, `pr_review_small_max`, an integer, default 3, loaded like the other loom keys.
It is the most findings PR-Review fixes in place when none is BLOCKING.
A task may set it in its `run_settings`.

Rationale: what "small" means differs by hub; a count the operator can tune is the simplest bound the board entry names.

### D6 — The miss is recorded

On a send-back the verb writes a friction note through `internal/friction` for the PR-Review row, at a path of its own per send-back, so a later send-back or the agent's own note never overwrites it.
The note says that Code-Review approved the diff, gives the number of findings by severity and each finding's title, and names the stored findings file.
The rubric's `## Review input` section says how the reviewer reads a PR-findings input: review the whole range as usual, and file each PR finding that holds against the code.

Rationale: the friction note is the run record Friction-Reflect already reads, so the miss reaches the operator without a new record type; reviewing the whole range lets the reviewer find the misses' siblings.

### D7 — Figures in the change description

The Describe brief, `contracts/stencils/landing/landing-template-describe.md`, and the darn writer's description rule state that a claim of agreement with a reference or of an improvement carries before and after figures from the current code, taken from the commits or the drift verdict, and that a claim with none is left out.

Rationale: Describe replaces the writer's description after Code-Review, so without this rule a claim D1 kept out of the commits could come back in the pull request's body.

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

- The findings that trigger a send-back are those the PR-Review agent receives as a review; a single operator change request is not one.

## Open risks

- A peer run, `drift-judge-and-line-range`, changes the drift judge's snapshot check and line-range matching in `internal/driftengine` at the same time; D3 touches the judge request and judgement in the same package, so the later run to land meets a merge there.

## Acceptance criteria

- Review: the Code-Review rubric carries the three result-check bullets of D1, worded for any language and hub, and accepts a trailer on any commit of the range.
- Review: every brief of a session that commits code in a recipe with a Code-Review segment includes the shared fragment stating D2's trailer and its shape.
- D3: a Drift-Verify run over a change that widens a limit in a test existing at the review base, with no trailer in the range, spawns the judge even with no changed cell, and the `uncaused` ruling makes the row Stuck with ways forward naming a trailer commit or a revert and `lym loom resume`.
- D3: a test loosened by a parent commit a Parent-Align merge brought in is not listed to the judge.
- D3: a judgement that leaves a listed file unruled, or carries an empty reason, is refused back to the judge.
- D3: Drift-Recheck over files whose diffs are unchanged reuses the stored rulings and spawns no judge for them.
- D3: a judgement record written before this change, without loosened rulings, still reads.
- D3: the regression check section lists each loosened check with its ruling and cause.
- D4: `lym loom pr-findings` on a file with one BLOCKING finding reroutes PR-Review to Code-Bouncer, and on a file with more findings than `pr_review_small_max`, none BLOCKING, does the same.
- D4: the re-entered Code-Bouncer starts a full generation whose review input names the send-back file.
- D4: the PR-Review row re-entered after a send-back opens a new review session instead of reading the send-back as an ending.
- D4: on a file with at most `pr_review_small_max` findings and none BLOCKING, on a recipe whose PR-Review declares no `on_reroute`, or with the target row's entry gate off, the verb answers fix-here and the row does not reroute.
- D4: the verb refuses a dirty worktree, and a file the verdict parser refuses, each naming its way forward.
- D4: the reroute cap halts PR-Review's send-back once exceeded, as for any reroute edge.
- D5: `pr_review_small_max` loads with default 3 and a task's `run_settings` overrides it.
- D6: each send-back writes its own friction note naming Code-Review's miss, the findings by severity and the findings file, and a second send-back leaves the first note in place.
- Review: the rubric's `## Review input` section says to review the whole range and file each PR finding that holds (D6).
- Review: the PR-Review brief tells the agent to run `lym loom pr-findings` with a verdict line on every findings set before fixing any of it (D4).
- Review: the Describe brief and the darn writer's description rule carry D7's figures rule.
