# Hardener rounds raise what they drive and converge on one rule

The hardener study (`docs/research/hardener-works-study.md`) found that the hardener finds real faults but does not converge on a sound basis.
A round could not raise faults it drove off the changed hunks.
The rubric and the Bouncer stated different convergence rules.
Several fixes brought new faults that no round caught.
This change applies the study's "Changes to consider" 1 to 7.

## What changes

- **A hardener round raises every fault it drives** (`internal/burlerengine/roundcheck.go`).
  An incremental round refuses a new finding below BLOCKING or outside the changed files.
  Until now only an `origin: focus` decision finding was exempt in a hardener round.
  `RoundContext.HardenerRound`, renamed from `FocusFindings`, now exempts every finding of a hardener round:
  a new finding passes at any severity and in any file, and a carried finding may move to any file.
  The parent-report rule and a carried finding's class and severity rules still hold.
  `burler-review-format.md` and `burler-step-review.md` state the same rule.
  The unused `FocusOrigin` constant is removed.
- **One convergence rule** (`internal/shedadapters/bouncerprompt.go`, `bouncer.go`, `internal/shedrecipe/entries_bouncer.go`).
  From round 2 on, the judge's decision rule lets a gating key first raised in the latest round converge, carried into `## Open risks`.
  The hardener rubric said instead that CONVERGED needs no MEDIUM finding.
  The new `BouncerConfig.FirstRaisedGates`, set for every `harden_round` Bouncer, renders round 1's rule in every round.
  The rubric's Converged bullet now states that rule:
  no BLOCKING finding, no `design` finding at MEDIUM, a key first raised in the closing round included, and a green live gate.
  The rubric classes every behaviour fault `design`, so the Bouncer's gating class covers it.
- **Rubric rules** (`contracts/stencils/harden/harden-rubric-review.md`):
  - a focus entry recorded FAIL is a BLOCKING finding of the fault, located where the fix goes, so the fix half fixes it;
  - kill each job from each caller twice, once with origin applying the in-flight push and once with origin refusing it,
    then type the printed way forward, placeholders filled, in the state left;
  - a prior finding stays `open` until this round has driven its whole reproduction;
  - a convergence verdict lists each scenario no sandbox can drive, such as an owner takeover, the pull-request sweep or a lost lease,
    as a follow-up for the run's parent to file.
- **The fix half drives the neighbours of its fix** (`contracts/recipes/hardener-recipe.yaml`):
  every other failure kind that a classifier the fix adds or changes receives,
  and every role and verb that reads a way forward the fix adds or changes.

## Design decisions

- The rubric's rule moves to the Bouncer's gating class rather than the Bouncer counting every class as gating.
  Go's facts count, circling evidence and carry-over all read gating as `design`.
  So a MEDIUM `consistency` finding, a doc or message that disagrees with a verb, no longer holds a hardener campaign from converging.
  It is carried into `## Open risks` instead.
- Item 8, running the next campaign with one opus round, needs no code change, as the parent recorded.
  `hardener.reviewer` is already settable per run in `run_settings`, and the template default stays `sonnet[high]`.
- The judge files no follow-ups itself.
  The verdict names them, and the run's parent files them.

## Tests

- `TestCheckReviewRound` gains hardener rows:
  new MEDIUM and LOW findings on untouched files pass, a carried finding moves to an untouched file with and without a changes file,
  a raised carried severity is still refused, and an uncovered `origin: parent` finding is still refused.
- `TestDecisionRuleMarker_CirclingOnlyFromTheCheckpoint` gains rows where first-raised keys gate.
- `TestBouncer_JudgePromptGatesAFirstRaisedKeyWithFirstRaisedGates` drives a Bouncer through two judge rounds.
  It checks that round 2's prompt carries the lighter rule only when the setting is unset.

## Merge danger

**Door:** two-way
**Blast radius:** hardener
