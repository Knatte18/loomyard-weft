```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F5 change set for stale siblings and unmatched twins (fix-consistency lens).

**Findings**

1. **LOW** — `internal/fabricengine/handover.go`, `HandOverTips` doc.
   - It still says the code tip is pushed "when origin's code branch lags behind it", and ends "It pushes no branch but the owed code tip."
   - `pushOwedCodeTip` now also creates an absent origin branch, and on give-up deletes the branch it created and pushes an owner archive tag.
   - Evidence: the new `originAbsent` path and `deleteGivenUpCodeBranch`.
   - The `HandOverResult.TipPushed` doc says the given-up tip is "moved off origin again"; the absent case deletes the branch, so that text is also incomplete.

2. **LOW** — `internal/beamcli/land.go:44`, the `PushBeam` doc ("pushes the beam branch and returns the head it pushed").
   - `PushBeam` now goes through `pairteardown.PushRun`, so it can return an error wrapping `fabricengine.ErrHandedOver` after a hand-over.
   - I did not check that the doc or callers (`pr.go:81`, `pr.go:172`) account for this. I did not read the doc text itself, only the grep hit.
   - Separately, `RunPush.InsidePair` stays false, so a hand-over also removes the beam's local pair. This holds even when the verb runs from inside it, the case `InsidePair` exists for. Evidence: `pairteardown/handover.go` `HandOver` doc.

3. **LOW** — test twin for F2.
   - The beam takeover path got no test, while `TestRejectedRecordsPush_ReadsTheTakeover` covers the fabric and `PushRun` paths only.
   - Twin callers are consistent. `fabriccli/weft_verbs.go:297` passes `Here` but no `RunSlug`, which is the verb the plan says it wires.

No inconsistency found for F3, F4 and F5:
- The `pushParent` doc was updated with the new re-record step.
- The extra `recordLanding` call is the only other place `ArchiveParentMerge` is consumed (`finalize.go:735`).
- Both `testtiming:keep` lines sit directly above their funcs.

**Tool calls**
- One grep for `PushPairAnchored(`, `PairPush{` and `ArchiveParentMerge()` callers outside tests, because my context held no list of twin call sites.
