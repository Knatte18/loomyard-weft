# Shorten the orch role file, the largest single start cost

`orch-role-shorter` · Task · enhancement, orch · running · Code-Bouncer

## Seen

Report `docs/research/tokens/startup-context-2026-11-05/report.md`: `.lym/orch/role.md`, 54,443 characters, costs 19,390 tokens each time it is read, which is every orch start.
It is the largest single source in an orch start, larger than everything Claude Code itself loads.
The operator finds it far too long and sloppily written (2026-11-06).

## Wanted

Rewrite `contracts/orchrole/orch-role.md` so it is much shorter and well written, with about half its tokens or fewer as the aim.

- Keep every rule that binds the orch, in force and meaning: a shorter sentence, never a weaker rule.
  Where two passages say the same thing, keep one; where they disagree, keep what the code does and name it in the change description.
- Move what the orch needs only for a rare verb or event, such as intake, making and landing a beam, landing halts, hibernate or Slack, into files it reads on demand, with one line in the role saying when to read which.
  Prefer files rendered beside the role over a new skill, so no other session sees them.
- The command list repeats what `lym <command> --help` and `lym help index` give: find how it is produced, and cut it to what the orch cannot get otherwise, keeping the rule that the orch uses only the commands open to it.
- Follow `scribe:prose`: point first, one rule stated once, no history, no padding.
  Read the role from the top as a new orch would, and order it by what the orch does: the run loop first, then stops and escalations, then the board.

Anything that reads, renders or tests the role moves with it, including the place it is copied to under `.lym/orch/` and any test that pins its text.
No other session's stencils change, and lym's behaviour does not change.

## Review

The risk is a rule that disappears without notice, so the change description lists each passage dropped, and each passage moved with where it went.
Code-Review checks that list against the old file, and states the start cost in tokens before and after, measured with `docs/research/tokens/startup-context-2026-11-05/measure.sh` or by the same count.
