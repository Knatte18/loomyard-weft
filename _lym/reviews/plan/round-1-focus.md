---
round: 1
focus:
  - "Fidelity, `_lym/discussion/decision-record.md`: walk D1–D10 and every Constraint and settle which card carries each clause. Likeliest gaps: D3's merge-removes-and-upserts rule (card 1), D5's lost report for a written record file whose entry origin lacks (card 4), D8's failed-commit-found-landed path (card 8), D10's `--record` refusals (card 5). Also check no card carries scope the record does not license."
  - "Test economy: cards 1 and 2 both plan new tests around `StoreDecisionRecord` and `DecisionRecord`; cards 6 and 7 plan `TestPastDiscussionApproval` and `TestDiscussionApprovalRowsCarryApproveSeam` over the same `discussionApprovalRow`; cards 8 and 9 plan `TestDecisionVerb_CopiesRecordToBoard` and `TestDecisionStoreVerb` over the same seams. For each pair, settle whether a named existing or sibling test already covers a row, and name that test."
  - "Reachability of prescribed refusals (`PATTERN-refusal-way-forward`): for each refusal card 1, 3, 5, 7, 8 and 9 prescribe, settle whether a named input and a named test reach it, and whether an upstream check rejects the input first. Look especially at card 9's three refused-session shapes, the `ErrRecordEntryDone` branch swallowed in card 7's `approveDiscussion`, and card 3's `design_prefix` rows against `LoadConfig`'s earlier checks."
  - "Card order and dependence: cards 8 and 9 use card 7's `storeDecisionRecord`, card 6's accessors and card 8's `decisionDeps` fields; card 9 edits `decisionFake` fields card 8 also edits. Settle whether any card needs something a later card creates, and whether card 2 and card 4 need anything from cards 3 to 9."
  - "Attack-surface question: card 1 exempts `StoreDecisionRecord` from the run lock, card 7 adds a new `approve_seam` value, card 9 adds a verb with three admitted callers. Settle whether each card's `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Member glyphs, re-sign arrow and `Extends` typing: check each `Edit` card's targets against its `Intent` where a function's parameters or results change (card 2 `boardCriticalSection`, card 4 `applyDesignFile`, card 8 `afterFailedCommit`, `decisionVerb`, `preRun`), and whether `Extends` targets (`plugins/ly/skills/board/SKILL.md`, the `doc.go` files) are only added to, never rewritten, since cards 2, 3 and 4 also describe doc.go edits under `Edit`."
  - "Board-entry rules: settle that no card edits this task's own board entry body or scope, files a task or promotes a note. Check card 7's `TestApproveDiscussion_Real` and card 5's `TestCLI` step, which upsert entries, against the run's board lock."
---

Seed focus for a plan of nine cards over the board engine, loomrecipe, shedrecipe and loomcli.
Entries name where to look and what to settle; none pre-states a verdict.
The mechanical checks, missing dependency fields and `ImpactSummary` on non-Edit types are left to the gate and the Do-not-flag list.
