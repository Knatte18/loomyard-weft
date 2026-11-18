```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I found no wrong result, no broken caller and no added test that passes with its defect in place. I did find one comment-style defect and two points I couldn't confirm.

**Walk-throughs**

- **`appendedDecisionRecord`:**
  - A committed parent append, with the record absent from `Changed` and `Untracked` and the copy's entries and blocks kept, returns `rel`, so the exemption holds.
  - An uncommitted append returns `""`, so the record is named.
  - A malformed carry-over set in either record makes `CheckCarryOversNotLost` return a finding, so the record is named. That is fail-closed.
  - The only caller, `Changes` (`judgesnapshot.go:249`), passes the full `CheckoutState`, so the signature change holds.
- **`carryOverFindings` / `CheckCarryOversNotLost`:** the `case !inCurrent && !includeAdded: continue` sits after `!inNext`, so a dropped block is still reported. `CheckCarryOversKept` behaves as before.
- **`checkCase`:** removing the unkeyed counter and the refusal leaves `listing` still used.
- **Refusal in `decisionVerb`:**
  - It sits after the status, filing, plan and approval checks and before `date`, `beginAmendment` and `deps.append`, so a refusal writes nothing.
  - A read error returns the error text plus `decisionWayRetry`, which contains "nothing is appended", so the new read-failure row's want holds.
  - The `decisionDeps` literal at `decision.go:1908` (a `decisionReplaceVerb`-style literal holding only `recordPath` and `programName`) never reaches `judgeSessionOpen`, since the `decisionVerb` refusal is the only caller.
- **Wiring:** `driftJudgeSessionOpen` reads `driftengine.ReadScope`, whose `recordLock` creates the drift directory. That is harmless, and it returns found-false when no scope record exists.

**Added tests against their defects**

- `RangeOverFullyKeyedBaselineCase`: fails if the refusal returns.
- `DecisionEntryUncommittedIsNamed`: fails without the `Changed` / `Untracked` check, because the content then passes.
- `DecisionCarryOverBlockEditedIsNamed`: fails without `CheckCarryOversNotLost`, because `IdentityText` strips the blocks and the identities match.
- `DecisionEntryBesideAKeptCarryOverBlockIsNoChange`: I computed the identity texts by hand and they match, so it passes.
- The three fail-closed rows each fail if their guard is dropped:
  - The unset `DecisionRecord`.
  - The absent copy.
  - The copy cleared by the second snapshot, which `TakeJudgeSnapshot` removes via `os.Remove(recordCopy)`.
- `TestCheckCarryOversNotLost`: its added-block row fails if added blocks are reported.
- `TestDriftDeps_CarryTheDecisionRecord`: fails if the `DecisionRecord:` line is dropped.
- The two refusal rows, `PATTERN change while a judge session is open` and `JudgeSessionOpen_Refused`, assert the refusal text. They fail if the check is removed, since the call would then succeed.

**Findings**

1. **LOW**, changed text: `internal/discussionparser/replacecheck.go`, doc comments on `CheckCarryOversNotLost` and `carryOverFindings`.
   - Several lines run past 120 characters. The `CheckCarryOversNotLost` first line is about 122 characters after `// `.
   - The sentence "the findings, the sort and the malformed-marker handling are those of CheckCarryOversKept" defines the symbol in terms of another named symbol, which `code-quality`'s self-contained rule forbids.
   - Evidence: the edit text in my context. Suggested fix: state the behaviour directly and split the long lines.

2. **LOW**, unconfirmed (not a defect I could show): a run that halts Stuck at Drift-Verify leaves `judge_session.Open` set.
   - `--amends-plan` and PATTERN-change calls are then refused until a resume closes the session.
   - The refusal's way forward ("once Drift-Verify has ended its judge session") covers this. I'm noting it only as a behaviour the doc comment states loosely as "while the scope record's judge session is open".

**Gaps**

- No test shows that a plain `decision add` still succeeds while `judgeOpen` is true, which is the point of issue 145.
- The refusal rows rely on the existing harness's `inputRefusal` handling, which I did not read.

**Calls**

- One Bash call combining four greps and a read of `recordLock` and `decisionWayRetry`. I used it to find every `decisionDeps{` literal and every caller of `appendedDecisionRecord`, to see what `ReadScope` does on disk, and to check that `decisionWayRetry` contains "nothing is appended". My context lacked all of those.
