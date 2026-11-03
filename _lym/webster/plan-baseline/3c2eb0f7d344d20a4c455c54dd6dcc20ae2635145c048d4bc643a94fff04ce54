# Card 9 — decision-store-verb

**Create:**
- `internal/loomcli#decisionStoreVerb`
- `internal/loomcli#TestDecisionStoreVerb`

**Edit:**
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#decisionFake`
- `internal/loomcli#decisionFake.deps`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomrecipe#DiscussionApprovalRow`
- `internal/loomcli#newDecisionFake`
- `internal/loomcli#newDarnDecisionFake`
- `internal/loomrecipe#PastDiscussionApproval`
- `internal/boardengine#ErrRecordEntryDone`

**Intent:**
Add `lym loom decision store [<slug>]`, the one idempotent way to re-copy a run's committed record onto its board entry (D9).

**The verb.**
`decisionCmd` gains a `store` subcommand.
- `Use` is `store [<slug>]` and `Args` is `cobra.MaximumNArgs(1)`.
- It carries the operator audience annotation and a `Short`, so the generated `lym help index` lists it.
- It takes no flags and runs `decisionStoreVerb` over the deps the group's shared `preRun` builds.
- The group's `Long`, which says `"decision add" is the only verb`, names both verbs.
- The file header's opening line, which says the subtree has one verb, names both.

**Callers.**
`decisionStoreVerb` admits three callers and refuses every other named session:
- the operator's own unnamed shell (`strandName` empty);
- the run's own arbiter strand, by `deps.arbiterStrand` alone with no label;
- the run's parent orch, judged by `deps.parent` as `add` judges it.

A refused session gets, as `add`'s refusals are shaped:
- with a named parent: `the caller %q is a task session; way forward: ask %s to run "lym loom decision store"; the operator's own unnamed shell may always run it`;
- with a parent whose name is empty: `the caller %q is a task session; way forward: the operator's own unnamed shell may always run "lym loom decision store"`;
- with a parent that cannot be resolved: `the caller %q is a named session; way forward: the run's parent could not be resolved (%s); the operator's own unnamed shell may still run the verb`.

**Refusals.**
It reads the run status through `deps.readStatus`.
A new `decisionDeps` field, `approvalRow func() bool`, filled in `preRun` from `loomrecipe.DiscussionApprovalRow(recipe)` for every call, reports whether the recipe has a discussion approval row.
It drops the accessor's error under the same comment as card 8's `pastApproval`: `resolveReviewTarget`'s `recipeUnknown` refusal has already refused every recipe the accessor cannot parse.
`deps.pastApproval(currentRow)` from card 8 judges whether the run has passed it.
- No approval row: `the run's recipe has no Discussion approval, so its record is never stored on the board; way forward: read the record in the run's worktree at <recordPath>`.
- Not past (`pastApproval` false): `the run has not passed its Discussion approval, which stores the record itself; way forward: let the run settle Discussion-Review, by "lym loom resume" or, when it waits on an escalation, by answering it with the "lym loom circling" verb its escalation names, then re-run this verb if "lym loom status" shows it past Discussion-Bouncer without a stored record`.
- A status read error is refused with the error and `way forward: re-run "lym loom decision store"`.

**The copy.**
Past the approval, it copies the committed record onto the board entry.
The copy reads `deps.committed()` and passes the bytes to `deps.storeRecord`, the seams card 8 added. It does not go through `copyRecordToBoard`, because the verb needs the error, through `errors.Is`, rather than the envelope text.
- A committed record HEAD does not hold is refused: `the run has no committed decision record; way forward: run "lym fabric commit" to commit it, then re-run "lym loom decision store"; a session lym refuses "lym fabric commit" from reports status: FAILED and the orch runs it`.
- A done entry, an error matching `ErrRecordEntryDone`, is refused: `the entry has landed and keeps the record it last stored; nothing to store`.
- Any other copy failure is refused with the error and the re-run of the verb.
- Success prints `{"slug":…,"record":…,"stored":true}`.

It changes nothing in the run: no commit, no routing and no notice.
Bound: it stores only the run's own committed record, and only past the approval as D8 judges it. A goto past Discussion-Bouncer by the parent counts as past, which the parent's own goto decided.

`loomcli/doc.go` gains the verb in one sentence.

**Tests.**
`TestDecisionStoreVerb` (tier 0) is a table over `newDecisionFake` and `newDarnDecisionFake`, with `decisionFake` gaining `hasApprovalRow bool`, filled into `deps()` as `approvalRow`, and using card 8's `pastRows`, `storeErr` and `stored`.
Rows that reach the copy seed `committedFound` and `committedCopy`, which `newDecisionFake` leaves unset.
- Past the approval, the committed record is stored and the verb exits 0.
- A darn run is refused naming the record path.
- A run at `Discussion-Burler-Fix`, inside Discussion-Review, is refused naming `lym loom resume` and `lym loom circling`.
- A done entry is refused, naming that it keeps its last stored record.
- A committed record HEAD does not hold is refused naming `lym fabric commit`.
- A status read error is refused naming the re-run.
- A named task session that is neither parent nor arbiter is refused naming the parent; with an empty parent name it names only the unnamed shell; with a parent error it names the error.
- The arbiter strand passes.
- The parent passes.
- The unnamed shell passes.
- A store failure is refused naming the re-run.

**ImpactSummary:** One new operator verb in loomcli's `decision` group, reusing `add`'s pre-run, caller seams and card 8's `committed` and `storeRecord` seams. It makes no commit and does no routing. The generated CLI index picks it up with no hand-written doc.
