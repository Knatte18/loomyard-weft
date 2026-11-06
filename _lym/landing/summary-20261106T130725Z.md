# Orch role: under half the start read, rare events in files beside it

`.lym/orch/role.md` was the largest single source in an orch start.
This change rewrites `contracts/orchrole/orch-role.md`, ordered by what the orch does: the run loop, stops and escalations, the board, then standing rules.
It drops the rendered command index and moves what only a rarer event needs into files rendered beside the role.

## Start cost

The measured cost of one read was 19,390 tokens for 54,443 characters, about 2.81 characters per token.
No live orch start has run this role yet, so the "after" figures use that ratio on the rendered text rather than a `measure.sh` transcript.

| Read | Characters | Tokens at the report's ratio |
|---|---:|---:|
| `role.md` before (current render, command index 19,259 of it) | 56,623 | ~20,160 |
| `role.md` after | 25,073 | ~8,930 |
| `role-beams.md`, on demand | 3,474 | ~1,240 |
| `role-intake.md`, on demand | 2,571 | ~915 |
| `role-landing-halts.md`, on demand | 2,427 | ~865 |
| `role-slack.md`, on demand | 1,297 | ~460 |
| `role-hibernate.md`, on demand | 775 | ~275 |

## What changed in the code

- `contracts/orchrole` embeds the five `role-*.md` texts as `OnDemand`, keyed by rendered file name.
  `RenderRoleFile` writes them beside the role on every render, so every path that renders the role carries them, and no other session sees them.
- The role's `command_index` marker is gone, and with it everything that existed only to fill it: the `index` argument of `RenderRoleFile`, `ResumeContext` and `NewWatcher`, `BeamUpOptions.Index`, and `internal/orchcli/index.go`.
  The role now says `lym help index` lists every command open to the orch, and that it uses no other.
- Stencil scans read the on-demand files through `fixedStencils`, so they meet the same guards as the role.
  A new test pins that each file renders beside the role and that the role names it.
- The watcher's role-render failure tests now break the role path, since the index source they failed through is gone.

## Design decisions

- **The index plumbing is deleted, not redirected to a file.** `lym help index` prints the same list, so a rendered copy would hold nothing the orch cannot get.
  One failure mode goes with it: a bring-up or a context cycle no longer stops because the index could not be read, for example when the strand's recorded `lym` path is gone.
  This is the one change of behaviour, and it follows from dropping the list.
- **The presence line is unchanged.** It still says "the orch role's Slack rules", which now sit in `role-slack.md`; the role's file list sends the orch there while the presence is `slack`.
- **`Never run intake on your own` stays in the role**, under Where a finding goes, because a file read only when the operator asks could not hold it.

## Disagreements settled by the code

- `arras` is no recipe: `contracts/recipes` has none, and the old text listed "tapestry, arras, darn or sampler" in two places while item 1 said "tapestry, motif, darn or sampler".
  Both now say `motif`.
- The old Bouncer bullet said a goto "gives the whole segment a fresh budget", against "a goto keeps the spent budget".
  `internal/shedengine` keeps the budget on a goto, so the bullet now gives only the reason the code bears out: the goto runs a new review round over an approved artifact.
- The old Notices list left out dobby's line about landings into a beam, which `internal/dobbyengine/announce.go` sends. It is now listed.

## Passages moved

| Old section | Where it went |
|---|---|
| Commands | One line after the intro; the list itself is `lym help index` |
| Making a beam | `role-beams.md` |
| Landing a beam, including telling a beam from a project that ran as a task run | `role-beams.md` |
| Landing halts | `role-landing-halts.md`, except "After a run lands, amend the board entries it superseded", now in After landing |
| Escalations: "Never a goto to Plan-Bouncer" | `role-landing-halts.md` |
| Intake | `role-intake.md`, except the rule that it runs only when the operator asks, now in Where a finding goes |
| Acting without asking: hibernate | `role-hibernate.md` |
| Slack | `role-slack.md` |
| Acting without asking: the rest | Working rules; "only design questions" stays once, in Escalations |
| Run loop item 4: mail delivery, stamps, `EXTERNAL`, `lym creel read` | Messaging |
| Intro: `--adopt-local` | Working rules |
| Status reports | Working rules |
| Scratch and notes, Orch rules | Notes and orch rules |
| Landing halts opening: "A task run lands with no orch turn" | After landing |

## Passages dropped

Each is a repeat, a reason, or history; none binds the orch beyond what stays.

- The command list itself, which `lym help index` prints.
- "A rules file kept at `_lym/orch/rules.md` … is no longer read" and its carry-over: history, and `lym orch rules --help` still says it.
- "This role is the same for every orch in every repository": says nothing the orch acts on.
- "heddle delivers every entry aimed at its `orch-prompt` target", shortened to "in an orch session and no other".
- "A landing needs nothing more from you than these steps": padding before the list.
- "A landing-stop halt at Parent-Align or Finalize: see Landing halts below": replaced by the file list.
- "A `[batten notice]` naming a Land or Worktree-Teardown failure is a stop": stated in Notices.
- "a review escalation never takes this bullet and follows the hand-on bullet or the fault paragraph": the bullet already says it is not a review escalation, and the fault paragraph follows.
- "A design call that amends the approved plan … never by a card edit" in Escalations: now a pointer to Fixing a run, which states it.
- "Settle it as a hand-on question" under cause `decision`: the bullet says it arrives as a hand-on, and the hand-on bullet governs.
- "since it tells each live agent of the run and a producer that starts later reads the record", on the Record-Write case: a reason; "with no message of your own" stays.
- "so a report that arrives after a context cycle meets the same re-check": a reason; the re-check rule stays.
- "The repository is its own state", in intake: its consequence stays.
- "A goto … gives the whole segment a fresh budget": see Disagreements.
