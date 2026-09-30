Friction from run `operator-surface` (Plan-Write, trace `7b2a74344919ecc3`).
1. A respawned Plan-Write moves an existing finished plan to `_lyx/plan/archive-<ts>/` and starts from an empty directory; the new session learned of the prior plan only from a cross-session message.
   The false `asking` that caused the respawn is fixed in `083e992cf`, but any respawn still discards the plan: a respawn should reuse or at least point the prompt at the archived cards.
2. A PreToolUse hook refuses the Agent tool in the Plan-Write pane and the prompt does not say so; five parallel explorer dispatches failed before work started.
   The prompt should state the tool restriction (see Someday "shuttle `Spec`: generic tools-restriction").