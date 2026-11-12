```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. I traced concrete inputs through each function and branch the diff adds or changes.

**List (function or branch: case run: verdict)**
- `roundSeed`: round-1 focus file missing: returns an error, the caller logs it and returns 0 (no ask). Correct.
- `roundParentAsk`, ask held and `open`: posts nothing and returns the number, whatever the key reads. Correct.
- `roundParentAsk`, ask held and `pending-post`: re-posts to the stored strand. Correct.
- `roundParentAsk`, ask held as `not-sent` or `no-parent`: returns the number, so the consolidation part fills (D7). Correct.
- `roundParentAsk`, key off with no held ask: returns 0. Correct.
- `parentAskDue`, empty record: asks. Correct.
- `parentAskDue`, latest ask has an older seed or no seed (legacy record): asks. Correct.
- `parentAskDue`, history unreadable: warns and falls through to the round-before and `not-sent` rules. Correct.
- `parentAskDue`, Discussion-Write after the latest ask: asks. Correct.
- `parentAskDue`, latest ask `done` in round-1: reads the report and asks only if it holds findings. A report that cannot be read asks again. Correct.
- `parentAskDue`, `not-sent` in an earlier round: asks. Correct.
- `parentAskDue`, `timeout` or `no-parent`: asks nothing. Correct.
- `sendParentAsk`, `Wait` returns a skip reason: records `not-sent` with no strand and posts nothing. Correct.
- `sendParentAsk`, `Resolve` returns an error or recorded=false: records `no-parent`, posts nothing. Correct.
- `sendParentAsk`, normal case: records `pending-post` with deadline and strand, then posts. Correct.
- `sendParentAsk`, `RecordParentAsk` fails: returns 0 and the round goes on. Correct.
- `postParentAsk`, post error: sets `no-parent` through `EndParentAsk`. Correct. A failed `EndParentAsk` is logged and the ask stays `pending-post`, so the next row re-posts, which D3 accepts.
- `postParentAsk`, `MarkParentAskPosted` fails or the parent answers first: the state stays `pending-post` or `done`, and `MarkParentAskPosted` no-ops on anything but `pending-post`. Correct.
- `settleRoundParentAsk` with number 0 or no seat: no-op. Correct.
- `settleRoundParentAsk`, ask `open`: set `timeout` under the lock, so a later answer is refused. Correct.
- `settleRoundParentAsk`, ask `done`: copies once, and skips when the copy exists. A report that cannot be read aborts the update and is logged. Correct.
- `heldParentAsk` on the hand-on path: uses the same seed lookup and settles a restart after acceptance. Correct.
- `EndParentAsk`, `MarkParentAskPosted`, `ExpireParentAsk`, `AnswerParentAsk`: the `errParentAskUnchanged` aborts write nothing. The past-deadline branch sets `timeout` and the refusal is returned. Correct.
- `parentCoverageGate`, ask not found or not `done`: passes. Correct.
- `parentCoverageGate`, `done` with no findings: passes. Correct.
- `parentCoverageGate`, review file unreadable or unparseable: fails with findings, re-prompting the chair. Correct.
- `parentCoverageGate`, ids uncovered: fails naming them. `PassOnCap` defaults to false (`shuttleengine/gate.go:76`), so the entry never passes on its cap. Correct.
- `parentCoverageGate`, report unreadable: gate error. Correct.
- `uncoveredParentIDs`: counts only `Origin == "parent"` findings, in report order. Correct.
- `withParentCoverage`: copies the gate slice and returns the table unchanged for 0. It is used by both `Review` and `probeTable`, so a probe or resume carries the same gate. Correct.
- `concludeReview`: falls back to `Findings` when the gate gives no `Reason`. Correct.
- `askParentSetting`: error or non-bool is logged and read as false. Correct.
- `latestCallAt`: absent file or no history returns the zero time; a bad stamp returns an error. Correct.
- `PostAsProgram` and `postAsProgram`: a client or post error comes back wrapped with the line, and the strand claim is ignored. Correct.
- Wiring: the parent seat lands on `PerchDiscussion` only. `Slug` is `seedSlug(WorktreeName)`, which is the worktree name. Correct.

**Findings**

1. LOW, `design`, `internal/shedadapters/parentseat.go:88-90`. A restarted row re-posts a `pending-post` ask without checking its deadline. If the row restarts after the deadline, the parent gets a request it can only answer late: the answer is refused and the ask set `timeout`. Fix: when `ask.Deadline` has passed, end the ask `timeout` (or `no-parent`) through `EndParentAsk` instead of posting. D3 accepts a second mail but does not cover a stale one, so this is a judgment call.

2. NIT, `consistency`, `internal/loomcli/parentseatwiring.go:1` and `internal/loomcli/wiring.go` (the `awaitCap: chairAwaitCap(...)` line). The file header says every closure reads on a call, never at wire time. `awaitCap` is resolved at wire time, as `segmentAssistBounds` does for the assistant. This is harmless; the header just overstates it.

I found no other defect on a failure path.

**Tool calls**
1. `grep` for `Attempts` in `internal/shuttleengine/*.go` and for `seedSlug`: my context lacked whether a gate entry can pass on its cap (`PassOnCap` defaults to false) and what `seedSlug` returns (the worktree name).
