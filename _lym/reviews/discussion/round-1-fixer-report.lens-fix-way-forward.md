```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward the fix introduced (lens fix-way-forward).

No findings.

The change set adds no refusal, error or next step:
- **D2:** the viewer text only restates reed's existing rule for a viewed window that ends, and the criterion that rested on the dropped claim is gone.
- **D3:** the criterion's new case, a grouped strand placed apart, can be reached from both states it names:
  - a declared size above `window_share_max`, which `planOwnWindowPlacement` places apart through `!group.Shared`;
  - a group whose shared window is full, through `liveCount >= size`.

  The Constraints bullet puts window naming in the tmux tier, so that case has a test tier.
- **D4:** the criterion's condition (a seed and a status file that is not done) can be built with injected pollers, the same as before.

No further tool calls made.
