# A spawned role loads all its skills at once, not one turn per skill

`skills-in-one-turn` · Task · enhancement, shuttle, stencil · running · Plan-Burler

Since `70bf30ae8`, `claudeengine` types each of a role's skills as its own `/<name>` turn before the prompt pointer (`shuttleengine.Spec.Skills`).
Each turn is a full model turn that ends in a reply like "Loaded scribe:prose … What would you like to work on?", so a role with three skills spends three slow turns of chatter before it reads its task.
Seen 2026-10-05 on `ly:selfreport-issues-say-nothing:burler`; the operator: loading the skills is right, but all of them should load at once.

Claude Code treats only a message's leading `/<name>` as a command, so several skills cannot share one typed message.
Keep the guarantee that every skill is loaded, without a turn per skill:

- Chosen: one typed message asking the model to load every listed skill through the Skill tool in a single turn and reply only "ok".
  Loaded skills are registered as such, so Claude Code re-injects them after a compaction, and the instructions arrive as user input.
  lyx checks the transcript for each skill's load and retries a missing one.
- Rejected: inlining the skill bodies into the prompt file.
  It costs no turn, but the text arrives as a Read result rather than user input, and a compaction summarizes it away instead of re-injecting it, which matters for long-lived sessions such as the Webster master.

The orch watcher types its skills the same way at each cycle; the same fix applies there.


## From issue #394

Imported from [issue #394](https://github.com/Knatte18/loomyard/issues/394).

A skill the session does not know stalls a shuttle launch for the whole skill timeout, though a missing skill should be skipped at once.

Seen 2026-10-05 with `lyx orch start --handoff .scratch/handoff.md`: the `ly` plugin was read from a stale checkout without the `board` skill.
Shuttle typed `/ly:board`, Claude Code answered `Unknown command: /ly:board` in the pane, and the launch sat there until the skill timeout (600 s) skipped it.
To the operator, `lyx orch start` looked hung.

The orch docs (`internal/orchengine/doc.go`, "The reload sequence") already say a skill step is skipped "when the provider reports it unknown", but the provider does not see this answer:
`Unknown command` is printed by the slash-command parser, ends no turn and fires no Stop hook, so the step waits for a turn end that never comes.

Fix: detect the unknown-command answer (pane text after typing, or the transcript's local-command record) and skip the skill at once with a Warn naming it, in shuttle's launch skill loading and in the orch watcher's reload sequence alike.
A missing skill should never block a launch.
