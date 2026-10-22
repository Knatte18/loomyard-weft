---
verdict: APPROVED
---

## Scope against the board entry

The record holds no `Added after Discussion` entry, so the measure is the board body of `decision-add-notifies`.

- Operator decision 1 and 2: `lyx loom decision add` records as before and, after the commit, posts one creel line to each live LLM run of the target worktree (`internal/loomcli/decision.go`, `notifyLiveRuns`); no new verb, alias or queue is added, matching the Dropped section.
- Operator decision 3: with no live run nothing is sent, and the file doc and `docs/overview.md` say a later producer reads the record and PR-Gate judges against it.
- The tackler is skipped as a receiver (`decisionNoticeReceiver`), as the entry asks.
- A stop stays `lyx batten pause`; the diff touches no stop path.
- Open, the flag: settled as no flag (D5), recorded as an auto-mode assumption.
- Open, the orch role: the Fixing a run section (`orch-template-role.md:109`) and the board policy (`orch-template-role.md:161`) both drop the separate message step.

The darn writer stencil's change names the notice line as a trigger to re-read the record; it is the receiving side of the same behaviour and within the entry's goal.

## Carried-over risks

Darn-Review round 1 carried over `notice-signing-split-untested` and `orch-board-policy-keeps-message-step`.
The fixer report and the diff show both fixed: `TestPostDecisionNotice_SignsByCaller` covers the signing split, and line 161 of the orch role stencil now has the verb alone record a darn run's decision.
Neither names an unrecorded parent correction.
The record's two open risks, a webster fork in its own worktree going untold and a busy agent getting the line late, are accepted by the entry's own fallback to PR-Gate.

## Overlaps

No parent commit is listed.

- `gate-tier-verbs` (row 0): both change `docs/overview.md`, at different module paragraphs (gate versus loom's `decision add`). Text-only; unknown order, since the peer's row is 0.
- `machine-handover` (row 5): its decision record names `docs/overview.md` among its doc changes; no changed paths yet. Text-only; this run (row 21) lands first.
- `quarry-csharp` (row 5) and `darn-verify-fixes` (row 4): no shared file or behaviour.

The note `creel-agents` will later move the orch role's `SendMessage` lines to creel; it is a note that has not started and does not contradict this run.

## Superseded

None: no other board entry is made obsolete by this run.
