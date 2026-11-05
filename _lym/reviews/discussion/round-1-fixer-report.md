# Round 1 fixer-report — discussion

Every finding's premise held against the code and the record.
All are fixed in `_lym/discussion/decision-record.md`, with matching entries in `_lym/discussion/support-log.md`: a Round 1 entry, the existing-paths 5 line, two rejected alternatives, and Q4 and Q13.

## Changes

- F1 (BLOCKING): D3 now says the loader admits a stored `partial` and reads it as `full`. D15 gains a reinterpretation bullet for a hub base whose scope is `partial`, and its kept `scope` values admit a stored `partial`. The D15 criterion now covers it.
- F2: D3 gains a rule for a recorded `partial`. It is read as `full`, the cases on its skip list are left out of the diff, and the verdict names them. The D3 criterion, the in-flight Open risks line and a D13 bullet were updated to match.
- F3: D8 keeps the no-cold-fixer premise only for Darn-Write. For tapestry it states that Master edits no code and clears a run-level gate failure only through a fixer fork or `fix-verify`. A failed `drift` entry at Master's gate is cleared the same way, and the plan wires that under decision 3. D16 says who acts on the entry's findings in each recipe. The Auto-mode line, the plan-writer note on `gatefixer.go` and the rejected cold-fixer alternative were updated.
- F4: D7's unexpected-cell Stuck now names `amend` and a route that fixes code. In darn that route is the parent's `lym loom goto --to Darn-Write` then `lym loom resume`, with its bound stated. In tapestry the plan settles the route under decision 3, and the route must be reachable from a Stuck Drift-Verify or Drift-Recheck. The D7 criterion was added and the rejected routing alternative was reworded.
  - Fixed differently in part: the tapestry route is not specified here. Decision 3 assigns the tapestry fix loop to the plan, and Webster re-entered after its last batch has no card to run (support log, Rejected alternatives).
- F5: D15 now states the true behaviour. `skip` and `verify_max_runs` are retired under `CategoryNone` (`internal/configengine/doc.go:76`), so a run seeds and the key is dropped. The D15 criterion and the support log's existing-paths 5 line were corrected.
- F6: D2's Bound states that the judge covers only the cases it reads from the log. The self-check then added the bounding check: the `judgement` entry refuses an empty command-mode ruling after a failing compare (D9 and its criterion).
- F7: D2 and D8 agree. In `command` mode the entry and `lym drift check` run the seal check and the compare, report any seal failure, and pass. The D2 and D8 criteria were updated to match.
- F8: D6's re-entry seals an existing valid file without spawning only when the expect session ended normally, as recorded through `AgentSessionOpen`. An interrupted session is probed first. The D6 criterion was updated.
- F9: D6 names a way forward for each seal-check failure. An edited file names `--restore`. A mis-hashed seal copy, a `Warp-SHA` mismatch, and an unsealed file past Drift-Expect name a new `lym drift amend --discard`, which sets the expectation aside so the run is checked against its amendments alone. D11 gained the mode, its bound, and its criterion. The D11 criterion's `--restore` claim now covers only an edited file.
  - Fixed differently: `defer` was not used. It drops the regression check, while `--discard` keeps containment and costs the parent an amendment per drifted cell. Removing the file by hand could not clear a seal commit that already exists.
- F10: D9's snapshot now covers the HEAD of both checkouts, the warp's untracked paths and `_lym/drift/`. After the self-check it names the warp and the weft. The D9 criterion covers a judge that commits.
- F11: D6 defines "past Drift-Expect": the current row, read from the run status, comes after Drift-Expect in the recipe's row order. It also states that before that point `lym drift check` checks nothing and says so.
- F12: D11's Bound was rewritten as proposed. D2 now names the sealed expectation and the amendments among what the command-mode judge reads.
- F13: D8 states the gate-key fact. The Open risks line on Webster's gate seam was deleted.
  - Fixed differently: D7's rationale clause was reworded, not cut. It now says inheriting a held slot keeps a check run inside Merriam's run-level gate from waiting on its own slot. Without it, the Decision's slot inheritance would have no reason given.
