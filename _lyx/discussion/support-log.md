## Interview

No operator answered; the session ran autonomously.
Read the board entry for this slug and the board list; the sibling `twb-*` entries are the same bench task and were not treated as scope.
Read code at base 800a0e892 with `git show`: `planglyph` DoneChecks and resolve policy, `websterengine` run writes and recovery prompt, the Master and recovery stencils, `shuttleengine` wait and stop, and PATTERN-completion-signal.
Findings that shaped the decisions:

- `background_shell_wait_min` already expires nothing at base, and a waiting turn end with a shell is already held, so item 1 targets a remaining finalize path and the missing discard record.
- `recordedSessions` audits Master sessions only, which explains #522's misattribution.
- The Master stencil already carries the fixer rule in a late section; the gate message itself says "fix every finding".
- Plan-time `glyph-ambiguous` already exists for non-Create targets, so item 6 is one shared build-constraint predicate.

## Rejected alternatives

- Item 1: killing the wait after `background_shell_wait_min` — rejected, it is the reported fault.
- Item 2: a new `lyx webster` verb that spawns the fixer — rejected, forks are spawned through the Agent tool in-session and a verb cannot do that.
- Item 2: a hard guard that blocks Master edits during a gate failure — rejected, the run-exit audit already enforces and a new guard is scope beyond text.
- Item 3: starting the second recovery from a clean tree with the first diff as reference — rejected, it discards usable work and needs a destruction path.
- Item 6: honouring build constraints inside quarry — rejected, quarry is an external dependency.
- Item 6: skipping the ambiguity finding entirely — rejected, a genuine ambiguity must still block.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: does a shell hold or a discard record close #505, #509 and #517? Auto-pick: both; hold first, record as the fallback. Resolved.
- Q2: verb or text for the verify-fixer rule? Auto-pick: text at the gate message and stencil. Resolved.
- Q3: attribution by batch or clean-tree restart? Auto-pick: attribution by batch via recorded recovery sessions. Resolved.
- Q4: which exit for a terminal failed recovery? Auto-pick: non-zero for terminal failed or dead only. Resolved.
- Q5: does quarry resolve unexported and test-file identifiers? Auto-pick: unknown; open, the planner establishes it and decision 6 covers both outcomes. Open.
- Q6: how to judge disjoint build constraints? Auto-pick: pairwise over the tags both expressions mention, with filename GOOS/GOARCH suffixes, failing closed. Resolved.
- Q7: whether a schema change for recovery session ids is acceptable? Auto-pick: additive only. Resolved.
