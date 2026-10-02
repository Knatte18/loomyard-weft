# Parent review as a step in Discussion-Write's gate

Discussion-Write now asks the run's parent to review the discussion once, after the mechanical `discussion` validator passes.
This ports millhouse's `orch-review`/`orch-wait` into the producer gate.
A reject re-prompts the still-live writer with the review, so the writer fixes the discussion with its own context.
The next arrival re-runs the validator and then goes through to Discussion-Review as before.
No new recipe row and no separate fixer.
Discussion-Write and the reviewer also read the board, so a design fits planned work instead of duplicating or contradicting it.

The board entry carries no GitHub issue, so there is no `Closes` line.

## What changes

- **Gate (`shuttleengine`, `shedrecipe`):** `GateResult` gains a pending state, and `GateEntryState` gains `waiting`.
  A pending entry sends no re-prompt and changes no count.
  The wait loop keeps polling, re-evaluates on a tick, and sends only when the writer is at a turn boundary (last arrival Done, nothing sent since), so no gate send lands inside a writer turn.
  Liveness checks and the run deadline keep running while an entry is pending.
  `finalize` never waits: a pending entry is reported `waiting` and counts toward `Passed`.
- **New gate name `parent-review`:** the Discussion-Write row's `gates` list becomes `discussion` (`attempts: 3`) then `parent-review` (`attempts: 1`, `pass_on_cap: true`).
  `attempts: 0` turns the review off.
- **`internal/parentreview` (new):** a per-round request directory under the run root, committed to weft with the discussion pair.
  It holds the request, the delivery record, the verdict and the copied review file.
  The gate closure reads that state, so an attach re-joins the open round, and the wait bound counts from the original open time.
  A fresh Discussion-Write start opens the next round and supersedes any open earlier one.
- **Delivery:** the live writer sends one `SendMessage` to the parent's full name, taken from the seed, and then runs `lyx loom review delivered`.
  The gate re-sends at most once a minute and at most three times per round.
  Nothing types into the reviewer's pane.
- **`lyx loom review notify|delivered|approve|reject [<slug>]`:** the slug is optional in a task worktree and required from the prime.
  Each refusal names its way forward and has a row in `refusal-spec.md`.
- **Status:** `lyx loom status` and `lyx batten status` show a run waiting on its reviewer, who it is, when the request opened and whether it was delivered.
  `shedverbs` gets a told `Waiting` hook on its generic status verb, so it stays path-free.
- **Config:** `loom.yaml` gains `parent_review_wait_min` (default 60).
- **Stencils:** the Discussion-Write stencil reads the board, handles a parent-review re-prompt and delivery prompt, and has two write-boundary carve-outs.
  A new reviewer brief points at the review-output format in the Burler stencil rather than restating it.
  The orch start and resume stencils handle a review notice through a one-shot fork.
- **Docs:** `shed-recipe-spec.md`, `loom-status-spec.md`, `refusal-spec.md`, `docs/overview.md` and `CONSTRAINTS.md`.

## Design decisions to check

- **Pending is bounded to `PassOnCap` entries.**
  A pending result from any other entry is a gate error, and `shedrecipe` rejects `parent-review` without `pass_on_cap: true` at construction.
  Pending can never let a must-pass gate through.
- **The verbs do not check who runs them.**
  Any session or the operator can submit.
  The worst a misused verb does is skip this one advisory review or spend the single re-prompt.
  The independent Discussion-Review rounds run after every outcome.
- **Consumed-reject window.**
  If the driver dies between marking a reject consumed and the writer receiving the re-prompt, that review is lost and the next evaluation passes.
- **Delivery wording sits behind the provider seam.**
  `shuttleengine` and `reedengine` gain no Claude specifics.
- **The request directory sits beside the Discussion-Review run directory,** never under `_lyx/discussion/`, which `discussionparser` owns.
- **Expiry:** a request with no verdict past `parent_review_wait_min` passes with a Warn and is marked expired.
  `discussion_timeout_min` still bounds the whole run.

## To check by hand

- **Live smoke with a hand-started parent session** receiving the notice through `SendMessage` was not run.
  The tests use a fake reviewer and a fake delivery seam.
- **`gofmt -l`** flagged `internal/loomcli/wiring.go` during the run; a later commit gofmt'd it, but re-run `gofmt -l` to confirm.
- **Test-only edits outside the feature:** `completionsignal_enforcement_test.go` (tripwire list follows an extracted helper) and the `loomcli` and `webstercli` `wiring_test.go` fixtures (the new required config key).
