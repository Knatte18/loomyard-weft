---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/01-gate-pending-state.md:33
    summary: Card 1 never says how an AttachGated run seeds the turn-boundary state, no test covers a pending entry across an attach, and the mid-turn-at-restart residual against D2 is unstated
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/01-gate-pending-state.md:25
    summary: GateEntry.Final is a new escape hatch at finalize with no bound at the shuttle contract; only pending is bounded to PassOnCap
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/02-loom-parent-review-config.md:24
    summary: parent_review_wait_min 0 is made legal as a near-off switch that still pings the parent, unlicensed by D1/D10 and at odds with the sibling knobs' 0 meaning
  - id: F4
    severity: LOW
    location: _lyx/plan/01-gate-pending-state.md:28
    summary: Card 1 does not say the per-arrival gate memo is cleared after a pending result, which both the poll-tick re-evaluation and the Final-at-finalize path depend on
  - id: F5
    severity: LOW
    location: _lyx/plan/05-parent-review-gate.md:42
    summary: Card 5's test list covers restart only for expiry and a consumed reject, not the cap and throttle or a notify recorded while the driver is down
  - id: F6
    severity: LOW
    location: _lyx/plan/07-loom-review-verbs.md:51
    summary: docs/overview.md edits miss the package tree line for internal/parentreview, the loom.yaml key list and the status waiting note
  - id: F7
    severity: NIT
    location: _lyx/plan/12-wire-parent-review.md:31
    summary: Card 12 widens the discussion commit pathspec with no bound sentence of its own
  - id: F8
    severity: NIT
    location: _lyx/plan/07-loom-review-verbs.md:34
    summary: Card 7 says only reject's review file resolves against the invoking cwd; approve --review is left unstated
---

The answer key is the decision record alone: there is no `_lyx/loom/rework/` directory, so this is generation 0.
Every Decision D1–D12 and every Constraint maps to at least one card, and no card carries scope the record does not license beyond the items below.

### [MEDIUM] Attach does not say how the turn-boundary state is seeded

**Location:** `_lyx/plan/01-gate-pending-state.md:33`

**Issue:** Card 1 keeps the turn-boundary flag, the last Done message and the pending-entry index on `Run`, and `reconstructAndWait` (`internal/shuttleengine/attach.go:208`) builds a fresh `Run` with all of them zero.
The restart acceptance criteria (no second request, at most one more gate prompt no sooner than a minute after the second, a `notify` made while the driver was down carried at the first boundary after restart) then rest on behaviour the plan never names: the attached run replays `events.jsonl` from offset 0, `pollEventsTick` (`wait.go:548`) reports `OutcomeDone` on that first batch because the output files exist, and that replayed Done is the arrival that puts the writer at a boundary.
That works for an idle writer, but card 1 neither states it nor tests it — `gate_pending_test.go` drives `Wait`, never `AttachGated`, and no other card covers a shuttle-level attach over a pending entry.
The same replay also reports Done when the writer was mid-turn at the restart (a delivery prompt or a reject re-prompt just sent), so a carried prompt can land inside that turn, against D2's "no gate send lands inside a writer turn"; the claude engine records no prompt-submit event the loop could use to tell the cases apart.
The plan states neither the mechanism nor the residual.

**Fix:** In card 1's Intent, state that an attached run starts mid-turn and that the replayed backlog's Done is its first arrival, so restart re-evaluation and a waiting `notify` are reached there; name the residual (a writer mid-turn at the restart looks idle after replay, the gap the existing discussion gate already has on attach) and its bound (at most one carried prompt per throttle window, the disk-held cap).
Add a `gate_pending_test.go` case driving `AttachGated` over a pending closure: the replayed Done evaluates once, poll ticks re-evaluate without a new arrival, and a pending result carrying text sends it.

### [MEDIUM] `GateEntry.Final` has no bound at the shuttle contract

**Location:** `_lyx/plan/01-gate-pending-state.md:25`

**Issue:** `finalize` evaluates `Final` in place of `Gate` whenever `Final` is non-nil, so a `Final` that returns passed lets an entry through at the deadline or final verdict that its `Gate` would fail.
Card 1's Bound sentence bounds pending to `PassOnCap` entries but says nothing about `Final`; the only thing keeping `Final` off a must-pass entry is that `shedrecipe` sets it for `parent-review` alone (card 6).
The rubric's attack-surface rule asks the introducing card to state what the new hatch lets through and what bounds it.

**Fix:** Card 1 states that `Final` is honoured only on a `PassOnCap` entry, that a non-nil `Final` on any other entry is a gate error at evaluation (mirroring the pending rule), and `gate_pending_test.go` adds that case.

### [MEDIUM] `parent_review_wait_min: 0` as a second near-off switch

**Location:** `_lyx/plan/02-loom-parent-review-config.md:24`

