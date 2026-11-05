Task: for each finding fixed as proposed (F1, F3, F4's Drift-Expect part, F5, F6), build a case where the proposed change is wrong or breaks something.

**Findings**

1. **MEDIUM** — `internal/driftengine/expect.go`, new `case rec.Mismatch:` in the `pass` switch. It halts even when no write would follow.
   - `doneUnsealed` writes only when `!rec.Scope.Unsealed`, and the sealed and diff-recorded paths may write nothing.
   - Case: a resume re-enters Drift-Expect with the scope off or deferred and `Unsealed` already true, with a records mismatch. Before the fix this was Done with no write; now it is a Stuck.
   - With scope off, the halt's way forward is a dead end. `Amend` refuses on `ScopeOff` ("nothing needs amending; way forward: lym loom resume"), and resume halts again.
   - The `amendRows` change does not help here. The fix text said to turn the write-refusal into a halt, but the case sits before the switch's scope-off and deferred cases and fires whether or not a write follows.
   - Evidence: the `switch` order in `pass`, the `doneUnsealed` guard, and `Amend`'s `ScopeOff` refusal in `amend.go`.

2. **LOW** — `internal/driftengine/amend.go`, `expectRow` added to `amendRows`. It widens admission past the mismatch case.
   - `--discard` and `--entries` are now admitted at any blocked Drift-Expect halt, for example the "code moved" or "never counted" halts.
   - A `--discard` recorded before the seal makes the check skip the seal check once the expectation is sealed, so the expect agent's file is silently ignored.
   - Evidence: `Amend` imposes no row-specific form restriction, and `CheckSeal` returns an empty expectation on a discard amendment (`doc.go`).

3. **LOW** — `internal/driftengine/expect.go`, `runAgent`'s close `SessionMark{OpenedAt: openedAt}`. It does not enforce "ended normally".
   - The mark closes whenever `ctx.Err() == nil`, including when `agent.Call` returned an error, such as a spawn that never started the agent.
   - Case: a first pass opens the mark and the spawn errors. The mark is now closed with `OpenedAt` set. A resume with a valid pre-existing `expected.yaml` then seals with no agent session, which is the defect F5 described.
   - Evidence: `err = errors.Join(err, p.setSession(...))` runs for any `err` when the context is live; `pass` tests only `OpenedAt != ""`.
   - Records written before this change also have zero marks after a normal end, so a resume of such a run spawns the agent once more. This is benign.

F1, F3 and F6: no finding.
- F1: the gate failures it adds are all clearable by code, or fall to the parent when attempts are exhausted.
- F3: an empty key group falls through to the next pattern or the unkeyed branch, and nothing downstream assumes a keyed line.
- F6 is a pure rename.

Tool calls: none. The context already held every file the lens needed.
