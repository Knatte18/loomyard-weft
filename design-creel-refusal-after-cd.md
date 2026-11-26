# creel post refusal after a persisted cd does not say why

`creel-refusal-after-cd` · Task · bug, creel, cwd

Depends on: [`config-keys-nested`](design-config-keys-nested.md)

Bundled from the notes below; each part keeps its note's text.

# creel post refusal after a persisted cd does not say why

Imported from [issue #245](https://github.com/Knatte18/loomyard/issues/245).

An agent's survey commands used `cd` into subdirectories, and the shell kept that directory for later calls.
The next `lym creel post` was refused because the working directory was no longer the anchored worktree root, which cost a retry.

Why it is abnormal:
- The shell's directory persisting across calls is easy to trip over, and the refusal surfaced only on the later creel call, away from the `cd` that caused it.

What to do:
- Have the refusal name the anchored root and say that a previous `cd` likely moved the shell, or accept calls from any directory inside the worktree.
- Mention in the agent guidance that `cd` persists and absolute paths or `git -C` avoid it.

Provenance: task `retire-d9`; note Darn-Write.md.


