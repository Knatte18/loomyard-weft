```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per-behaviour list**

burlerengine (`parentreview.go`)
- Ask numbering from 1 in recording order: pinned by `TestParentAsks_RecordAndMoveForward` (`Number == i+1` loop).
- `MarkParentAskPosted` opens only `pending-post`, and leaves an ended ask alone: pinned by the first two rows of the same test. No row for a number the run holds no ask for, or for a non-latest number.
- `ExpireParentAsk` sets timeout only at or past the deadline, never moves a done ask, and writes nothing for an empty run: pinned by the expiry rows of the same test.
- `Ended()` counts an unknown state as ended, so an ask in such a state is never moved: **none.** No test records an ask with an unknown state.
- `AnswerParentAsk`:
  - A satisfied report, P1/P2 ids with severity and class kept, and each refusal kind (no-ask, not-latest, ended, past-deadline) with no report written: pinned by `TestAnswerParentAsk`.
  - An answer racing the deadline leaves exactly one end state: `TestAnswerParentAsk_RacingTheDeadline`.
- Report written inside the locked update surviving a failed state write: none. It is hard to fake, and it is not an acceptance criterion.
- Preserving unknown keys in the ask record (the summary claims it): none in the diff. `PATTERN-tolerant-records` has its own test.

loomcli (`parentreview.go`)
- `readParentFindings`:
  - Refusals for an empty list, no `change`, a bad severity and a bad class: pinned by rows of `TestParentReviewAnswerVerb`.
  - An empty file (the `io.EOF` path), malformed YAML, an unreadable path, and a whitespace-only `change`: none.
  - The given severity and class landing in the report: none at this level. The verb test asserts only `findings: 2` and the state. The burler test builds `ParentFinding` directly, so dropping the mapping in `readParentFindings` fails nothing.
- `parentAnswerCallerRefusal`:
  - The operator's unnamed shell is the fixture default and is covered by the answering rows.
  - The ask's strand answering a `pending-post` ask after the parent resolved elsewhere, and a stranger refused: pinned.
  - A named caller equal to the resolved parent orch being admitted: **none.** No row sets `strandName` to `parentReviewParent`.
  - A named caller refused when the parent cannot be resolved (the `parentErr` branch): none.
- `parentReviewWayDecision` and `parentAnswerRefusalText`:
  - No ask (parent named, and operator's shell when resolution fails): pinned.
  - Not-latest while the latest is `open`, naming the latest number: pinned.
  - Not-latest with the latest ended, plus already done, timeout, no-parent and not-sent: pinned.
  - Past deadline setting timeout and naming `decision add` with the parent orch: pinned.
- Flag refusals (missing `--ask`, both forms, neither): pinned.
- Answer envelope (`slug`, `ask`, `state`, `report`, `findings`): pinned.
- `parentReviewAwaitVerb`:
  - Ready on an answer during the wait with the report path, ready on an already-timed-out ask, and ready on a not-sent ask with its reason: pinned.
  - The deadline reached during the wait setting timeout, not ready when the cap runs out, the sleep never above one second, and refusal of a run with no ask: pinned.
  - The context-cancel path (`the wait was cut short`): none.
  - A `no-parent` ask answering ready: no row, but it shares the ended branch.
- `inReviewGroup` skipping `parent-review`: **none.** `cli_test.go:40` loops over `{"circling", decisionGroup, peersGroup}` and does not include `parentReviewGroup`. Without the skip, the loom pre-run would arm the verbs and read the slug as a run-id.
- `preRun` wiring (strand from the environment, `awaitCap` from `KeepWarm`, files, parent): none. It needs a hub fixture, so this is not raised as a finding.

loomengine
- `CapParentReviewWait`: pinned by `TestCapParentReviewWait`, covering fits, capped, zero review timeout taking the run timeout, room spent to exactly zero, and allowance past the chair. The Warn log is not asserted; that is acceptable.
- `chairTimeout` extraction leaving `CapAssistTimeout` unchanged: pinned by the existing `TestCapAssistTimeout`.
- Config defaults for both keys: pinned by `templateDefaults` in `config_test.go`.
- Overrides: pinned by the new override row.
- Refusal of `0` and a negative value naming the key: pinned by the two new `TestLoadConfig_Refuses` rows.
- Category-none registration: none directly, though the `ConfigSpec` checks likely cover it. The template comment saying "settable per run through `run_settings`" is static prose and exempt.

**Findings**

1. **MEDIUM, consistency**
   - Location: `internal/loomcli/cli_test.go:40`.
   - Summary: the loom pre-run's skip of the `parent-review` group, added in `inReviewGroup`, has no test row.
   - Evidence: the table lists only `circling`, `decisionGroup` and `peersGroup`.
   - Fix: add `parentReviewGroup` to that list.

2. **MEDIUM, design**
   - Location: `internal/loomcli/parentreview_test.go:75-81`.
   - Summary: no test checks that a findings file's severity, class and change reach the report. This is the D4 acceptance criterion that each finding keeps its given severity and class.
   - Evidence: the row asserts only the `findings: 2` count and the `done` state. The burler test feeds structs directly.
   - Fix: read the written report in that row and compare it with the file's items.

3. **MEDIUM, design**
   - Location: `internal/loomcli/parentreview_test.go:96-103`.
   - Summary: the guard's admission of a named caller equal to the resolved parent orch is untested, as is the refusal of a named caller when the parent cannot be resolved.
   - Evidence: the answering rows run as the unnamed shell or as the ask's strand. The only refused caller is `ly:task-a:darn`.
   - Fix: add one row with `strandName = parentReviewParent` that expects `done`. Add one row with a failing `parent` and a stranger caller, expecting the could-not-be-resolved refusal.

4. **LOW, design**
   - Location: `internal/loomcli/parentreview_test.go:92`.
   - Summary: the empty findings file (the `io.EOF` branch), malformed YAML, an unreadable path and a whitespace-only `change` have no rows. Each is a distinct refusal naming the file's shape.
   - Fix: add rows for an empty file, a file holding `[`, a missing path and `change: "  "`.

5. **LOW, design**
   - Location: `internal/burlerengine/parentreview_test.go:52`.
   - Summary: the unknown-state rule is untested. It is the focus's "never moved" invariant.
   - Fix: add a row in `TestAnswerParentAsk` (and one for `ExpireParentAsk`) with `State: "future-state"` and a past deadline. Assert the state is unchanged and the answer is refused as ended.

6. **LOW, design**
   - Location: `internal/loomcli/parentreview_test.go:191`.
   - Summary: the await's `the wait was cut short` refusal, from a `sleep` error or a cancelled context, has no row.
   - Fix: add a row whose `sleep` returns `context.Canceled`, and assert the refusal text and `parentReviewWayAwaitRetry`.

7. **LOW, consistency (test economy)**
   - Location: `internal/burlerengine/parentreview_test.go:240`.
   - Summary: `TestAnswerParentAsk_RacingTheDeadline` is listed as contained. It pins what the covering tests cannot see: concurrent exclusion of an answer and an expiry under the lock, which the sequential tests never exercise. It should be kept, not folded or deleted.
   - Fix: add `//testtiming:keep` directly above its `func Test…` line, with a reason such as "pins that a concurrent answer and expiry leave exactly one end state under the lock".

**Tool calls**
1. `grep` for `inReviewGroup`, `peersGroup` and `parentReviewGroup` across `*_test.go`. My context held no test for the pre-run skip, and this showed `cli_test.go:40` omits the new group.
