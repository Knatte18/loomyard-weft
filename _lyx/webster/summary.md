# Orch role stencil fixes from the stencil intent audit

The run had one batch, `01-role-file-repository-rules`, which holds all four cards.
It reported `status: done` at head `5f04cd3f335fdd27d1ca4959550aaaf039ae50ee`, with warnings `null`.

The fork reports it built the following; I did not read the code.

- The role file gains a last section, `## This repository`, filled from `ORCHESTRATOR.md` at the prime worktree's root at every role-file render.
- The role stencil's escalation, stop, PR-Gate, landing, finding and notes prose now follows D1–D5.
- The Bouncer escalation brief settles only on a recorded operator decision and decides nothing at cause `budget`.
- `ORCHESTRATOR.md` was created with the moved orch-only rules, and `CLAUDE.md` now carries a pointer line.
- The #673 board note was added.

## Scope guard

The scope guard listed one deviation, `internal/orchengine/prompt_test.go`, a file changed outside the batch's own targets.
It is informational and did not stop the batch.
