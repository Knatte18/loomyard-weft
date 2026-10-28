# Fixer report — discussion, round 2

## Changes

- F1 (MEDIUM), fixed differently.
  D3 now covers the LLM driver session (`DriverLLM`) as a reader of the escalate halt, D5 lists `contracts/stencils/shed/shed-template-driver.md` and `contracts/stencils/shed/shed-template-driver-guide.md`, and an acceptance criterion pins the skip.
  Not made: the proposed rule that the stencil skips "a stop whose reason opens with the escalate prefix".
  The stencil branches only on envelope fields, never on reason text (`shed-template-driver.md:25` and `:72`, "never match error text"), and the guide's binary-watch exit reads only `lym shed status` fields (`shed-template-driver-guide.md:67`).
  The self-check's fix-way-forward and fix-consistency lenses found this.
  Instead, D3 adds a recipe-neutral `restep_hold` key to the loop envelope and to `lym shed status`.
  The key is computed like `hand_back`, from a recipe's told hold check, and loom and darn tell the escalate predicate.
  The stencil skips a re-launch and starts no binary watch at a stop marked `restep_hold`, so the predicate stays the prefix's only reader.
  Scope, D5's status spec and stencil bullets, the criterion and a plan-writer note follow.
- F2 (LOW), fixed as proposed, then refined by the self-check: the driver criterion now names the re-check, open-attempt and fresh-start paths separately, and the fresh start runs its start check first.
- F3 (LOW), fixed as proposed, then widened by the self-check: D3 scopes the no-re-run guarantee to batten's re-step, and points to D4's Bound for the Go driver's re-step on any start with no open arbiter attempt.
- F4 (LOW), fixed as proposed: D5 lists the `hand_back` section of `contracts/specs/loom-status-spec.md`, and the spec also gains a `restep_hold` section.
- F5 (NIT), fixed as proposed: Turn 4 moved, unchanged, to the end of `## Interview`, followed by a new Turn 5 for this round.
  Q10 joins the question ledger.

## Self-check

- fix-consistency, MEDIUM, merged with fix-way-forward's MEDIUM (the LLM driver skip was a second reader of the prefix, contradicting D3's "only reader"): fixed with own change, the `restep_hold` key.
- fix-consistency, LOW (the Scope list of readers left out the new reader): fixed as proposed; Scope names the `restep_hold` key.
- fix-consistency, LOW (no ledger entry for the LLM driver skip): fixed as proposed; Q10 added.
- fix-way-forward, MEDIUM (the stencil cannot match reason text, and the watch exit cannot see the reason): fixed with the `restep_hold` key the lens proposed.
- fix-way-forward, LOW (the criterion pinned only stencil wording): fixed as proposed; the criterion now pins `restep_hold` in `lym loom status` and the envelope.
- fix-claims, LOW (the fresh-start clause left out loom's start check, `checkBlockedStart` with `finalizeGuard`): fixed as proposed; the criterion and D4's Bound name the start check, which may hand the stop back.
- fix-proposal, LOW (any fresh driver start with no open attempt re-steps, wider than the build hand-over D3 and D4 named): fixed as proposed; D3 and D4's Bound widen to any Go driver start with no open arbiter attempt, bounded by the re-run fixer's gate, caps and decision record.

## Disputed

None.

## Deferred

None.
