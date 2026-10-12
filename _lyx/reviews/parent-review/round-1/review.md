---
verdict: APPROVED
---

## Scope against the board entry

The decision record covers every point the `judge-cap-call` entry sets, and adds nothing the entry rules out.

- The judge tags findings mechanical or design-wrong from the finding text alone: D1.
- At the soft cap, mechanical-only accepts unconverged and a fixed but unreviewed design-wrong finding buys one more round, with no arbiter spawn: D2 steps 2 and 5, D4.
- A returning blocker after the soft-cap round is circling and stops: D2 step 4.
- A hard cap with its own loom key beside `review_max_bounces`, default 4, per-perch overrides on the `<perch>_max_bounces` pattern, never exceeded: D3, D5, and D2's hard-cap check on a recorded `continue`.
- A design-wrong blocker at the hard cap or a returning blocker goes to the arbiter through the existing escalation, and the arbiter's `hand-on` goes to the orch: D6, D7, with the eligibility list unchanged.

The design applies the tags in Go rather than having the judge write the decision.
The entry's own Idea section says the Bouncer settles the round from the judge's tags, so this matches the entry.

Additions beyond the entry's text, each one a direct consequence of it:

- Darn-Review gets the rule and its own hard-cap key (A5): it is a Bouncer row with its own budget key, and the entry asks for the hard cap per perch.
- The arbiter stencil's fixed review rule (D7) is the one item of `arbiter-when`'s "Changes this implies" that this task makes stale.
  That note itself says the rule "moves to the judge".
  The record names the carve-out and leaves the eligibility list and the driver stencil to `arbiter-when`.
- The orch role stencil's `circling continue` and small-task budget advice follow from the hard cap.

## Other board entries

- `arbiter-when` (Ready): no contradiction.
  Its proposal narrows eligibility to hard-cap and circling causes; this design keeps the list unchanged and leaves the narrowing to that note.
- `go-driver-assistant` (running): no contradiction.
  Its D6 keeps spawning the arbiter at every Bouncer escalation the list admits, which is the list this design leaves alone.
- `report-timing` and `orch-split`: untouched; the record puts the reports out of scope.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits.

- `go-driver-assistant` touches files this run also changes: the arbiter stencil (its case-brief marker, D14), loom config (`driver_kind` and the tackler keys), `internal/loomcli/doc.go`, the orch role stencil (its D15 line) and `contracts/specs/refusal-spec.md`.
  The parent already recorded a decision on this overlap in that run's decision record ("Added after Discussion (parent, 2026-10-12)").
  `go-driver-assistant` lands first, and `judge-cap-call` takes its arbiter-stencil, loom-config and `loomcli/doc.go` changes at Merge-In; the designs do not conflict.
  The edits touch different parts of each file: a case-brief marker versus the review rule, and a go-driven-run line versus the escalation guidance.
  Since the parent has already settled this, it does not escalate.
- `pause-esc-rewind` changes `claudeengine`, `shuttleengine`, `pauseengine` and `orchengine`, and adds `PATTERN-guarded-interrupt`: no overlap.
- `reed-multi-window` changes reed, shuttle and seat window placement: no overlap.

Bookkeeping files only: none named yet at this stage beyond `contracts/specs/refusal-spec.md`.
`go-driver-assistant` also edits that file, in different rows; the parent decision above covers it.
