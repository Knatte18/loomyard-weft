# A spawned role loads all its skills at once, not one turn per skill

`skills-in-one-turn` · Note · enhancement, shuttle, stencil

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
