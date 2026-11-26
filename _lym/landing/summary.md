# Hardener rounds raise what they drive and converge on one rule

`docs/research/hardener-works-study.md` found that the hardener finds real faults but does not converge on a sound basis.
Closing rounds could not raise what they saw or had not driven what they claimed, and some fixes brought faults no round caught.
This change applies the study's "Changes to consider" to the hardener's round check, Bouncer, stencils and recipe.

**What a hardener round may raise**

- `burlerengine.CheckReviewRound` no longer holds a hardener round to the BLOCKING-only or changed-file rules.
  A new finding takes the severity it has and sits where the fault lives, and a carried finding may move to any file.
  `burler-review-format.md` and `burler-step-review.md` say the same.
- `RoundContext.FocusFindings` is renamed `HardenerRound`, along with the matching fields in `shedadapters`.
  The narrower `origin: focus` exemption and the `FocusOrigin` constant are removed.

**Convergence**

- New `BouncerConfig.FirstRaisedGates`, set by `shedrecipe` on `harden_round` rows, makes the judge rule every round by round 1's rule.
  A gating finding first raised in the closing round then holds convergence, because its fix is behaviour no round has driven.
  Other Bouncers keep the lighter rule from round 2 on.
- `harden-rubric-review.md` states the same rule: no BLOCKING finding, no `design` finding at MEDIUM, and a green live gate.

**What the rubric and recipe now require**

- A focus entry recorded FAIL is a BLOCKING finding.
- Faults in module behaviour are classed `design`, which is the gating class.
- A prior finding is `resolved` only when its whole reproduction was driven on the fixed build.
- Each job is killed from each caller, once with origin applying the in-flight push and once refusing it, and the printed way forward is then typed.
- The fix half also drives the neighbours of each fix: other failure kinds a changed classifier receives, and every role and verb that reads a changed way forward.
- A convergence verdict lists the scenarios no sandbox can drive as follow-ups for the run's parent to file.

## Design decisions

- The study's item 8, comparing an opus round with the `sonnet[high]` rounds, makes no code change.
  The default reviewer in `internal/hardenshed/template.yaml` stays, since changing it would put opus into every campaign.
  The next hardener campaign sets an opus round through `hardener.reviewer` in its `run_settings`.
- The exemption covers every finding of a hardener round, not only focus ones, because the round drives behaviour that the changed hunks do not bound.
  The other round checks still apply, such as the refusal of a carried finding raised above its earlier severity.

## To check by hand

- `harden-bouncer-first-raised-gates-untested`: nothing tested that only a `harden_round` Bouncer row sets `FirstRaisedGates` (`internal/shedrecipe/entries_bouncer.go`).
  `BouncerRowGatesAFirstRaisedKeyOnlyWithHardenRound` in `internal/shedrecipe/entries_harden_test.go` now covers it; check that it pins the wiring.
- The rubric and recipe instructions steer agents, so their effect shows first in the next hardener campaign.

## Merge danger

**Door:** two-way
**Blast radius:** hardener
