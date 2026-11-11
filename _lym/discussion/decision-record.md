## Goal

```markdown
## Seen

On 2026-11-09 the operator reported that Codex has so far been mostly trouble in lym: it insists on running under its bubblewrap sandbox, which gets in the way of the work lym gives it.
The `astra` model (`gpt-6-astra` in `models.yaml`) fails on OSL-1046 for that reason.
`testmap-verb`'s first Code-Review round recorded its astra assist only as `assist_ending: stopped`, with no error.
The operator then had astra removed from every task's `run_settings`.

`codexengine` already has a way out: `codex.yaml`'s `sandbox_mode` may be set to `danger-full-access` on the machine layer, which launches Codex with no sandbox.
Nothing sets it or says it is needed: the default stays `workspace-write`, and a machine whose bubblewrap cannot create a user namespace only sees a seat that stops.
OSL-1046 runs Ubuntu 24.04, whose AppArmor restricts unprivileged user namespaces by default; check whether that is the cause.

## Fix

- Find what makes Codex's sandbox fail in a lym session on a machine like OSL-1046, and settle the sandbox mode lym launches it with,
  so a Codex seat runs there like a Claude one without the operator setting anything by hand.
- lym reports a Codex seat that cannot start, naming the cause, instead of ending it quietly as stopped.

The run works on Neve and cannot reproduce on OSL-1046 itself; whatever it cannot confirm there it states in its summary.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - openlaunch-workspace-write-refusal-untested (class design, severity MEDIUM, open in rounds 1)
  - probe-error-degrades-auto (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