- F14: D14 removes the `drift-verify` gate key from the gateslot template, from `goReadGates`, from the rows' `gate_key` and from a hub's `gate.yaml`. D7's capture takes its slot without a key; `Pool.Acquire` takes a `Holder`, not a key.
- F15: D13 renders the old fields of a run that checked under the old Drift. The Open risks line was extended. After the self-check, D14 keeps those fields read-only in the record model.
- F16: D4 defines a passing check in `command` mode: the compare exited 0, or the judge ruled every case it read `matches`. That lets Drift-Recheck skip at the same head. D2's "judged again" now applies to a head no passing check covers. The D4 criterion was updated.
- F17: the D6 criterion's clause about a hand-edited record was deleted.
- F18: D10 stores each `open_ranges` sentence beside the tuples. D13 renders each open range with its `why` and the judge's sentence. The D13 criterion was updated.
- F19: D12 admits `defer` at a Stuck Drift-Expect, which now stops naming `defer` when the baseline is missing. The D12 criterion was updated.
- F20: fixed through the review's second option. The occurrence is a separate field wherever an identity is written, so the identity syntax has no ambiguity (D1, D5 and their criteria).
  - Fixed differently: the review's first option, refusing a key that ends in `#` and digits, was made first and then replaced after the self-check. It would refuse ordinary output such as `Test #3: passed`. Its way forward, editing `key_patterns`, breaks an expectation already sealed against the old identities.
- F21: D5 gained the Bound line as proposed.
- F22: the D6 rationale sentence and the D12 rationale clause were deleted.

## Self-check

- fix-claims: no findings.
- fix-consistency, LOW, D11 recording and containment sentences and D7's classification left out `--discard`: fixed as proposed.
- fix-consistency, LOW, Auto-mode D12 line understated where `defer` is admitted: fixed as proposed.
- fix-consistency, LOW, D8 Rationale still had the writer fixing in tapestry: fixed as proposed.
- fix-consistency, LOW, support log's Round 1 count and "all fixed" claim: fixed as proposed. The count was cut, and the line names the two tapestry halves handed to the plan.
- fix-way-forward, MEDIUM, a seal failure at the writer's gate entry names a verb no session there can run: fixed with my own change. The entry reports a seal failure without failing, checks containment against the seal commit's copy, and leaves the stop to Drift-Verify (D6, D8, D2 and their criteria). Ending the writer at an immediate Stuck was not chosen because no existing mechanism does that.
- fix-way-forward, MEDIUM, no stated tapestry code-fixing route: fixed as proposed. D7 states that the plan's route must be reachable from a Stuck Drift-Verify and a Stuck Drift-Recheck, and why.
- fix-way-forward, LOW, dropping `skip` named `lym config drift`: fixed as proposed. It now names `lym config reconcile`.
- fix-proposal, MEDIUM, F15 contradicted D14 and Open risks: fixed as proposed. D14 keeps the old fields read-only, and the Open risks line says only the verdict reads them.
- fix-proposal, MEDIUM, the darn goto skips review of the fix: fixed as proposed. The bound is stated in D7.
- fix-proposal, MEDIUM, the F20 refusal fires on `Test #3` and breaks a sealed expectation: fixed as proposed, by moving to a separate occurrence field.
- fix-proposal, LOW, D9 named no checkout: fixed as proposed. It names the warp and weft HEADs and limits weft changes to `_lym/drift/`.
- fix-proposal, LOW, an empty command-mode ruling counts as passing: fixed as proposed. The `judgement` entry refuses an empty ruling after a failing compare (D2, D9).

## Disputed

None.

## Deferred

None. The tapestry halves of F3 and F4 are not deferred: they are handed to the plan by the operator's decision 3, and the record now states the facts the plan needs.
