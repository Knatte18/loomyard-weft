```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to a goal point of the board entry.

**Decisions** (the goal points of the board entry under `## Goal`; the record has no `Added after Discussion` entries):
- **G1:** The message reads origin before it claims the board was never seeded.
- **G2:** The message names a way forward that works when origin holds the seed.
- **G3:** A killed seed push that took the records branch and withdrew it says so, as the declined-board message already does.
- **G4:** "Drive it in the second hardener campaign, after `harden-sandbox-fabric-verbs` lands." This is a scheduling note for the campaign, not a code change, and the diff has nothing to build for it.

**Changes**, mapped to decisions:
- **C1:** `refuseUnseededBoard` reads origin's board with `originBranchHead` and names the `--reset` clone only when origin holds no board (`pushclassified.go:205-213`). Traces to G1.
- **C2:** Origin holding this board's tip or an ancestor of it names the four `git config` upstream commands, then a rerun (`pushclassified.go:215-244`). Traces to G2.
- **C3:** Origin holding another board, or an unborn local board, names the remove-and-join way forward (`pushclassified.go:228-232`). This is forced by C1: once origin is read, an origin holding a board that is not this seed cannot honestly be called "never seeded", and needs its own way forward.
- **C4:** An unreadable origin gets a refusal with a rerun way forward, and a failed read of `PrimeRecordsBranchKey` is wrapped (`pushclassified.go:207-209`, `233-235`). Forced by C1 and C2, because each adds a read that can fail.
- **C5:** `withdrawUnansweredSeedPush` now returns `withdrawn`, and the records-branch push's error names the withdrawal (`destroy.go:1758-1764`, `1838-1874`). Traces to G3.
- **C6:** The board push's unanswered-failure wording names what origin took, that the board was deleted, and that the records branch is being returned (`destroy.go:1778-1788`). Traces to G3: this path also lands and rolls back the records branch, which the entry's wording gap covers.
- **C7:** `pushCloneSeed`'s doc comment and the two `doc.go` paragraphs (`destroy.go:1717-1718`, `doc.go:266-268`, `doc.go:540`). Forced by C1 through C6, which make the old text stale.
- **C8:** The table rows in `clone_singlerepo_integration_test.go` and the wording assertions in `clone_singlerepo_killedpush_integration_test.go`. Forced by C1 through C6.
- **C9:** Commit f7d565cb6 wraps comments that ran too long and adds the unborn-board case; commit a32d529bd splits a test comment into two sentences. Both are forced edits to C3 and C8.

**Per decision:**
- G1 is built by C1, with C3 and C4.
- G2 is built by C2.
- G3 is built by C5 and C6.
- G4 is built by no change; none is needed, since it is not code.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point that asks for code has a change that builds it.

**Tool calls:** none.
