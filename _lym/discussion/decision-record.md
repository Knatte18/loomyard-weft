## Goal

```markdown
## Seen

`docs/research/hardener-works-study.md` (`c6618b538`) finds that the hardener finds real faults and fixes them with tests that hold, but does not converge on a sound basis:
both campaigns ended on rounds that could not raise what they saw or had not driven what they claimed, and several fixes brought new faults no round caught.

## Idea

The study's "Changes to consider":

1. Exempt a hardener round's findings from the changed-file and changed-hunk rules in `burler-review-format.md` and `burlerengine.CheckReviewRound`, not only `origin: focus` ones.
2. Name FAIL beside WARN in `harden-rubric-review.md`, so a failed focus entry always blocks.
3. Make the rubric and the Bouncer state one convergence rule for a MEDIUM first raised in the closing round.
4. Add a kill rule to the rubric: kill each job from each caller, once with origin applying the in-flight push and once refusing it, then type the printed command in the state left.
5. A prior finding with an undriven half stays `open`; `resolved` needs a whole repro.
6. The fix half drives the neighbours of its fix: the other failure kinds a new classifier receives, and every role that reads a new way forward.
7. The verdict lists what no sandbox can drive (owner takeover, the pull-request sweep, a lost lease) as filed follow-ups.
8. Run the next campaign with one opus round and compare it with the `sonnet[high]` rounds before keeping the default.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-26): Item 8 is a per-campaign setting, not a code change

Decision: This run makes no code change for item 8; the default reviewer in internal/hardenshed/template.yaml stays. The next hardener campaign sets an opus round through hardener.reviewer in its run_settings. Item 7's convergence verdict listing sandbox-undrivable scenarios as follow-ups for the run's parent to file stands.

Rationale: hardener.reviewer is already settable per run; changing the template default would put opus into every campaign, not one.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - harden-bouncer-first-raised-gates-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
