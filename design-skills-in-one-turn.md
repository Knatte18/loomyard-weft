# A spawned role loads all its skills at once, not one turn per skill

`skills-in-one-turn` · Note · enhancement, shuttle, stencil

Since `70bf30ae8`, `claudeengine` types each of a role's skills as its own `/<name>` turn before the prompt pointer (`shuttleengine.Spec.Skills`).
Each turn is a full model turn that ends in a reply like "Loaded scribe:prose … What would you like to work on?", so a role with three skills spends three slow turns of chatter before it reads its task.
Seen 2026-10-05 on `ly:selfreport-issues-say-nothing:burler`; the operator: loading the skills is right, but all of them should load at once.

Claude Code treats only a message's leading `/<name>` as a command, so several skills cannot share one typed message.
Keep the guarantee that every skill is loaded, without a turn per skill:

1. Inline the skill bodies into the rendered prompt file, the way the `PATTERN.md` overview is inlined: zero extra turns, and the content is there whether or not the model acts on it.
2. Or one message asking for every skill through the Skill tool in a single turn; one turn, but the model may skip a load again.

The orch watcher types its skills the same way at each cycle; the same fix applies there.
