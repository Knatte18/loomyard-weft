```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes on concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- **`TakeJudgeSnapshot`, clearing the copy** (`judgesnapshot.go:159-162`): the copy is absent (`ErrNotExist`), so the error is ignored and it goes on. Another remove error returns a wrapped error. Correct.
- **`TakeJudgeSnapshot`, `DecisionRecord` empty**: no copy is written. Correct.
- **`TakeJudgeSnapshot`, record absent at snapshot time**: the `Stat` fails, so no copy is written. A record that appears later has no copy, gets no exemption and is named. This fails closed. Correct.
- **`TakeJudgeSnapshot`, `copyFile` fails after the old copy was removed**: the old `snapshot.json` stays without a copy, so the exemption is off. This fails closed. Correct.
- **`TakeJudgeSnapshot`, copy succeeds but `WriteJSON` fails**:
  - From `openJudgeSession`, no session mark opens and the next pass retakes the snapshot.
  - From `resumeJudgeSession`, the snapshot is already gone, so it is recorded as lost and retaken.
  - No session ever compares against the mismatched copy. Correct.
- **`recordsCheckoutPath`**: the path under `WorktreeRoot` maps to the same relative path under the records root. That matches how `loomRecordsDirs` anchors its paths (`driftwiring.go:290-293`). Correct.
- **`appendedDecisionRecord`, `DecisionRecord` empty, copy missing, or record deleted**: each returns empty. A deleted record is then named "was deleted". Correct.
- **`appendedDecisionRecord`, copy or record unreadable for a reason other than not-exist**: the error propagates through `Changes` to `changedSince`. Correct.
- **`appendedDecisionRecord`, one parent entry appended** (the `testRecord` → `testRecordEntry` row): the identity texts are equal and `CheckAddedEntriesSubset(now, before)` returns no finding, so the record is exempt. The argument order is right: it reports each entry of the copy with no match in the current record. Correct.
- **`appendedDecisionRecord`, Goal edited beside an entry**: the identity texts differ, so the record is named. Correct.
- **`appendedDecisionRecord`, earlier entry dropped or rewritten**: the subset check reports it, so the record is named. Correct.
- **`appendedDecisionRecord`, an existing carry-over block in `## Open risks` dropped or rewritten**: `IdentityText` strips every well-formed carry-over block from both texts, so the change is exempt and unseen. **Not correct** (finding 2).
- **`Changes`, `rel == appendedRecord` with `appendedRecord` empty**: no listed path is empty, so nothing is wrongly exempted. Correct.
- **`checkCase`, all lines keyed plus a `lines` entry**: refused. Correct.
- **`checkCase`, mixed case** (keyed and unkeyed lines): accepted. Correct.
- **`checkCase`, empty baseline case**: both counts are 0, so it is accepted. Correct.
- **`checkCase`, case only under `added`**: the listing is the zero value, so it is accepted. Correct.
- **`checkCase`, all keyed but no `lines` entry**: the loop never runs. Correct.
- **`ParseExpectation` refusal reached through `sealedAndAmended`** (`check.go:243-247`), on an expectation sealed before this change: every drift check fails hard. **Not correct** (finding 1).

**Findings**

1. **MEDIUM, design.** Location: `internal/driftengine/expectation.go:297-300`, as reached by `check.go:244-246`.
   - Summary: an expectation sealed before this change that holds a range over a fully keyed case now fails every drift check with "the sealed expectation does not parse", and nothing names a way forward. Before the change, the same file gave uncovered cells, which `lym drift amend --entries` with `keys` entries could cover.
   - Why amend cannot fix it: an amend adds entries but leaves the sealed bytes untouched, and `sealedAndAmended` re-parses those sealed bytes on every check.
   - Effect: this is the issue #141 run's own state. An in-flight run in it loses the workaround the issue describes, and gets a bare error instead of a refusal that names a way forward. A discard amendment exists (`seal.go:171`), but the error does not mention it.
   - Fix options: apply the new check only where `expected.yaml` is first parsed (`expect.go:349`) and in amend (`amend.go:156`), not when the sealed file is re-read; or have the error name a way forward.
   - Evidence: `check.go:187`, `check.go:243-247`, `expect.go:349`, `amend.go:156`.

2. **MEDIUM, design.** Location: `internal/driftengine/judgesnapshot.go:331-332`.
   - Summary: the exemption compares `IdentityText`, which strips every well-formed carry-over block from both the copy and the current record. A judge session that rewrites or drops an existing carry-over block is therefore exempt and unseen.
   - Why the docs miss it: the bound in `doc.go` names only a block "the judge session writes", and the commit message claims "every earlier entry kept".
   - Fix: add `discussionparser.CheckCarryOversKept(before, now)`, which exists for this comparison (`replacecheck.go:24`), so a changed or dropped block is named and only added blocks pass.
   - Evidence: `identity.go:11-23`, `identity.go:36-51`, `judgesnapshot.go:308`.

**Tool calls**

- `check.go` 225-275, `expect.go` 335-360, `amend.go` 140-170: my context did not show who re-parses the sealed expectation.
- grep for `sealedAndAmended`, the amendment kinds and any reseal path: to check whether a way forward exists for an unparseable sealed file.
- `check.go` 175-200 and grep for the discard and restore kinds: to see how the parse error propagates and what the discard amendment does.
