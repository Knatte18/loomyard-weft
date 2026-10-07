---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/websterengine/beginbatch.go:83
    summary: Changed comments still sit inside legacy hard-wrapped sentences at about twenty sites across the diff; the fix pass reflows only the changed lines, so PATTERN-comment-line-breaks is still violated
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/destroy.go:50
    summary: The recording-contract comment pins "the nine executors", but the diff adds updateRemoteBranch and the executor table now holds ten
  - id: F3
    severity: LOW
    class: consistency
    location: internal/gitrepo/push.go:2
    summary: The rewritten file header pins "All six" push-shaped calls, a tally that line 1's own list already states
  - id: F4
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-rubric-plan-review.md:31
    summary: The Do-not-flag entry still bounds the planparser checks at the last-row ID commit-subject-mismatch and says "except plan-unapproved" before the next line excludes delete-target-gone
  - id: F5
    severity: LOW
    class: consistency
    location: contracts/specs/refusal-spec.md:224
    summary: The Publish push-rejected row's condition names only a remote task branch holding commits the local branch lacks, but pushRejectedReason also reaches the failed-remote-read case with the merge way forward
---

### [BLOCKING] Changed comments still sit inside legacy hard-wrapped sentences

**Location:** the sites below, all new or changed comments after the merge base.

