```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round's change set.

**Findings**

- **LOW**, `internal/gateslot/config.go` (the `ResolveCommandDir` comment) and `internal/gateslot/doc.go:68`, against `internal/driftengine/doc.go:8`.
  - The fix reworded the gateslot claim to "the rule the drift steps and the hub-wide dir checks resolve a dir by".
  - `driftengine/doc.go:8` still says "A step's `dir` resolves by gateslot's one rule for every hub command, so it means the directory a gate tier's `dir` means".
  - That is the "one rule" claim F9 asked to retire, and it now contradicts the reworded gateslot text, which says the gate run path reaches the same directory a different way.
  - Fix: reword `driftengine/doc.go:8` to say a step's `dir` resolves by `gateslot.ResolveCommandDir`, and means the directory a gate tier's `dir` means.
  - I did not open `driftengine/doc.go` this turn. The line comes from the grep output above.

- **LOW**, `internal/driftengine/seats.go` `captureReading` against the doc comment of `Deps.BaselineTable` (and the `BaselineInputs` users).
  - `AnchorDir` is now a required `BaselineInputs` field, and `requireTold` refuses it empty.
  - The grep shows only `loomcli/driftwiring.go` and `seats_test.go` construct `BaselineInputs`, and both were updated, so no caller is left broken.
  - I did not check whether the `Deps.BaselineTable` doc or `driftengine/doc.go` list the told inputs. A stale list there would be the twin.

- **Checked, no finding**:
  - No stencil text names the anchored directory. `contracts/stencils/drift/drift-template-baseline.md:29` only renders `{{.capture_command}}`, so the new wording fits it.
  - The three `Site` literals have no other consumer: the only match outside the constants is a `"drift compare"` substring in `verbs_test.go:314`, which is unaffected.
  - `configreg.go:84` and `driftengine/config.go:45` and `:226` already describe the worktree root plus anchor, so they agree with the fix.

**Tool calls**
- One `grep` for "anchored directory", "one rule" and "one base" across `internal` and `contracts`, plus a `grep` of the baseline stencil. My context did not hold those files.
