```yaml
lens: fix-way-forward
outcome: reported
```

I checked the ways forward the fix added. One is blocking, two are medium and one is low.

**Ways forward the fix added:**
- (a) Prepare refuses a launch under `.codex/rules/` and names a way forward (D5).
- (b) Prepare warns on an `auth.json` or `sessions` that is not a link: "remove it, and the next Prepare links it again" (D5).
- (c) Config validation refuses an assist timeout that is not below `review_timeout_min` and names both keys (D3, D1 Bound).
- (d) The probe builds the chair's table from the recorded `<round>b` token (D2).
- (e) A done run whose seat gate failed gets the `gate failed` marker text and is not copied (D1, D2).
- (f) The writable root is "to be proven in the build" (D6).

**BLOCKING: the D3/D1 timeout rule refuses a legitimate `review_timeout_min: 0`.** Changed text: D3 "config validation refuses an `assist_reviewer_timeout_min` that is not below `review_timeout_min`, naming both keys", D1 Bound, and the D3 acceptance criterion.
- `internal/loomengine/config.go:1221-1222` accepts zero on purpose: "Zero is deliberately accepted rather than rejected: Spec.Timeout treats 0 as 'defer to shuttle's run_timeout_min', which is a legitimate configuration."
- With the assist default of 60, every `loom.yaml` that sets `review_timeout_min: 0` now fails to load, even when no assist model key is set.
- The rule's way forward points the operator at a valid setting. Validation should treat 0 as deferring to shuttle (or compare against the resolved run timeout), and should apply only when an assist model key is non-empty.

**MEDIUM: the `.codex/rules/` refusal's way forward never reaches the operator.** Changed text: D5 "Prepare refuses the launch … naming the directory with the way forward …; the assistant then does not start, its marker reading `not started`, and the chair goes on".
- The marker text is fixed at `not started`. No Warn is stated, and the chair is told to "go on without the assistant", so nobody passes on the refusal's text.
- D11's twin case does carry it: "with the reason in its marker and a Warn".
- As written, every assisted round under such a repository silently runs without an assistant. The refusal should put its reason in the marker and log a Warn, as D11 does.

**MEDIUM: the second way forward cannot help the round that was refused.** Changed text: D5 "or clear the round's assist model key so no assistant runs".
- D2 says a probe or resume builds from the round's record, "never from config edited since". Clearing the key therefore only changes later rounds, and the refused round already goes on without an assistant.
- The step can be carried out but changes nothing beyond what the refusal already did. The way forward should say it applies to later rounds, or be dropped in favour of the removal path.

**LOW: "to be proven in the build" has no fallback.** Changed text: D6 "the spike did not test `sandbox_workspace_write.writable_roots`, so it is to be proven in the build".
- D2 now depends entirely on this root, and the fix dropped the `/tmp` path.
- If the build disproves the setting, no Decision or Open risk says what the assistant does. Its report write fails and it ends `gate failed` or at its timeout. Nothing tells the plan writer to stop and return to the parent.

Tool calls:
1. Grepped `review_timeout_min` across `internal`, to find where the key is defined and what values it takes; my context did not have the loom config.
2. Read `internal/loomengine/config.go:1215-1240`, to see how the timeout keys are validated, including whether 0 is allowed; my context did not have that block.