**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line, breaking inside a sentence only before an independent clause (a comma before a coordinating conjunction, or a semicolon).
A changed line inside a legacy wrapped sentence makes the whole sentence a changed comment (the focus directive's rule).
A script over every comment block of the diff's `.go` files found each multi-line sentence that holds a changed line.
After excluding sentences whose breaks fall at a semicolon or a `, so`/`, and` boundary, these remain hard-wrapped or broken at a non-clause boundary:

- `cmd/lyx/destructiveguard_test.go:41-47`: "Its blind spots are deliberate and / significant: …", and the two sentences after it, are wrapped mid-phrase; only line 41 was reflowed.
- `internal/fabricengine/doc.go:562-568`: the "The vocabulary is `Kind` (…)" sentence; line 563 gained `remote_branch_updated`, `worktree_reset`, and the rest of the sentence stays wrapped.
- `internal/fabricengine/doc.go:681-682`: the break falls before the non-coordinated clause "which refuses on any tracked dirt …".
- `internal/planparser/doc.go:95-98`: "This package's parse step is lenient … so Validate can enumerate every defect in one / pass …"; only line 98 changed.
- `internal/websterengine/beginbatch.go:7-8`: "Under the flat card-list model … no oversized-batch / escalation, and …".
- `internal/websterengine/beginbatch.go:83-94`: the `BeginDeps` sentence; line 90 is a new item, and items `Batches` and `Geom` stay wrapped mid-phrase.
- `internal/websterengine/beginbatch.go:243-245`: the `BeginBatch` doc sentence wraps after "Master forks".
- `internal/websterengine/beginbatch.go:333-339`: "A no_report re-fork never / calls begin-batch again …"; line 333 is the changed line.
- `internal/websterengine/beginbatch_test.go:3-8` and `:95-99`: file header sentence and `beginFixture` doc sentence.
- `internal/websterengine/doc.go:247-263`: the Agent-tool-fork paragraph, several sentences, wrapped throughout, with line 247 and line 260 changed.
- `internal/websterengine/doc.go:355-359`: "This crash window resumes on the SAME / machine only: …"; line 355 is the changed line.
- `internal/websterengine/runlevel.go:764-771`: the cycles comment, wrapped mid-phrase, with two sentences on line 767 ("read first. Non-done outcomes …").
- `internal/websterengine/runlevel_test.go:936-938`: the `TestRun_MasterSpawn` doc sentence.
- `internal/webstercli/cli_test.go:377-381`: the `seedTwoCardGlyphPlanDir` doc, two sentences wrapped across five lines.
- `internal/webstercli/wiring.go:324-325`: the break falls before the participial phrase "shared by both wireHub and wireStandalone …".
- `tools/sandbox/suite.go:128-129`: the `websterSuite` doc sentence.

The same class of defect was fixed in earlier rounds site by site, and each fix pass left the legacy remainder of the sentence wrapped.
That is a gap in the method (reflow the changed line), so it is filed once as a `design` finding and not once per site.

**Fix:** For every comment above, reflow the whole enclosing sentence to one line, breaking only at the clause boundaries the rule allows, and split lines that hold two sentences.
Detect the sites mechanically rather than by reading: walk each changed `.go` file's comment blocks, split them into sentences at lines that end in `.`, `?`, `!` or `:`, and report every multi-line sentence containing a changed line.
Re-run that detector over the whole diff after the fix and expect an empty list apart from semicolon and `, so`/`, and` breaks.

### [LOW] The recording-contract comment pins a stale executor count

**Location:** `internal/fabricengine/destroy.go:50`

**Issue:** "every one of the nine executors below" is a perishable tally that this diff made false: `updateRemoteBranch` is a tenth executor (the table in `cmd/lyx/destructiveguard_test.go` lists ten).
The same diff removed the equivalent "nine" from the guard test's own comments.

**Fix:** Write "every executor below".

### [LOW] The push.go header pins a count of calls

**Location:** `internal/gitrepo/push.go:2`

**Issue:** "All six are push-shaped remote calls" repeats a count that line 1's own list defines, and the diff had to bump it from five.

**Fix:** Write "All of them are push-shaped remote calls".

### [LOW] The rubric's Do-not-flag entry bounds a list at a last-row ID

**Location:** `contracts/stencils/loom/loom-rubric-plan-review.md:31-32`

**Issue:** Line 31 names the planparser checks "from `format-unrecognized` through `commit-subject-mismatch`", a range that ends at a row ID and goes stale when a planparser row is added after it, and its "except `plan-unapproved`" leaves `delete-target-gone`, a listed check the gate never runs, to the next line.
Line 17 of the same stencil already states the gate's coverage without a range: "every check but `plan-unapproved` and `delete-target-gone`".

**Fix:** Reword line 31 to "Every check ID … lists is enforced deterministically by this round's own gate over this round's own output, except `plan-unapproved` (…) and `delete-target-gone` (…)", and keep line 32's reason for `delete-target-gone` as its clause.

### [LOW] The Publish push-rejected row does not cover the failed-read case

**Location:** `contracts/specs/refusal-spec.md:224`

**Issue:** `pushRejectedReason` has three outcomes.
The two spec rows cover remote commits the local branch lacks, and no commit or no remote branch.
When `Deps.RemoteOnlyCommits` itself errors, the reason keeps the rejection, names the cause and gives the `git merge origin/<task-branch>` way forward, which the first row's condition ("because the remote task branch holds commits the local branch lacks") does not describe.
`TestPublish_StuckBeforePullRequest` reaches that case, so the spec row is the only gap.

**Fix:** Widen the first row's condition to "the remote rejects the task branch's push, and the remote task branch holds commits the local branch lacks or its tip could not be read", or add a row for the failed read.

## Focus departures

- Directive item 5, "do `expiredShellWarning` and `backgroundShellSentence` both survive as separate functions with one wording between them, and is the friction note's call site the single user of the shell wording?": `backgroundShellSentence` no longer exists; round 2's F5 folded it into `expiredShellWarning`, which now holds the one wording.
  I judged the surviving shape instead.
  `expiredShellWarning` has two users, `runlevel.go`'s per-outcome warnings and `triage.go`'s friction note, and that is the intended design (the warning and the note cannot drift), so there is no finding.
- Directive item 2 (no live text names Master's last-asserted model): no live text does.
  The only remaining mentions of `assertedModel` are `state_test.go`'s regression rows that pin the removed field loading unchanged and never being written back.
- Directive item 4: `pushRejectedReason`'s empty-tip branch says "the remote has no task branch" and does not assert the branch exists, the second spec row's condition covers it, and the `push rejected with no remote task branch …` row of `TestPublish_StuckBeforePullRequest` reaches it; no finding on the empty-tip case itself (F5 concerns the failed-read case beside it).
