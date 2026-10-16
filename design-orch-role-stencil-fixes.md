# Orch role stencil fixes from the stencil intent audit

`orch-role-stencil-fixes` · Note · bug, stencil, orch, loom, config

Bundle of the orch-role findings of the stencil intent audit of 2026-10-15, all in `contracts/stencils/orch/orch-template-role.md`.
Line numbers as of `main` on 2026-10-16.

## Review escalations: the arbiter is the orch's filter (F8, was `arbiter-filters-orch`)

### Seen

A review loop that cannot settle goes to the arbiter first, and reaches the orch only when the arbiter hands on (`orch-template-role.md:52`, `shed/shed-template-driver.md:63-64`).
The decision is worded three times:

- `loom/loom-template-arbiter.md:65-66` gives the arbiter its rule per cause (`hard-cap`, `circling`).
- `orch-template-role.md:54` gives the orch its own: "accept unless a finding makes the design wrong, and hand a blocker that remains after round 3 to the operator", with a fixed round 3 that ignores the configurable hard cap.
- `bouncer/bouncer-template-escalation.md:31-39` has the orch's one-shot fork settle "on the materials" or report back to the orch, "which asks the operator".

Neither orch text accounts for the arbiter having already applied its rule.

Not every review escalation passes the arbiter:

- an escalation of cause `budget` is never arbiter-eligible and goes straight to the parent (`internal/arbiterstop/doc.go:10-11`); it arises only from a review that does not parse or a ledger or previous decision that cannot be read (`internal/shedadapters/doc.go:80-84`);
- the driver relays the parent notice after an arbiter refusal, timeout or malformed verdict (`shed/shed-template-driver.md:64`);
- a stop with the arbiter switched off or out of spawns is never held for it (`internal/battenshed/doc.go:45`).

### Decision

Operator decision (2026-10-16): the arbiter is the orch's filter.

Operator decision (2026-10-16): the orch role keeps one short fallback rule for an escalation that reaches it with no arbiter verdict (cause `budget`, an arbiter refusal, timeout or malformed verdict, or the arbiter switched off): it is treated as a fault and investigated as a stop, not as a design call.

Operator view, not a decision: the operator doubts this case occurs in practice.

### Idea

Agent's suggestions:

- The arbiter's per-cause rule is the only statement of how a stuck review is decided.
- What reaches the orch with a verdict is the arbiter's hand-on question, which sits in the driver's stop report (`orch-template-role.md:53`), not in the escalation brief: Bouncer renders the brief before the arbiter runs.
  The orch hands its one-shot fork the arbiter verdict path with the brief; the fork answers the question from the operator's recorded decisions, and otherwise brings it to the operator.
- `orch-template-role.md:54` and its fixed round 3 go, replaced by the fallback rule above; the orch role points at the stop report and the brief instead of restating a rule.
- Cross-reference `shed-arbiter-all`, which would give batten's own lifecycle run an arbiter too.

## Comparing related runs is the alignment agent's job (F9, was `orch-no-peer-compare`)

### Seen

`orch-template-role.md:94` has the orch read both Discussions itself at parent-review for related runs, pause both after Plan-Write to compare their plans, and review both PRs side by side.
`orch-template-role.md:92` says the same: "Work that spans runs stays with you: comparing plans across runs, comparing PRs that touch the same packages".
That predates the alignment agent: `:40-45` of the same file has the alignment agent settle parent-review, an escalated review go to a one-shot fork, and the orch's context grow by the fork's summary only.
The alignment agent checks overlap with every run in flight and with the parent's landed commits, at both parent-review and PR-Gate, and escalates a real design conflict (`loom/loom-template-alignment.md:36`, `:55-71`).

### Decision

Operator decision (2026-10-16): comparing related runs is the alignment agent's job; the orch does not do it itself.

### Idea

Agent's suggestions:

- Drop the reading, pausing and comparing in `:94`, and the plan and PR comparing in `:92`, keeping only "deciding when held runs go on".
- Keep "two runs never talk to each other; you align them" only for an alignment escalation that names a peer, handled with the peer's escalation as `:68` already says, by a one-shot fork that reads both decision records.
- Plans are not compared directly, since they follow from the records alignment compared, and a code-level clash surfaces at Merge-In and Merge-Review.

## No repository specifics in the orch stencil; `ORCHESTRATOR.md` (F14, was `orch-hub-rules`)

### Seen

`orch-template-role.md:131` and `:135` are hub-generic but name this repository's `update-plugins.sh`, and tell the orch to run it by hand after a landing while the hub's landing `deploy` key is empty.
This repository's `CLAUDE.md` lets the orch run that script only after its own pushed fix; the lines are dormant in this hub, whose `landing.yaml` sets `deploy`.
`orch-template-role.md:167`, under Where a finding goes, says a workaround the orch applies by hand "is written down only in the repository's CLAUDE.md, with the issue number"; the orch followed it on 2026-10-16 with `977a177cf` (`## Workarounds`, issue #673), so the role still routes orch-side notes into `CLAUDE.md`.

### Decision

Operator decision (2026-10-16):

- The orch stencil carries nothing specific to one repository.
- What an orch must remember for one repository is specific to that repository: it lives in one fixed place and reaches the orch at every context cycle, on top of the orch's general role.
- `CLAUDE.md` holds only what every agent in the repository must have; instructions about the orch's behaviour go to the orch only, since giving them to every agent through `CLAUDE.md` wastes their context.

Operator decision (2026-10-16):

- `ORCHESTRATOR.md` at the repository root is the orch-only counterpart of `CLAUDE.md`.
- The orch watcher renders it into the orch's start prompt, the way `lyx help index` is inlined today.
- A workaround only the orch applies goes into `ORCHESTRATOR.md`, and the role stencil's workaround rule at `:167` changes accordingly; `CLAUDE.md` takes a workaround only when every agent needs it.
- The #673 workaround now in `CLAUDE.md` (`## Workarounds`) belongs in `ORCHESTRATOR.md`.

### Idea

Agent's suggestions:

- Drop `:131`'s tail and `:135` from the orch stencil.
- Track `ORCHESTRATOR.md` in git beside `CLAUDE.md` and `PATTERN.md`; Claude Code does not load it on its own, so no other agent reads it.
- Move `CLAUDE.md`'s orch-only sentences into that file, starting with the orch's `update-plugins.sh` exception under `## Production` and the #673 workaround.

## A small PR-Gate defect has no owner (catalogue gap)

### Seen

`orch-template-role.md:118` says "A small defect in that goal is fixed on the task branch" but names no one who fixes it, while `:99` forbids the orch to edit or commit in either worktree of a task pair.

### Idea

Agent's suggestion: say the reject sends the defect back to the run's own agents (`lyx loom reject` re-spawns the rework writer), so the sentence names who fixes it.
