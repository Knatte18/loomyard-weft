# Audit every stencil for design assumptions the operator never meant

`stencil-intent-audit` · Note · audit, stencil

## Seen

On 2026-10-15 the operator found that the Webster master template has told the master to read the codebase up front since at least July, which was never the design (task `webster-master-no-code`).
If one stencil carried a wrong design assumption this long, others may too.

## Idea

An orch agent (Opus) reads every stencil under `contracts/stencils/` and lists, per stencil, each sentence that states a design assumption: what the role reads and when, what it inherits from a parent or master, what it may assume is stable during the run, who it reports to, and what it must not do.
Each entry carries the file and line and a one-line plain restatement.
The agent flags the entries that look doubtful (state assumed stable while other agents edit it, inherited context, duplicated or conflicting rules across stencils) but decides nothing.
The orch shows the operator the list in short batches to confirm or reject; each rejected assumption becomes a note.
The findings go under `docs/research/` through a fresh agent.

## Status 2026-10-15

The audit pass is done and archived in `docs/research/stencil-intent-audit-2026-10-15/`: `catalogue.md` lists every assumption per stencil, `findings.md` the flagged entries F0 to F17, and `README.md` says how to resume.
The review with the operator has not started: no entry is confirmed or rejected, and no finding note is filed yet.
The operator was shown batch 1 (F2 to F5) but gave no verdict.
The session that did the pass can be resumed from the prime: `claude --resume ec3437ad-288b-4c22-b14b-4d3ab4519a4b --dangerously-skip-permissions`.