**Issue:** Card 2 makes 0 legal with the meaning "expires an open request at its first evaluation".
The closure's first call still opens the request and carries one delivery prompt, so the parent is notified of a request that expires at the very next evaluation, and every verdict verb then refuses it as expired.
D1 names `attempts: 0` as the off switch and D10 defines only the default of 60; neither licenses 0 as a second, noisier off switch.
The four sibling timeout knobs in `LoadConfig` (`internal/loomengine/config.go:399-411`) give 0 the meaning "defer to shuttle's `run_timeout_min`", and their shared refusal says so — adding this key to that loop would give it a way-forward clause that is false for it.
Card 2 states no bound for the new value either.

**Fix:** `LoadConfig` rejects `parent_review_wait_min` below 1 in its own check, with a message naming `attempts: 0` on Discussion-Write's `parent-review` gate as the way to turn the review off; `config_test.go` covers 0 and a negative value.

### [LOW] The gate memo after a pending result

**Location:** `_lyx/plan/01-gate-pending-state.md:28`

**Issue:** `evaluateGate` returns `run.gateVerdict` whenever it is set (`wait.go:809`), and today only the re-prompt path clears it.
If a pending arrival leaves its memo in place, a poll-tick re-evaluation reads the stale pending verdict and never sees a verdict the reviewer records, and a deadline `finalize` returns the memo without calling `Final`, so an open request is never marked expired.
Card 1 says the helper "evaluates the gate list from the first entry" and that `finalize` picks `Final`, but never says the memo is cleared after pending; the deadline test would catch the second half only.

**Fix:** Card 1 states that a pending outcome is not memoised past its arrival: the memo is cleared after a pending result, as it is before a re-prompt.

### [LOW] Restart cases missing from card 5's tests

**Location:** `_lyx/plan/05-parent-review-gate.md:42`

**Issue:** The acceptance criteria name a restart after two undelivered prompts (at most one more, no sooner than a minute after the second) and a `notify` run while the driver is down.
Card 5's test list simulates a restart only for expiry and for a consumed reject; the cap, the throttle and a waiting notify are tested within one closure only.
Card 14's integration test and card 1's tests cover neither.

**Fix:** `gate_test.go` adds the two restart cases by building a second `NewGate` over the same store: two prompts carried, then a new closure carries one more only after the minute and none after the third; and an `AddNotify` between closures is carried by the new closure's first call.

### [LOW] `docs/overview.md` edits are incomplete

**Location:** `_lyx/plan/07-loom-review-verbs.md:51`

**Issue:** The Documentation constraint covers `docs/overview.md`'s loom row, and CLAUDE.md asks for the module tree when a package is added.
Card 7 updates the loom row's verb list and names `internal/parentreview`, but no card adds `internal/parentreview/` to the overview's package tree, adds `parent_review_wait_min` to the loom row's `loom.yaml` key list, or adds the `waiting` note to the row's `status` paragraph.
The package docs the focus named (`shedverbs`, `battenshed`, `shedrecipe`) describe no hook, wait message or gate vocabulary the plan changes, so they need no edit.

**Fix:** Card 7's `docs/overview.md` Prosa also adds the tree line, the new key, and one sentence on `status`'s waiting-on-reviewer note.

### [NIT] Card 12 has no bound sentence

**Location:** `_lyx/plan/12-wire-parent-review.md:31`

**Issue:** Card 12 is where the gate entry goes live on Discussion-Write and where the discussion commit seam gains a directory whose files include a review copied in by an unchecked verb.
Card 6's bound covers the gate entry's `pass_on_cap` shape, which settles that half; the pathspec half has no bound stated anywhere.

**Fix:** One Bound line in card 12: the pathspec adds only the parent-review directory, whose files only the store writes, and which already sits under `_lyx/reviews`, tracked content the per-transition status commit carries.

### [NIT] `approve --review` path resolution

**Location:** `_lyx/plan/07-loom-review-verbs.md:34`

**Issue:** Card 7 says `reject`'s review-file argument resolves against the invoking cwd but is silent on `approve --review <file>`, which the verb runs from the prime against a task worktree's store.

**Fix:** State that both review-file inputs resolve against the invoking cwd.

## Focus settlements without a finding

- Cards 6 and 12 against D4, on empty rounds: D3 has every fresh start open the next round number with no parent condition, so `BeginRound` on a parentless start is licensed.
  The empty `round-<N>/` holds no request, so "without opening a request" holds; git tracks no empty directory, so the discussion commit is unchanged; card 4's `Latest` returns the empty round, the closure passes on no reviewer, and `BeginRound` supersedes only a round with an open request.
  Not a defect.
- Card 12's gate-entry half of the attack-surface question is settled by card 6's bound plus card 1's pending rule; only the pathspec half is raised (F7).
- Card 14's scripted shuttle fake follows the repo's convention that hermetic fakes evaluate a gate once and the real loop is covered by the smoke (`internal/loomcli/smoke_gate_test.go`).
  The live smoke carries the SendMessage criterion.
  "No path of delivery types into the reviewer's pane" is structural: the closure sends nothing (card 5), and the only send is shuttle's `Run.Send` to the writer's own strand (card 1).
