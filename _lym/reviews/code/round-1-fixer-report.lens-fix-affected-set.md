```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set — checked the sets under F2 (engines reading `spec.Skills`), F5/F12 (consumers of `StepStopReport`) and F1 (callers of the changed startup helpers).

**MEDIUM — F2: both engines' launch manifests still expect the skill-load turn when `SkipSkillLoadTurn` is set**
- Changed text: `SkipSkillLoadTurn` gating in `claudeengine.go` (`loadsSkills`) and `codexengine.go` (`!spec.SkipSkillLoadTurn`).
- The fix gated the launch line in `Prepare` but not the manifests built from the same spec.
- Evidence:
  - `claudeengine/manifest.go:209-215` adds a `ChannelSkillLoad` / `DeliveryDelivered` candidate for every `spec.Skills` entry, proved `inLaunchLine`, with the skill list as the proof text.
  - `codexengine/manifest.go:190-199` does the same, with the skill file path as the proof.
  - On a resume the launch line carries only the prompt pointer, so each candidate fails its proof. That makes the resumed launch's `manifest.json` report the skills as omitted in its `Verdict`.
  - This is the same manifest the fix now copies to `manifests/` for F3.
- Fix: skip these candidate loops when `spec.SkipSkillLoadTurn` is set.
- The review named only `claudeengine.go:243` and `subagents.go:63`, so this member of the set was missed.

**No findings for the other sets**
- F2 engines: `Prepare` is implemented by `claudeengine`, `codexengine` and the test fakes. `shuttlefake` never reads `Skills` or `PromptLine`, and `claudeengine/command.go` reads `OpenSpec.Skills`, a separate type with its own skill-load turn.
- F1 callers: `checkLivenessTick`, `sessionStartRecorded` and `turnStartedAfterSessionStart` are only called in `wait.go`, and every call is updated.
- F5/F12 consumers: `StepStopReport` is consumed only by `loomcli/looparm.go`, which renders both new fields, and by the `doc.go` paragraph.

Tool calls (4 of 6, each a lookup my context lacked):
- Grep of `spec.Skills` across non-test `.go` files and `Prepare` implementers, to find every engine and manifest reader.
- Grep of `StepStopReport` and its field readers, and of the startup-helper callers, to find every consumer and call site.
- Read of `shuttlefake.go` (grep for `Skills`/`PromptLine`), to see whether that fake needed gating.
- Read of `claudeengine/manifest.go` lines 200-225 and `codexengine/manifest.go` lines 183-200, to confirm the manifest skill candidates.
