---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:145
    summary: D7 premise is false, the plan's integration verify sits in the overview's verify section below the Card Index, so a hash over the framing above the index leaves verify, Shared Decisions, Rename mechanic and frontmatter unguarded while the Bound claims otherwise
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:151
    summary: D7 Bound and the acceptance line say a begun card's index line cannot be touched, but the cited card-set rule compares only NN-slug ids, so an edit to a begun card's index summary passes and no mechanism is stated
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:90
    summary: D3 defers the tree a reset to start leaves to D4's per-mode list, but D4 lists only the moving reset, so the pair-mode archive-only path (no dirt refusal, nothing discarded) has no stated tree
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:127
    summary: D6 does not say what a second amendment of a card already holding a rendered true entry does, so a mid-recovery re-edit can be cleared by the recovery's done record while the code follows the older text
---

### [BLOCKING] D7 hashes the wrong region and misstates what the overview framing carries

**Location:** `_lyx/discussion/decision-record.md` D7 (lines 143-154), Scope "Out" (line 31), Acceptance (line 275).

**Issue:**
D7 accepts an overview change "confined to its `## Card Index` section" and checks it with a second hash "over the framing alone", on the premise that the framing "carries the plan's integration verify".
That premise is false.
`planparser.splitFraming` (`internal/planparser/parse.go`) takes the framing as the prose between the title heading and `## Card Index`, and ends the index at the next `## ` heading.
`contracts/specs/loom-plan-spec.md` (lines 66 and 476) puts the integration verify in the plan-level `## verify:` section, which follows the Card Index, beside `## Shared Decisions` and `## Rename mechanic`.
The overview's frontmatter (`root`, `language`, `first_card`) is outside both regions.
With a framing-only hash, rebaseline would accept an edit to `## verify:`, to Shared Decisions, to Rename mechanic or to the frontmatter, because none of them is the framing and none is checked by the card-set rule.
Today all of these are pinned, since rebaseline refuses every overview change (`rebaseline.go`, the `planOverviewFile` branch).
So D7 downgrades a guard, and its Bound ("it cannot touch the framing or a begun card's line") states a narrower exposure than the mechanism leaves.
Scope "Out" ("Amending `00-overview.md` mid-run outside its Card Index") says the opposite of what the hash enforces, so the artifact also disagrees with itself.
The verify section is what the verify gate and landing run; letting it change mid-run without a restamp of anything but a hash is the case the old absolute refusal existed for.

**Fix:**
Define the second hash over the whole overview file with the Card Index section's lines removed (everything except the lines between the `## Card Index` heading and the next `## ` heading), so frontmatter, framing, `## verify:`, `## Shared Decisions` and `## Rename mechanic` stay pinned.
Correct D7's rationale and the "framing" wording to match, state the Bound in those terms, and make the acceptance line name the verify section and the frontmatter as refused changes, not only "the overview's framing".

### [MEDIUM] "A begun card's index line cannot change" has no mechanism

**Location:** `_lyx/discussion/decision-record.md` D7 Bound (line 151) and Acceptance (line 275).

**Issue:**
D7 holds the index change "to rebaseline's existing card-set rule", and the Bound and acceptance then say a begun card's index line is untouchable.
The existing rule (`rebaselineBatches`, `batchCardIDs` in `internal/websterengine/state.go`) compares only `NN-slug` ids against the recorded partition.
An edit to a begun card's one-line summary, which `render.go` puts into the batch's prompt, keeps its id and passes.
A plan writer implementing "existing rule plus framing hash" cannot make the acceptance line true, and a test pinned to it would fail.

**Fix:**
Either narrow the Bound and the acceptance line to "a begun card's number and slug" and say the summary text is free, or name the mechanism that pins begun lines (for example the second hash covering the index lines of begun cards).

### [LOW] D3 refers to D4 for a case D4 does not state

**Location:** `_lyx/discussion/decision-record.md` D3 Bound (line 90), D4 Bound (lines 104-108).

**Issue:**
D3 says the tree a following run starts from "is the reset tree plus whatever uncommitted changes the reset left, which D4 states per mode".
D4's list covers a moving reset only (pair: the run's own tracked changes discarded, foreign ones refuse; standalone: every uncommitted change stays).
The archive-only path drops the dirty-tree refusal and moves nothing, so in pair mode it leaves the run's own tracked changes in place, the opposite of the pair row in D4.
D3's own sentence "any uncommitted changes" covers it, but the cross-reference points at a list without that row, and D4's "envelope lists the uncommitted paths it left" is stated for standalone only.

**Fix:**
Add the archive-only row to D4's per-mode list (nothing discarded, nothing refused, tracked and untracked changes stay), and say whether the envelope lists the left paths on that path in pair mode.

### [LOW] A repeat amendment of an already rendered card is not specified

**Location:** `_lyx/discussion/decision-record.md` D6 (lines 124-141).

**Issue:**
D6 records a card in `AmendedCards` "with `rendered` false when recorded", sets it true at the spawn that renders it, and clears the list when a recovery records done.
It does not say what rebaseline does when the named card already has an entry with `rendered: true` (a recovery is live on the first amended text, the operator edits the card again).
If the entry is left as is, the recovery's done record clears it, and the code follows the older text while the plan carries the newer one, the disagreement D6 exists to prevent.
If the entry is reset to false, the recovery is forced failed and re-run, as intended.

**Fix:**
State that a rebaseline naming a card with an existing entry sets that entry's `rendered` to false.

## Focus verdicts

- **D7 landing path (focus 1):** defect, F1 and F2.
  The need for a Card Index line is real (`planparser.ParsePlan` builds cards from that index) and the index-only acceptance is the right shape, but the guard region is wrong.
  D7, Scope "Out" and the acceptance lines do not agree on one bound until F1 is fixed.
- **D1 and Open risks (focus 2):** not a defect.
  The three checks (highest begun batch, no live recovery strand of batch NN, report on disk) are mechanical for the one writer lyx can see.
  The recovery strand is already removed before `recover-batch` refuses on a head mismatch (`awaitTerminal` in `recoverbatch.go`), so the check does not dead-end the refusal that sends the caller there.
  The in-session fork is stated as an unchecked residue in Open risks.
  The remote task branch moved by `ResetPairCode` is part of the stated bound (fourth bullet), and matches the function's own documented bound (own task branch only, dropped commits reachable from HEAD, lease on the remote tip).
- **D6 (focus 3):** not a defect beyond F4.
  The mark clears when a recovery's report is recorded done, a stuck, dead or failed recovery leaves it, and the `rendered` flag bounds the forced failure to one per amendment.
  Master's ladder reads `card_amended: true` on a `batch_failed` refusal, so a recovery that fails for another reason keeps the existing stop.
- **D12 (focus 4):** not a defect.
  A profile with both keys keeps its own `orientation`.
  `startup_context` is a `weights` key (`profileWeights` in `internal/batcher/estimate.go`) and `alone_above` a profile-level key (`profile` in `internal/batcher/config.go`), and D12 places both at those levels.
- **D3 and D4 (focus 5):** the guards before target resolution (run lock, no state, merge in progress, foreign branch) match `PlanReset`'s order, and the live recovery strand removal stays in the CLI half as today, which D3 states.
  The one gap is F3.
