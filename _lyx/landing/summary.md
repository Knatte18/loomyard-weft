# Orch role: arbiter-first escalations, ORCHESTRATOR.md rendered in

The hub orchestrator's role stencil now follows the operator's decisions of 2026-10-16 from the stencil intent audit, and each repository gets an orch-only rules file, `ORCHESTRATOR.md`, that lyx renders into the orch's role file.

## What changes

Role stencil `contracts/stencils/orch/orch-template-role.md`:

- A review escalation whose arbiter attempt's `envelope.json` carries verdict `hand-on` and an empty `cause` goes to a one-shot fork.
  The fork answers the envelope's question from recorded operator decisions only, and otherwise reports back so the orch asks the operator.
  The fixed "accept unless the design is wrong, hand on after round 3" bullet is gone; the arbiter's per-cause rule in `loom-template-arbiter.md` is the only rule for a stuck review.
- A review escalation without the arbiter agent's own `hand-on` (cause `budget`, an arbiter refusal, a `hand-on` carrying a `cause`, a job exit without an envelope, a failed re-launch, or the arbiter off) is a fault, investigated under `## Investigating a stop`.
- The design-call bullet covers only non-review escalations.
- Comparing related runs is left to the alignment agent.
  The orch no longer reads both Discussions, pauses after Plan-Write or reviews PRs side by side.
  Peer escalations go to one joint fork, and a second one waits for the first fork's report.
- A small PR-Gate defect is fixed by the run's own agents through `lyx loom reject`.
- Repository specifics are cut: the `update-plugins.sh` and `origin/prod` text in `## After landing`.
  Orch-only workarounds and durable notes are routed to the repository's `ORCHESTRATOR.md`.

Escalation brief `bouncer-template-escalation.md` and the two decide fragments:

- The fork reads the arbiter's `envelope.json` and, when needed, the support log's `## Interview`.
- It settles only when a recorded operator decision answers the envelope's question directly, and records it with `--by parent`.
- It decides nothing at cause `budget`.

`ORCHESTRATOR.md`:

- The role gains a last section, `## This repository`, with a second marker that `internal/orchengine` fills from `ORCHESTRATOR.md` at the prime worktree's root at every role-file render: `start`, the watcher's render after `/clear`, and `resume-context`.
- `internal/orchcli` joins the path and tells it to `orchengine`.
- An absent or whitespace-only file renders one "no `ORCHESTRATOR.md`" line.
- Any other read failure fails the render, naming the file.
- A role stencil without the marker fails the render, so an override cannot silently drop the rules.
- `internal/orchengine/doc.go` describes the section and marker.

This repository:

- A new root `ORCHESTRATOR.md` holds the orch-only rules moved out of `CLAUDE.md`: the `update-plugins.sh` exception, the terminal-window and VS Code rules, and the #673 workaround.
- `CLAUDE.md` loses them and gains one pointer line under `## Notes go in git, not file-memory`.
- The `reed-add-parentless-placement` board note says its #673 removal targets whichever file holds the entry.

## Design decisions

- `--by` on `lyx loom decision add` is an unverified label, so a `--by operator` entry counts as a recorded operator decision only if it is in the committed decision record before the escalation arrives.
  The fork records only with `--by parent`.
- The file is inlined with no size cap, and the section tells the orch to follow the role and raise any contradiction to the operator.
  A landed change to `ORCHESTRATOR.md` reaches the orch at its next delivery, including a landing the alignment agent made with no orch turn.
- `ORCHESTRATOR.md` is read at render time from the prime worktree, never embedded, and `orchengine` is told the path (`PATTERN-told-geometry`).

## To check by hand

- Plan-Review finding `card1-orchestrator-md-inlining-no-bound`: the inlining of `ORCHESTRATOR.md` has no size bound.
  Check `internal/orchengine/prompt.go` and the `## This repository` section against that, as a large file inflates the role file at every cycle.
- Discussion-time design points, worth reading in the stencil text: the board-source provenance and the unverified `--by` label in the recorded-decision definition, and the design-call bullet beside the new escalation rules in `## Escalations from a child`.
- A hub whose deployed role stencil is a hand-edited override refuses to start the orch until the override gains the marker or is deleted.
- If `reed-add-parentless-placement` lands first, the #673 entry in `ORCHESTRATOR.md` is removed by the orch on that run's `landed` notice, per the board note.
- The integration suite (`lyx gate test --tags integration ./...`) was not run by this description step.
