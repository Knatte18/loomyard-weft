# Fixer report — Discussion-Review round 2

The one finding is fixed; `lyx loom validate-discussion` exits 0 after the edits.

## Changes

- **F1 (BLOCKING), standalone-mode naming.**
  `decision-record.md`:
  - Goal: the two-segment form now covers a standalone run as well as the prime.
  - Scope In: standalone runs (`--target-dir`) get a code derived from the target, told by `standalonegeom`.
  - Name grammar: a new decision says a standalone code is `s` plus the first five hex digits of the hash8, with `agentname` owning the rule and `standalonegeom.ReedGeometry` telling it.
    The same decision says strands take `<code>:<role>`, `LYX_PARENT` is never set, the `strand_name` retirement applies unchanged, no watchdog repair or `lyx reed list` row exists there, and a hub never takes a derived code.
    Its rationale explains why this does not reopen the rejected repo-name fallback.
  - Mirrors: the geometry is built by `hubgeom.ReedGeometry` in a hub and `standalonegeom.ReedGeometry` in a standalone run.
  - Constraints: the Told-Geometry bullet names `standalonegeom` as the teller of the derived code, and a new Standalonestate Leaf bullet says `standalonestate` does not change.
  - Auto-mode assumptions: standalone naming was picked without the operator.
  - Acceptance criteria: a standalone `webster run --target-dir` with no `.lyx-code` spawns `<code>:<role>` strands with `LYX_STRAND_NAME` set and `LYX_PARENT` unset, and an `agentname` test checks that the derived code fits the code grammar.
  - Notes: both tellers are named as the ones to extend.

  `support-log.md`: interview turn 18 (testing moves to 19), four rejected alternatives (refuse standalone, keep the template, a fixed sentinel code, a `--code` flag), and ledger entry Q18.

## Deferred

None.
