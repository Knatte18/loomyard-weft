```yaml
lens: code-correctness
outcome: reported
```

Task: lens `code-correctness`, running each function the diff adds or changes on a concrete input.

**Per-function list**

- `ParentAskState.Ended`: I ran an unknown state `"x"`. It counts as ended, so it is never moved. Correct.
- `RecordParentAsk`: I ran an empty record, which gives Number 1, and a record whose latest is 3, which gives 4. Correct. A failed update returns the zero ask and a wrapped error.
- `MarkParentAskPosted`: I ran pending-post, an ended ask and an absent number. Only pending-post moves. The other two return `errParentAskUnchanged`, which writes nothing and is swallowed on purpose. Correct.
- `AnswerParentAsk`: correct on the paths below, with one finding.
  - **No ask:** returns the no-ask refusal and writes nothing.
  - **Number is not the latest:** returns the not-latest refusal and writes nothing.
  - **Latest already ended, including an unknown state:** returns the ended refusal and writes nothing.
  - **At the deadline (`now == Deadline`):** sets timeout, writes the ask record, writes no report, and returns the refusal. Correct.
  - **Before the deadline:** writes the report, then the record. Correct.
  - **Racing an expiry:** both take the one write lock, so exactly one end state results. Correct.
  - **Failed record write after the report was written:** see F1.
  - **Failed record write on the past-deadline path:** the caller gets the refusal because `refusal != nil` is checked before `err`, even though the timeout was not persisted. See F2.
- `writeParentReport`: I ran empty findings and two findings. The result is `satisfied: true` with `findings: []`, or ids P1 and P2 in file order. Severity and class are kept. Correct.
- `ExpireParentAsk`: correct on the cases below.
  - **No ask:** found false and nothing written.
  - **Active ask before its deadline:** unchanged.
  - **Active ask at the deadline:** set to timeout.
  - **Ended ask:** unchanged, and the ask is returned with found true.
  - **Failed write:** returns the zero ask and found false with the error.
- `readParentFindings`: correct on every case below.
  - **Empty file:** the decoder gives EOF, so it reports "holds no finding".
  - **`findings: []`:** same refusal.
  - **Item with no change, or a whitespace-only change:** refused, naming the item.
  - **Severity `HIGH`, or lowercase `blocking`:** refused.
  - **Class `style`:** refused.
  - **Empty severity or class:** treated as unset.
  - **Unreadable file:** refused with the file-shape way forward.
  - **Ids and unknown keys:** the engine assigns ids in file order. Unknown keys are ignored.
- `parentAnswerCallerRefusal`: correct on every case below.
  - **Unnamed caller:** admitted.
  - **Named caller equal to the ask's strand:** admitted even when the parent lookup failed.
  - **Named caller equal to the parent name:** admitted.
  - **Any other named caller:** refused. The refusal names the lookup failure when the parent could not be resolved.
  - **`--ask` naming no ask (zero value):** falls through to the parent check.
- `parentReviewWayDecision` and `parentAnswerRefusalText`: I ran each refusal kind.
  - **No-ask:** names the decision verb.
  - **Not-latest with the latest open or pending-post:** names the latest number and says to review the record again.
  - **Not-latest with the latest ended:** names the decision verb.
  - **Past-deadline and ended:** name the decision verb.
  - **Who runs it:** the operator's shell when the parent lookup failed or the name is empty, otherwise the parent orch. Correct.
- `parentReviewAnswerVerb`:
  - **`--ask 0`:** refused.
  - **Both forms or neither:** refused, via `satisfied == (file != "")`.
  - **Output:** the ok envelope carries slug, ask, state, report and findings. Correct.
  - **Order of checks:** the findings file is read before the caller guard, so a refused caller can see file-shape errors first. This is harmless.
- `parentReviewAwaitVerb`: correct on every case below.
  - **Cap 0:** one read, then not ready.
  - **Deadline inside the wait:** `Expire` runs before the `remaining` check, so an end that lands exactly on the cap is not missed.
  - **Sleep:** capped at `min(1s, remaining)`, so the loop wakes at most once a second.
  - **Cancelled context:** the sleep errors, so it refuses and does not spin.
  - **No ask:** refused.
  - **Ready answer:** carries the reason for not-sent and the report for done.
- `preRun`:
  - **Group command:** skipped.
  - **Operator shell with an empty strand:** `KeepWarm` falls back to `DefaultAwaitCap`. Correct.
- `chairTimeout` and `CapAssistTimeout`: the arithmetic is identical to before, so behaviour is unchanged.
- `CapParentReviewWait`:
  - **Room ≤ 0:** returns zero and a reason naming both keys.
  - **Want above the room:** returns the room, so the slack equals the margin.
  - **Want within the room:** returns the want.
  - **Zero review timeout:** uses `runTimeout`.
- `LoadConfig` validation: a value of 0 or below is refused, naming the key.
- State writes: `internal/state` keeps unknown keys on write, so the tolerant-record claim holds. Go 1.26 supports `omitzero`.

**Findings**

- **F1 — LOW, design.** `internal/burlerengine/parentreview.go:229-235`. `AnswerParentAsk` writes the report inside the update, before the record is written. If the record write then fails, `UpdateJSON` returns an error, but the report stays on disk while the ask remains `open`. The focus asks that no report survive a failed answer. The orphan is overwritten by a later successful answer, so it does no lasting harm. Fix: write the report only after the update commits, or remove it on a record-write failure.
- **F2 — LOW, design.** `internal/burlerengine/parentreview.go:224-228, 237-239`. On the past-deadline path the mutate returns `nil`, so the record write can fail. The failure is swallowed because the refusal is returned first, so the caller is told the ask timed out when it was not persisted. A later await or answer re-derives the timeout, so nothing lasting breaks. Fix: when `err != nil`, return the error ahead of the refusal.

No other defects.

Tool calls used: 1.
- I read `writeJSONUnlocked` in `internal/state/state.go` and the Go version in `go.mod`. My context lacked whether `UpdateJSON` keeps unknown keys and whether `omitzero` is supported.
