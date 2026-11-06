# Shorten the orch role to under half its size, rare events to files

The orch role was the largest single source in an orch start: 54,443 characters, 19,390 tokens, read on every orch start.
This rewrites `contracts/orchrole/orch-role.md` to about 25,800 characters, 47% of the old rendered size, and moves what only a rare event needs into files the orch reads on demand.
No rule is weakened, and lym's behaviour does not change beyond what the role file says and how it is rendered.

## What changes

- The role is rewritten in the order the orch works: the run loop, then notices, stops and escalations, then fixing a run, landing, the board, and messaging.
  Duplicated passages are stated once, and history is cut.
- The command list is gone.
  It was the rendered `lym help index`, about 15,600 characters of the old file.
  The role keeps the rule that the orch uses only the commands open to it, and points at `lym help index` and `lym <command> --help`.
- Five on-demand files render beside the role, each named in the role with one line saying when to read it:
  - `role-beams.md`: making a beam, landing a beam, and a project that ran as a task run.
  - `role-landing-halts.md`: any halt at Parent-Align or Finalize.
  - `role-intake.md`: when the operator asks for intake.
  - `role-hibernate.md`: when the operator asks to hibernate the machine.
  - `role-slack.md`: mail from `<shortname>:SLACK`, and while the presence is `slack`.
- No skill is added, so no other session sees these files.
  `orchrole.OnDemand` embeds them, and `RenderRoleFile` writes them beside the role before the role itself, so a role the session reads never names a file that is not there yet.

## Code that moves with the role

- `RenderRoleFile` no longer takes an index, and the `command_index` marker is gone; only `decision_note_title` remains.
  `ResumeContext`, `NewWatcher`, `startOrch`, `BeamUpOptions` and `beamcli`'s start drop the index parameter and source.
- `internal/orchcli/index.go` is deleted, with `indexRunner` and the watcher's index source.
  A role render can no longer fail because the orch strand recorded no `lym` binary or a stale path, so the watcher's idle-with-reason case for that is gone, along with its tests.
- Tests that pinned the old text or the index move with the change: `contracts/stencils` marker and registry tests, `orchengine` prompt, resume-context and watcher tests, and `orchcli` start, watch and smoke tests.
- Docs follow: `docs/overview.md`, `PATTERN-stencil-ownership.md` (the on-demand files share the role's exception), and the `doc.go` of `orchcli`, `orchengine` and `helpcli`.
- No other session's stencils change.

## Passages dropped

- `## Commands` and its `{{.command_index}}`: replaced by the pointer to `lym help index`.
- The `_lym/orch/rules.md` paragraph in `## Orch rules`: history of a file that is no longer read.
- `## Acting without asking` as a section: its rules are in `## Working rules`.
  Its sentence "Ask the operator only design questions…" repeated `Escalations from a child`, and the go-ahead part stays in "A question is never a go-ahead".
- "each is handled as under Investigating a stop" after the notice lines: the section's lead sentence says it once.
- The sentence in the design-call bullet that a review escalation never takes it and follows the hand-on bullet or the fault paragraph: the bullet still says it covers an escalation that is not a review escalation.
- "never by a card edit" in the same bullet: `Fixing a run` keeps "never edit a card yourself".
- "The repository is its own state:" in Intake: the next sentence carries the point.
- In `## Orch rules`, the phrases "after lym's own sentences" and "heddle delivers every entry aimed at its `orch-prompt` target to an orch session, and to no other session": the shortened text keeps that the rules arrive appended to the system prompt, in an orch session and no other.

## Passages moved

- `## Making a beam`, `## Landing a beam`, and the project-ran-as-a-task-run paragraphs go to `role-beams.md`.
  The role's `Notices` section names landings into a beam.
- `## Landing halts` goes to `role-landing-halts.md`.
  The rule against a goto to Plan-Bouncer went with it.
  "After a run lands, amend the board entries it superseded" moved to `After landing` in the role.
- `## Intake` goes to `role-intake.md`.
  The rule that intake runs only when the operator asks stays in the role, under `Where a finding goes`.
- The hibernate paragraphs of `## Acting without asking` go to `role-hibernate.md`.
- `## Slack` goes to `role-slack.md`.
- The wake-up and creel-mail mechanics of run loop item 4, and the other-hub stamp and `EXTERNAL` rules, move into `## Messaging`.
- The "A landing that succeeds sends nothing" sentence moves from `Notices` to `After landing`.
- `## Status reports` and `## Scratch and notes` fold into `## Working rules` and `## Notes and orch rules`.
- `Never pass --adopt-local to lym fabric add` moves from the opening to `## Working rules`.

## Disagreements settled

- The old role named the inner runs "tapestry, arras, darn or sampler"; the rewrite says "tapestry, motif, darn or sampler", the name the run loop item already used.
  Check this against the recipe names the code accepts.

## To check by hand

- Start cost in tokens before and after, measured with `docs/research/tokens/startup-context-2026-11-05/measure.sh`; the character counts above are not a token count, and the on-demand files are read only on their event.
- Each passage in the two lists above against the old file (`git show b30cccce:contracts/orchrole/orch-role.md`).
- A fresh orch start: the role renders, and the `role-*.md` files appear beside it under `.lym/orch/`.
