---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomshed/samplersession_test.go:279
    summary: no test row reaches the wake-path done-flag branches, a dirty checkout cleared, resumed, mailed and waited on, or a damaged decision record Stuck
    fix: In TestSamplerSession add two rows whose onWake raises the flag at wake 2 on a live session. One sets w.dirty to a dirty checkout and wants a cancelled-context error, one mail naming the path and sent to the resumed strand, a resume of the recorded session id and a record DoneStamp of 0. The other sets w.findings to invalidRecord and wants Stuck with Halt and a reason holding the finding and "git -C _lym checkout". The existing dirty and damaged-record rows write the flag before the call, so they reach only the entry path in execute.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/decision.go:651
    summary: an unreadable sampler record or status refuses the run's parent orch too, because the samplerStrand read error is returned before the parent check
    fix: In decisionVerb's named-caller branch, when deps.samplerStrand returns an error, fall through to the existing parent comparison rather than refusing. Refuse with the "read the run's sampler session" message only when the caller is not the parent orch. Add a row in decision_test.go with a failing samplerStrand and the parent as caller that wants the append to succeed.
---

### [MEDIUM] Wake-path done-flag branches have no test row

**Location:** internal/loomshed/samplersession_test.go:279, covering `wake` at internal/loomshed/samplersession.go:367
**Issue:** D6 gives the done flag two judgments: on entry, and on a wake during the wait. The entry judgments are pinned. On the wake path the dirty-checkout branch returns not-ended and moved, so the session is resumed, mailed and the wait continues. The damaged-record branch is Stuck with Halt. Neither has a row, because every dirty and damaged row writes the flag before the call. A break specific to the wake, such as the dirty branch ending the call or dropping the mail, passes the suite.

### [LOW] An unreadable sampler record refuses the parent orch

**Location:** internal/loomcli/decision.go:651
**Issue:** The new sampler check runs before the parent comparison for every named caller. On a sampler run standing at Sampler-Session, an unreadable status or sampler record makes the call refuse with "read the run's sampler session", including the run's own parent, whose call did not depend on those files before this diff. The refusal does name a retry, so the cost is a failed call, not a wrong append.

## Rejected

- The `samplerDoneCmd` closures, `currentRow` and `stamp`, have no test (lens:code-tests): they are thin wiring over seams the verb's table test drives, and a test of them would re-cover that behaviour.
- `samplerStamp` is run with one history only (lens:code-tests): the `goto` and multi-predecessor rules live in `EntryStamp`, which `TestEntryStamp` covers, so a wiring-level row would duplicate it.
- No board row for `recipe: sampler` accepted on a task and refused on a project (lens:code-tests): the diff changes only the refusal message in `normalizeTask`, and the vocabulary is pinned by the `RecipeNames` and seed tests, so no behaviour is added without coverage.
- No `--amends-plan` refusal row for a sampler run (lens:code-tests): the refusal is the existing rows-keyed code darn already uses, and the diff adds no behaviour there.
- `landingDeps` call sites that pass `recipe` are not asserted (lens:code-tests): the seams are tested directly, and a compile error would catch a missed caller.
- Reports and timing spans list only tapestry and darn (lens:code-affected-set, considered and dropped): the decision record asks for no sampler reporting.
