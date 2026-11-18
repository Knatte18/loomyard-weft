```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that every refusal, error, next step, row or test the fix added can be reached from the state it names and leads to progress.

Ways forward the fix introduced:
1. D2: Preflight refuses an adjacent-duplicate `reviewer` list, with a `run_settings` write as the way forward.
2. D4: the `live` gate refuses a key that names tier 3, with a `lym config gate --set` write as the way forward.
3. D4: `--copies` is refused without `--gate`, with a tier-3 key, or with no package named.
4. D4: once `live_attempts` are spent, the row ends with the live report red and the failures deferred.
5. D10: two refusals of a segmented build row: one no Bouncer edge reaches, and one whose segment holds no Bouncer.
6. D8: a reinterpretation, applied by `hubreconcile`.
7. D5: reaps by Finalize and by a halted campaign, and pruning of the hub list.

Findings:

- **MEDIUM.** D4, "It refuses either key naming tier 3, with the `lym config gate --set` write that drops the tier as its way forward."
  - Summary: the refusal comes too late for anyone to act on it. The live gate runs inside the Harden-Burler-Fix row, partway through an unattended campaign. The fixer cannot carry out a hub `gate.yaml` write, and no operator is present.
  - Evidence: D2's rationale says "the run needs no operator during its rounds", and D2 puts the run's other config refusals at Preflight.
  - Effect: a misconfigured key is found only after review and fix work has been spent, and the run then stalls. The way forward can only be reached if the check runs at Preflight.

- **MEDIUM.** D4, "When they run out, the gate ends the row instead of halting the run: the live report is marked red and each failing test is recorded as a deferred finding".
  - Summary: the new outcome leads into a state where convergence is impossible. D6's CONVERGED rule still needs "a green live gate". So a red gate caused by a deferred finding outside the module makes CONVERGED unreachable in every later round. The campaign can then only end capped or circling and escalate to the parent. The row goes on, but the campaign cannot finish.
  - Evidence: D6, "CONVERGED needs a round whose review records no BLOCKING or MEDIUM finding, a green live gate, and a reviewer model different from the previous round's". D6's third bullet says a fix outside the module "is deferred by the fixer ... never fixed in the round".
  - Missing: the record should state how CONVERGED treats a red gate whose failures are all deferred, or state that such a campaign ends capped by design.

- **MEDIUM.** D10, "with adding the segment's Bouncer row or dropping the row's segment as its way forward".
  - Summary: the second option does not make progress. A build row with no segment is checked by today's order clause. A ring fix row sits only on on_stuck edges, never on the on_done path, so today's clause refuses it ("does not lie after the record row ... on the on_done path"). Following "drop the segment" therefore swaps one refusal for another.
  - Evidence: `internal/shedbuild/loomcontract.go` `checkLoomContract`, the onDonePath index check. D10's rationale says the fix row "sits on the ring's on_stuck edges, never on the on_done path".
  - Fix needed: the way forward should name only the change that passes, which is adding the Bouncer row, or say when dropping the segment applies.

Further tool calls: none. My context already held the changed record, the review and `loomcontract.go`.
