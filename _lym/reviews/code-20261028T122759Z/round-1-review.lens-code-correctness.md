```yaml
lens: code-correctness
outcome: reported
```

**Task:** the correctness lens. I ran one concrete input through each branch the diff changes. The list comes first, then the findings.

**Branch list**
- `gateSegmentPattern` and `MatchPath`, incident command `lym gate test … --card 6-…-weft 2>&1 | tail -40`: **correct**. The segment ends at the `&` of `2>&1`, which is harmless here because the card ID comes before it. The text blanked equals the segment length, so no offset is used.
- `MatchPath`, command chained after `git commit -m "…\n…" &&`: **correct**. `blankQuoted` blanks the quoted newline first, and the match starts at the second `&`. The segment is blanked.
- `MatchPath`, `lym gate test ../other-weft/x`: **correct**. `pathPattern` runs on the unblanked `cmd` and still matches.
- `MatchPath`, `lym gate test --card 6-prime-weft && git -C other-weft status`: **correct**. The `&` ends the segment, and the later bare name still matches.
- `MatchPath`, `echo lym gate other-weft` (gate not in command position): **correct**. The pattern does not match, so the name still counts as a reference.
- `MatchPath`, `lym gate test 2>&1 --card x-weft`: **minor gap**. The `&` in `2>&1` ends the segment early, so a bare name after the redirect is still flagged. The result is a false positive, not a bypass (F1).
- `AcceptBatchFabricReference` after the split: **correct**. The checks and their order are unchanged, the mutation happens only after the evidence passes, and the dirty-tree text is character-identical. The new `readOnly == nil` guard replaces a former nil-function panic.
- `batchFabricReferenceEvidence`, HEAD at start on a clean tree: **correct**. It returns the at-start basis and never calls `readOnly`, as before. The not-failed and not-pathless refusals are unchanged.
- `acceptAuditWayForward`, evidence holds (at start, clean): **correct**. It names accept-audit then recover-batch, with the reset-to-start route appended.
- `acceptAuditWayForward`, no recorded start: **correct**. The reason already contains `stepResetToStart` through `resetToStartSteps`, so nothing is appended twice.
- `acceptAuditWayForward`, read-only refusal past start: **correct**. That text holds `reset --to batch-start`, not `reset --to start`, so `freshRestartSteps` is appended. The prefix trim matches `%w: ` exactly.
- `acceptAuditWayForward`, dirty-tree refusal: **correct**. `rerun` is the "re-run accept-audit then recover-batch" text, and the reset-to-start route is appended.
- `acceptAuditWayForward`, git read failure: **correct**. A non-`ErrAuditNotAcceptable` error is returned unwrapped at `recoverbatch.go:541-543`. The git read failure replaces the refusal, which is acceptable.
- `contracts.Rest` against `bs.Uncheckable`, `recoverbatch.go:524-541`: **correct**. `Rest` is a subset of `prior.Uncheckable`. If the batch also holds contract-path entries, the evidence refuses as "not pathless" and names reset-to-start. accept-audit would refuse the same way, so the advice matches.
- `PlanReset` run-lock refusal: **correct**. `rerun` is defined at `reset.go:131`, before it is used. `report-head` still skips the branch. The message is well formed for `ResetToStart` with batch 0.

**Findings**
- F1, LOW, design, `internal/fabricengine/refscanner.go:29-34`: `gateSegmentPattern`'s `[^;&|()\n`]*` stops at the `&` of `2>&1` or `>&`. A bare weft-suffixed card ID placed after a redirect, e.g. `lym gate test 2>&1 --card x-weft`, is not blanked and is still flagged. The result is a false positive only, and no audit bypass follows. It is the same false-positive class as D1 on a rarer argument order. A fix would treat `>&` and `&>` as non-separators in the segment pattern, or document the limit.

**Tool calls**
1. Read `internal/websterengine/recoverbatch.go:490-545`. My context lacked the code around the `acceptAuditWayForward` call site, which I needed to see whether `contracts.Rest` agrees with `bs.Uncheckable` and how `err` is scoped.
