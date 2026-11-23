## Goal

```markdown
## Seen

While a beam's pull request is recorded, each member landing asks the orch, through its landed line, to add the landing to the change description and run `lym beam describe <slug>`.
The beam orch writes `summary.md` by hand, and nothing in lym rewrites it.
Since `per-task-settings-fixups` the orch gets that line only when the member's `land_notice` is on, so with notices off the pull request's description falls behind.

## Decision

Operator, 2026-11-22: the description is updated at every member landing, and rewritten whole, never appended to.

## Design

- When a member lands under a recorded pull request, lym starts a describer agent on the beam.
  It reads the current `summary.md`, the beam's commits and every member pull request, and rewrites the description as it now stands:
  what the beam ended up with, never the way there, with every member pull request listed.
- `lym beam describe <slug>` then records it and updates the pull request, as today.
- No orch is asked: the landed line drops its follow-up.
- A rewrite that fails leaves the old description standing and is retried at the next landing; the beam orch and the operator can still edit and record by hand.
- Where it runs (dobby after the landing, or a row of the member's landing) and how the describer role is reused from the Describe row is for the run to settle.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Accept round 1: all findings fixed

Decision: Accept the code review at round 1. The BLOCKING finding describer-stencil-restates-summary-format is fixed: beam-template-describe.md points at the final-summary spec instead of restating its format. The fixer fixed F1 to F14, F6 differently: the blast radius stays the one word the final-summary spec allows, beam, and the body names the description, its commits and the pull request.

Rationale: Every finding has a committed fix and the gates passed; F6's alternative follows the spec every gate enforces.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
