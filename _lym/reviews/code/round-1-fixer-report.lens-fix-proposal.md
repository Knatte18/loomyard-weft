```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposed fixes for F1 and F2 as the fixer carried them out.

No findings.

- **F1** (unconditional `idle-stalled`/`no-output` in `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`):
  - I found no case where the proposal is wrong.
  - With no registry entry, the liveness is unproven.
  - The review says an unproven liveness now settles nothing, so the hook-derived reading holds on every platform, and the gate run passed.
  - The fixer kept `t.Setenv("HOME", ...)` and the `registry` `MkdirAll`. This still isolates the test from the real `~/.claude/sessions`, so it is not dead setup.
  - Removing the `proc` import is safe: the only use was the deleted line, and the gate compiled.
- **F2** (hidden-only explanation in the `--anchor` refusal):
  - Hidden gets the full text, and every other value gets `invalid --anchor %q; want own-window`.
  - The comparison `anchor == string(render.AnchorHidden)` can't misfire on the other rows. `overview`, `sideways` and the empty string all take the plain branch.
  - The `AnchorOverview` and `AnchorEmpty` rows assert the substring `own-window`, which the plain message still contains.
  - The `AnchorHidden` row now pins the whole hidden text, including the `--focus=false` clause, and the `AnchorUnknown` row pins the plain text with the quoted value.
  - The message is built before the prompt file is read and before `c.runner`, so the ordering the review praised is unchanged.

Tool calls made: none. My context already held every changed file and the gate result.
