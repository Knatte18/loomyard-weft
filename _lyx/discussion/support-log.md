# Support log — arbiter-fixes

## Interview

Autonomous run: no operator answered; every pick below is the writer's own, listed in the Question ledger.

Turn 1 — exploration.
Read the board entry, `arbiter-when`, `orch-split` and the board list.
Read `arbiterconsume.go`, the arbiter call in `arbiter.go`, `shedengine.Relaunch` and `Goto`, the Go driver's `arbitrate`, `recheck` and rule table, the LLM driver stencil's `## Arbiter`, battenshed's doc on the first-verify-failure retry, the arbiter and alignment stencils, the implementer and Master stencils' `deviations` text, record-batch's scope-guard call, `ScopeGuard` and the digest.
Found: the attempt's stop key is read at the verb's start; a goto writes `paused`, a new history entry and clears the pause conditions; batten's verify retry performs a goto and a resume on its own; the Go driver hands a watched verify halt back without an agent and rechecks after a verdict, while the LLM driver sends every blocked stop to the arbiter; the hub's `driver_kind` is `llm`.

Turn 2 — design settled as D1 to D5 of the decision record.

Turn 3 — lens review: six forks spawned (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness); all six returned.
Duplicates merged; one line per finding:

- pattern-fit MEDIUM, D2 overlaps the existing Publish-halt refusal row: fixed; D2 now widens that refusal to Merge-In and edits its row, dropping the watched/unspent condition.
- scenarios MEDIUM, D2 depends on a live batten after the refusal: fixed by the same change; the widened refusal matches the Publish behaviour, where the parent takes the run when no batten retries.
- scenarios LOW, watch marker with no scratch dir reads the retry unspent: fixed by the same change; D2 no longer reads the watched facts.
- scenarios LOW, shared helper not a copy for the watched facts: fixed by the same change; no longer needed.
- premises LOW, goal-scope LOW, scenarios MEDIUM, pause check misfires after a goto: fixed; D1 refuses only a condition present now and absent at spawn, with a criterion.
- scenarios MEDIUM, a parent's goto to the same row is now stepped: fixed; D1's bound states it.
- scenarios LOW, hub pause on an admitted re-launch: fixed; D1 and a criterion say the step holds under the hub pause.
- scenarios LOW, batten's resume after an admitted re-launch: rejected; start and resume already give a no-op success over a running run, and D1 changes neither.
- pattern-fit LOW, D1 needs a generic skip signal in `shedengine`: fixed; D1 and Constraints name it under `PATTERN-shed-producer-seam`.
- pattern-fit LOW, consume refusal rows go stale: fixed; Constraints name both consume rows.
- premises LOW, pattern-fit LOW, `ParseReport` refuses unknown keys: fixed; D4 says the parser keeps accepting the key.
- premises MEDIUM, pattern-fit LOW, the guard's findings carry no file field: fixed; D4 gives the finding a structured file and Scope's Out no longer forbids the output change.
- goal-scope MEDIUM, leanness MEDIUM, non-code files drop out of the list: fixed; D4 adds a file-level comparison for files the delta gives no symbols for, with a criterion and the glyph chokepoint constraint.
- goal-scope MEDIUM, D5 silently dropped escalation on a duplicated decision: fixed; D5 keeps duplication as a design conflict.
- goal-scope LOW, D5 tie and unknown position: fixed; D5 reports a tie or unknown order with the verdict unchanged.
- leanness MEDIUM, unreadable-peer rule sat in assumptions: fixed; moved into D5.
- goal-scope LOW, D3's alignment part had no rationale: fixed.
- goal-scope LOW, D1's tackler extension is beyond the entry: rejected; one shared admit keeps the two consumers from drifting, ledger Q3.
- goal-scope LOW, D2 still costs a park: rejected as a finding; the park is the designed hand-back, and the criteria name the hand-back.
- board-fit MEDIUM, D4 removes what `glyph-use-audit` planned to compare: fixed; Open risks names it.
- board-fit LOW, `arbiter-when` also edits the same spec row: fixed; the Open risks line says it extends D2's row.
- board-fit LOW, `go-driver-assistant` dependency unnamed: fixed; first Constraints line.
- board-fit LOW, `orch-split`, `logging-cleanup`, `twb-ftb-1` checked: no change needed.
- leanness LOW, assumption 1 restated D2: fixed; cut to the records-gone fact.
- leanness LOW, D2's driver narrative in the Decision: fixed; moved to Rationale.
- leanness LOW, D1's bound restated its conditions: fixed.
- leanness LOW, Scope's In restated the decisions: fixed; reduced to the entry's sections.
- pattern-fit LOW, test budget: fixed; Constraints line.
- premises note, `lyx pause` writes no status pause condition: confirms the record; no change.

## Rejected alternatives

- Admitting any moved run whose current producer equals the verdict's goto, whatever the history: lets through a run that stepped on and came back, which the verdict never judged.
- Fixing the race on batten's side, by holding its verify retry while an arbiter marker is live: keeps a spawn whose verdict only restates batten's goto, and leaves the LLM and Go drivers handling the same stop differently.
- Refusing the arbiter only at a watched Merge-In verify halt with an unspent retry, from the Go driver's watched-verify facts (the first draft): overlapped the existing unconditional Publish refusal, left a run parked if batten died, and misread a watch marker without a scratch dir.
- Parsing the file out of a scope finding's detail text: a second, unowned parse of a message.
- Accepting that non-code files drop out of the deviation list: the incident's own list held one, and the fork's list carried them.
- A new LLM driver stencil rule for watched verify halts: the verb's refusal already routes the driver through its existing hand-back path, so a stencil rule would restate Go's decision.
- Having the fork report every path it changed and the guard filter it (the entry's first option): the delta already knows every changed path, so the fork's list is redundant work and a second source.
- Keeping the scope findings in record-batch's warnings beside the digest list: shows the same fact twice.
- Accepting a bare slug in `lyx board get`: a CLI change for one stencil's wording.
- A verdict key for the landing order: the entry asks for it in the report, and `ReadVerdict` refuses unknown keys, so a key would change the verdict contract for no reader.
- Extending the arbiter-trigger narrowing to every blocked stop that names a goto: that is `arbiter-when`'s scope.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Which actor moved the run during the arbiter's spawn? Batten's first-verify-failure retry, from the code path and both incidents' shape; the attempt records are gone, noted as an open risk.
- Q2 (auto-pick): How strict is the verdict-already-in-effect admit? Paused on the exact target, reached by goto entries only after the attempt's history length, and no pause condition present now that was absent at spawn.
- Q3 (auto-pick): Does the tackler's consume take the same admit? Yes, one shared check.
- Q4 (auto-pick, revised after the lens review): Close the race on the arbiter side or batten's side? The arbiter verb's eligibility, by widening the existing Publish-halt refusal to Merge-In.
- Q10 (auto-pick): How does the deviation list keep non-code files? The guard compares a changed file with no symbols at file level against the cards' target files.
- Q11 (auto-pick): Is a duplicated decision with a peer still an escalation? Yes, a design conflict.
- Q5 (auto-pick): Which of the entry's two deviation options? The fork reports nothing, and the guard's findings over the batch delta are the list.
- Q6 (auto-pick): Do scope findings stay in record-batch's warnings too? No, the digest list alone.
- Q7 (auto-pick): Where does the alignment agent record the landing order? In its report; the verdict keys stay.
- Q8 (auto-pick): Is a same-PATTERN-entry change still an escalation? Only when both runs change it in different ways.
- Q9 (auto-pick): Fix the stencil or the verb for the board-get JSON error? The arbiter and alignment stencils name the JSON payload form.
