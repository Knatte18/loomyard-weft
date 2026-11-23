```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I traced what the diff's changes force outside the lines it touches, and found no member of the affected set left wrong.

**Per change, the set found and whether it is covered:**

1. **`wait` gains a `keepsSession bool` parameter.**
   - It has three production callers, `internal/landingshed/prreview.go:306`, `:346` and `:371`, and all three were updated.
   - Its one test caller, `internal/landingshed/prreview_wait_test.go:1131`, was updated.
   - No other caller exists, since `wait` is unexported. Covered.

2. **`prWaitState.keepsSession` field.**
   - It is written only at `prreview.go:625` and read only in `haltOnEndedSession`. Covered.

3. **New behaviour: the wait halts once the session it keeps is no longer live.**
   - The halt's reason has the trailing `way forward:` clause the refusal rule asks for.
   - `PATTERN-refusal-way-forward` binds only the `internal/webster*`, `internal/shed*` and `internal/loom*` packages, so it does not bind `landingshed`. The new row still asserts the message.
   - The docs that describe the behaviour were updated in the diff: the package doc (`doc.go:24`), the `PRReview` type doc (`prreview.go:49`), the `wait` doc and the `wake` doc.
   - The `Call` doc (`prreview.go:72`) says the session is stopped on every verdict. That still holds: the halt runs `endSession`, which finds the strand dead and stops nothing.
   - Production wiring is `sessionLiveness` in `internal/loomcli/landingdeps.go:1056`. It returns `Live && !Retiring` for a known strand and false for an unknown one, so a strand removed with `lym reed remove` reads as not live. That matches the issue.
   - The shared fixture `internal/testkit/envkit/envkit.go:272` answers `SessionLive` false and `Sleep` nil. No test outside `internal/landingshed` drives the PR-Review wait into an open review over it, because such a wait would never end even at the base.
   - The PR-Review session stencils, `contracts/stencils/landing/landing-template-pr-review.md:49` and `landing-block-pr-review-operator.md`, say "The run ends on the pull request's state, not on this session". The halt does not end the run, so the stencils are not contradicted, and the diff changes no stencil, so the "Stencil wording" check does not apply.
   - The orch role docs (`contracts/orchrole/orch-role.md:202-208`, `role-autonomy.md:44-48`) describe PR-Review asks only, nothing about how the row ends. Nothing is stale.
   - No spec under `contracts/specs` describes the PR-Review wait's ending.

4. **Commit messages and departures.**
   - `0471ada9c` carries a `Root-Cause:` trailer naming the defect the diff removes: `wake` never read the recorded session's liveness.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer is needed or present.
   - The commit message's claims match the code: a wait entered on an ended review or a merged pull request is not watched.
   - No declared PATTERN changes are listed for this round, so nothing is due under "Declared PATTERN changes landed".

**Findings:** none.

**Tool calls:**
1. Searched `contracts`, `docs` and `plugins` for "PR-Review" and "no timeout", and `internal/landingshed` for `wait` callers. The context lacked the docs and specs that describe the wait, and a complete caller list.
2. Searched the PR-Review stencils and the orch role docs for wording about a session ending, halting or resuming. Those files were not in the context.
3. Read the operator block and the template's session lines in full, to judge the "run ends on the pull request's state" wording against the new halt.
4. Read the `SessionLive` stubs in `envkit` and listed tests outside `landingshed` that name PR-Review. The context lacked the shared fixture's other seams, which decide whether any existing test reaches the new halt.
