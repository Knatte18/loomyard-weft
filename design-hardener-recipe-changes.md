# Hardener rounds raise what they drive and converge on one rule

`hardener-recipe-changes` · Task · enhancement, burler, stencil · running · Describe

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
