```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether the round-2 change set agrees with its sibling decisions, criteria and cross-references, and whether any changed text has an unchanged twin elsewhere.

**MEDIUM: D9's guard exemption contradicts its own sentence and drops coverage the old text had**
- Changed text: D9, "a change in either tree that the session made, other than `judgement.yaml` and the drift records Drift-Verify itself writes, is a Stuck".
- The same sentence opens "The judge edits no code, no expectation and no record".
- The exemption covers changes *the session made* to `scope.json` and `judged.json`. A judge that writes a `matches` tuple into `judged.json`, or edits the newly recorded snapshot in `scope.json`, now passes the guard.
- The pre-fix text exempted only `judgement.yaml` ("a change under `_lym/drift/` other than `judgement.yaml` that the session made is a Stuck"), so the old guard caught these edits.
- The D9 acceptance criterion names no such exemption, so the decision and its criterion disagree.

**MEDIUM: the command-mode "passes" twins were left unchanged after the new digest failure**
- Changed text: D7, "Every check, in either diff mode, … first recomputes it. A mismatch, or a missing baseline directory, stops the check, as … a failure at the entry". D8 adds "It fails … when the counted baseline directory is missing or its digest mismatches".
- Three unchanged twins still say the entry passes unconditionally under `command`:
  - D8's next line: "In `command` mode it runs the seal check (D6) and the compare, reports any seal failure, and passes".
  - D2: "The writer's gate entry and `lym drift check` (D8) … report any seal failure, and pass".
  - The D2 criterion: "the drift gate entry passes, reporting the compare's exit…".

**LOW: D4's Drift-Expect row does not mention the digest backfill**
- Changed text: D7, "A capture counted before this lands, with no digest, gets one at Drift-Expect".
- D4 row 3 still describes Drift-Expect as "spawns the expect agent (D5) and seals its file (D6)".
- D4 row 2 was updated to name the digest for Drift-Await, but this twin was not.

**LOW: D16's judge stencil does not mention the spawn id**
- Changed text: D9, "hands the judge a spawn id; the file carries that id, and the `judgement` entry refuses a file carrying any other".
- D16's `drift-template-judge` text is unchanged and never tells the judge to write the id.
- The Review criterion checks only "what D16 names", so nothing checks the stencil for it.

**LOW: support-log Q5 now contradicts D2**
- Changed text: D2, "A `does not match` or `unexpected` ruling is stored in `judged.json` keyed by (warp head, case)".
- `support-log.md` Q5 still reads "the judge reading its log per case and no stored rulings. Resolved (D2)".
- D2's expectation also changed to per case, which Q5 does not reflect.

**LOW: D11's "a resumed row checks again" no longer fits command mode**
- Changed text: D2's stored failing rulings.
- D4's criterion still says "otherwise a resumed Stuck row at an unchanged head checks again".
- Under command mode, a resumed row at the same head now ends as a Stuck without a judge. That might count as "checking again", but D4's text does not say so.
- This is borderline, at most LOW.

I made no further tool calls; my context already showed every file I needed.
