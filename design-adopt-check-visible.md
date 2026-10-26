# Adopt machine check: a gate key of its own, and visible while it runs

`adopt-check-visible` · Note · enhancement, gate, board

## Seen (operator, 2026-10-26, OSL-1046)

`lyx adopt --prime` on OSL-1046 started the machine check in the background (`lyx gate test --gate finalize --wait`) and printed only its pid and a log path, `_board/.lyx/adopt/suite.log`, inside the JSON.
Nothing on screen showed that a test run had started, how far it had come or how it ended.
The check exists to find what a move between machines breaks, here from Neve on Ubuntu 26 to OSL-1046 on Ubuntu 24.
The operator dislikes the gate key `finalize` for it: the run is the adopting machine's check, not a Finalize.

## Idea

- The adopt check runs under a gate key of its own, such as `adopt`, by the rule one gate key per producer, named after it.
- Its progress and result are visible from where the operator works: a window or strand of its own, or a line per tier to the orch and in `lyx beam status`, and a notice when it ends, failed or green.
