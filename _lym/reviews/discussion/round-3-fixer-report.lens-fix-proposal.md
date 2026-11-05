```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fix carried out as proposed (F1, F2 step-5 wording, F3, F4).

**Proposals challenged**
- F1, board-wide refusal of store slugs: one finding (1).
- F2, step 5 releases parts still on the board: no case found. A pruned part is skipped. A removed unreleased part is skipped too, which matches D8's drop. The step 5 write is one atomic board write, so no part can be released only halfway.
- F3, mail first and then the marker, plus the new Fact: one finding (2).
- F4, a failed read still writes the session's file: one finding (3).

**Findings**

1. **LOW.** Changed text: D10's added sentence "also refuses a slug the store holds as a Sketch's slug or as a filed slug … it exempts the entry whose `created` stamp the store records for its own Sketch slug", and the new D10 acceptance bullet.
   - Summary: the exemption covers only the Sketch. A filed entry that is turned back into a task under its own slug is refused, even though the store names that very entry.
   - Case: `Board.MakeSketch` turns a task into a note (`internal/boardengine/sketch.go:49`). A motif run on a filed discussed part would end that way. D2 line 39 also expects "a discussed task made a note again". Promoting that note is a note→task write, which `checkNewSlugs` judges as "made" (`slugrule.go:30`). The new rule refuses it because the store holds the slug as a filed slug, and no exemption applies: the store records `created` stamps only for Sketches, not for filed slugs.
   - Effect: the claim "a slug in the store always names the entry that filed or was raddled under it" is true of this entry, yet the entry is still refused. For a raddle promotion, D2's start guard already refuses it, so the outcome matches but the refusal moves earlier and names a different way forward ("a different slug", where D2 names `lym board rename`). Renaming the note first works, because a note's rename is not judged. The refusal is reachable, but D10 does not say this case is intended.

2. **LOW.** Changed text: the new Constraints Fact "the board entry `pawl-orch-landing` gives Beam-Up the re-invoke interrupt policy … (its D11, stored with that entry on the board)".
   - Summary: the Fact cites a record that disappears once that entry lands and is pruned.
   - Evidence: the artifact's own Fact says "removing an entry deletes its stored record" (`internal/battencli/land.go`, `internal/boardengine/decision.go`). After `pawl-orch-landing` lands, the policy lives in `loomshed.InterruptPolicies`, which `_bolt/decision-pawl-orch-landing.md:163` names. That code location is the citation that lasts.

3. **LOW.** Changed text: the D12 added rule "writes the session's file from `SessionUsage`'s four counts with the cost unknown", and the D12 acceptance clause "still writes its usage file from `SessionUsage`'s counts with an unknown cost".
   - Summary: for Claude, the fallback recovers no counts. Claude's `SessionUsage` is summed from the same transcript and fork transcripts that just failed to read.
   - Evidence: `internal/shuttleengine/claudeengine/usage.go:7`: "SessionUsage sums a finished session's tokens from its transcript and its fork transcripts". `internal/shuttleengine/run.go:295`: the reading "is known only for an OutcomeDone run on an engine implementing UsageReader that could read the session".
   - Effect: for a Claude session whose transcript is gone, the file holds unknown counts. The acceptance clause reads as though counts are recovered. A test would need to assert unknown counts for Claude, or the clause should say "the counts `SessionUsage` holds, unknown where it could not read".
   - Partial fork read: neither the rule nor the clause says what happens when only one fork's transcript fails while the parent's is read.

Tool calls:
- grep of `internal/beamcli/create.go` and `boardengine` note-kind writes, to check whether any flow re-makes a filed slug as a task. My context lacked those files.
- grep of `UsageReader` in `shuttleengine`/`claudeengine`, to see where Claude's `SessionUsage` comes from. My context lacked that source.
