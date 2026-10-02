---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (Goal; Mirrors, name-forming bullet; Constraints, Told-Geometry bullet; Acceptance criteria; Notes, teller bullet)
    summary: the record still never decides how a standalone strand (lyx webster/burler run --target-dir, geometry from standalonegeom.ReedGeometry) is named, although the Claude line's --name and every mirror depend on a name reed must form there
---

### [BLOCKING] Standalone-mode naming is still undecided

**Location:** `decision-record.md`: Goal, Mirrors ("reed is told the name prefix … through its geometry, built by `hubgeom.ReedGeometry`"), Constraints (Cwd Resolution / Told-Geometry bullet: "only `hubgeom` reads the code record and the seed to tell them"), Acceptance criteria, and Notes ("`hubgeom.ReedGeometry` is the one teller to extend").
`support-log.md` has no interview turn, rejected alternative or ledger entry for it either.

**Issue:** Neither file mentions standalone mode or `--target-dir`.
lyx has two reed geometry tellers: `CONSTRAINTS.md` names `internal/hubgeom` and `internal/standalonegeom` as the only `Geometry` constructors, and `internal/webstercli/wiring.go` and `internal/burlercli/wiring.go` build reed's geometry with `standalonegeom.ReedGeometry(res.Target, res.StateDir, res.Hash8)` for `--target-dir` runs.
A standalone target has no weft:main, so no `.lyx-code`; no fabric task slug; and no shed seed to take a parent from.
It also has no watchdog: `reedengine.SpawnWatchdog` is inert without a hub path, "which standalone never does".

The record makes a name mandatory for every reed strand and decides the refusal only "on a hub".
The Claude engine's launch and resume lines reference `$LYX_STRAND_NAME` in every mode, since the engine is mode-blind, so a standalone strand needs a name reed formed, or the line hands Claude an empty `--name`.
The plan writer is left to choose between refusing every standalone spawn (kills standalone webster and burler), keeping today's `strand_name` template there (contradicts its retirement), or a fallback code (looks like the rejected repo-name fallback).
Each one invents a decision the record should own.

On the focus directive's sub-questions, the record currently answers none: which teller supplies the prefix, whether `LYX_PARENT` is ever set, how the `strand_name` retirement applies, and whether a criterion keeps a standalone run spawning.

**Fix:** Add a Name-grammar decision for standalone mode, recorded as an auto-mode assumption since the operator never addressed it:

- `standalonegeom.ReedGeometry` tells reed a code derived from the hash8 it already receives, through an `agentname` function that owns the rule (`s` plus the first five hex digits, which fits the code grammar); standalone strands take the two-segment prime form `<code>:<role>`.
  `standalonestate` stays untouched, so the Standalonestate Leaf Invariant holds, and the code is told, so the Told-Geometry Invariant holds.
- It does not contradict the rejected repo-name fallback or "no name without the code": a hub never takes a derived code and still refuses uncoded, and a standalone target has no code record that a later `lyx fabric code` could change.
- `LYX_PARENT` is never set in standalone mode; it has no seed.
- The `strand_name` retirement applies unchanged: standalone loads the same reed `Config` from its state directory.
- No watchdog drift repair and no `lyx reed list` row in standalone; the pane title and `--name` are set at launch only.

Correct the Constraints and Notes lines that name `hubgeom` as the only teller, and add an acceptance criterion: a standalone `webster run --target-dir` on a plain checkout with no `.lyx-code` spawns its strands named `<derived code>:<role>`, with `LYX_STRAND_NAME` set and `LYX_PARENT` unset.
In `support-log.md`, add the interview turn, the rejected alternatives (refuse standalone, keep the template there, a fixed sentinel code, a `--code` flag on standalone runs) and a ledger entry.
