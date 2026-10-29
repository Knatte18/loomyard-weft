```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on a concrete case and checked the end state against the decision's purpose.

**Findings**

1. **BLOCKING — D3 and D4: the entry gate described here cannot be built.**
   - Case: the parent is `on`, the entry field is `off`, the hub has a GitHub origin.
   - D3 says the PRReview registry entry "builds an `entry_gate`" from a landingshed resolver. But `internal/shedbuild/entrygate.go` builds a row's `shedengine.EntryGate` only from the recipe YAML's declarative checks. Each check has the form `&lt;operand&gt;`, `&lt;operand&gt; == &lt;literal&gt;` or `!=`, and an operand is a `module.key` config setting or `budget.&lt;Segment&gt;`. A registry Constructor never supplies a gate.
   - A `beam.pull_request` operand would also read the run's own config stack, which is the task worktree's anchor and not the parent's. D1 says the mode must be read from the parent's worktree.
   - The board field and D4's check of the origin URL and upstream are not config keys, so no `entry_gate` check can express them.
   - End state: either the row cannot be Skipped as D3 and D4 say, or it reads the wrong anchor and misses D1's purpose.
   - Fix: either the row resolves the mode itself and returns Done with a skip reason, which the record must then say changes the history outcome, or the record decides a new entry-gate operand kind and adds it to the `PATTERN-shed-recipe-registry` change.

2. **MEDIUM — D9 and D7: Done can leave a dirty tree for Finalize.**
   - Case: an approval lands on the pushed head while the agent is mid-edit.
   - D9's Done test checks that the head is pushed, gated and approved, but not that the tree is clean.
   - On Done, D7 stops the session and leaves its uncommitted edits. Finalize's clean-tree check then goes Stuck naming the paths.
   - The run halts after an approval, which works against D9's purpose of following the pull request to the landing.
   - Fix: Done should require a clean tree, as D8 does before gating.

3. **MEDIUM — D3, D4 and D11: a stale pull-request record overrides a later `off` or skip.**
   - Case: PR-Review opens a pull request and writes the record. Finalize reroutes. The operator then sets the parent to `off`, or the field to `off`.
   - On re-entry the row is Skipped, but the record is still there.
   - Finalize's "With a record" branch takes the pull-request path anyway, against the mode D3 says it shares with the row.
   - Fix: the record should say whether a skip clears the record or closes the pull request, or whether Finalize's mode check wins over the record.

4. **MEDIUM — D9: the review-state rule misreads GitHub.**
   - Case: reviewer A requests changes, then leaves a plain comment review.
   - "No reviewer whose latest review is CHANGES_REQUESTED" reads A's latest review as COMMENTED, so the run passes.
   - On GitHub, a COMMENTED review does not clear a change request; only an approval or a dismissal does.
   - The result is a Done over an outstanding change request.
   - Fix: when picking a reviewer's latest review, skip COMMENTED (and PENDING) ones.

5. **LOW — D11 and D9: a pull request merged outside lym loses lym's trailer.**
   - Case: someone squashes the pull request on GitHub with GitHub's default message.
   - The parent then carries no `Lym-Task:` trailer and none of the `engine_trailers`, which Finalize writes only on its own merge (`internal/landingshed/finalize.go` and `doc.go` are the only places that name `Lym-Task`).
   - The landing still works, but the commit-trailer contract does not hold for this path, and the record does not say so.
   - Fix: state the bound in D11.

6. **LOW — D2: "a beam whose own `pull_request` is `off`" is ambiguous.**
   - That `pull_request` could be D1's `beam.yaml` key in the beam's worktree or D2's field on the project entry.
   - Walked either way, the outcome is right: the parent `main` decides. But a plan writer could wire the beam's landing to read its own `beam.yaml`.
   - Fix: name the D1 key explicitly in that sentence.

**Decisions walked**

| Decision | Case | Purpose met? |
|---|---|---|
| D1 | Prime file unset, task worktree sets `off` | Met if read from the parent anchor; the gate wiring breaks it (finding 1) |
| D2 | Parent `required`, field `off` | Met: a pull request is opened with a Warn |
| D3 | Effective mode `off` | Not met: no gate mechanism builds it (finding 1); stale record (finding 3) |
| D4 | Local origin, default `on` | Not met: Skipped needs a gate that cannot test the origin URL (finding 1) |
| D5 | `pr_account` set, `GH_TOKEN` set | Met |
| D6 | Crash after opening, before writing the record | Met: the lookup by head and base finds the open pull request |
| D7 | Re-entry with the strand live, then with the window closed | Met |
| D8 | Agent commit fails the tier-0 gate | Met: nothing pushed, one mail to the agent, approval waits |
| D9 | Approval while the tree is dirty; a comment after a change request | Not met (findings 2 and 4) |
| D10 | Open, then end | Met |
| D11 | Approved pull request, parent moved, branch protection dismisses stale approvals | Met: halt, re-approve, resume, then the merge goes through |
| D11 | Merged on GitHub | Met through the already-landed path; trailer gap (finding 5) |
| D12 | Cancel with an open pull request | Met |
| D13 | Pawl beam landing into `main` set to `on` | Met if mode resolution is fixed (finding 1); wording (finding 6) |

**Tool calls**

1. grep for `Lym-Task` readers and `entry_gate` in shedrecipe — my context did not show who reads the trailer or how entry gates are built.
2. grep for `EntryGate` across loomrecipe, shedbuild and shedengine — the first grep found no builder in shedrecipe.
3. read the top of `internal/shedbuild/entrygate.go` — needed the operand grammar to test D3 and D4.
