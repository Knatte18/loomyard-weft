---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D3 (round gate sentence), D1 rationale
    summary: The Webster-Burler round gate reads no merge base; it diffs from the plan verify's last recorded pass, so no Go-run gate ever checks a branch's card-phase growth against the line, and D1's premise fails
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (never-blocked sentence), D5 rationale
    summary: After a Publish verify failure the round gate's base is the pre-merge pass, so main's merged-in growth counts as the branch's growth, blocking branches that never touched PATTERN.md and demanding a cut to the line, not under the cap
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 Bound, D3 Bound
    summary: Neither Bound states what the round gate's no-usable-base pass and a fork-skipped card gate let through, or what bounds that growth
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Scope (In, /proc item), D8, Acceptance criteria (/proc literal)
    summary: Three more test files read /proc/<pid>/stat raw (pairteardown, shedverbs, gatecli), so the acceptance line and D8's no-test-file-reads-/proc property fail with Scope's three-file list
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3, Constraints
    summary: The round gate gains a new failure cause, but no Decision or Constraint requires its finding to name the way forward or the refusal-spec row for the exhausted Webster-Burler gate to name it, per PATTERN-refusal-way-forward
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: D4 creates a background file for an entry that has none but never requires the entry to link it, so moved detail can become unreachable from the overview
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria (gofmt guard line)
    summary: D7 makes an unparsable file a failure naming the file, but no acceptance line covers that case
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D4 rationale, Auto-mode assumptions
    summary: D4 says the trim leaves room for a few entries; the auto-mode assumption says about two
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D7 (today gofmt -l lists sentence)
    summary: The six-file gofmt list is a perishable snapshot the plan writer cannot act on, since D7 says it is recomputed; it is already in support-log.md Q9
---

### [BLOCKING] The round gate has no merge base, so branch growth goes unchecked

**Location:** `_lyx/discussion/decision-record.md` D3 ("runs the same pattern function in-process from the merge base it already reads for the comment lint to HEAD") and D1's rationale ("With every branch's growth ending at or under the line").

**Issue:** The Webster-Burler round gate has no merge base to reuse.
`loomshed.NewVerifyGate` takes the whole-diff form only when `mergeBase` is non-nil, and only `internal/loomcli/darnwiring.go:127` sets `env.VerifyMergeBase`, for darn runs.
In a loom run the gate takes the round form (`internal/loomshed/verifygate.go:72-91`): its lint base is `verifytree.LatestPass` of the plan verify command, resolved through `impactset.Derive`.
Webster's plan-level verify gate (`internal/websterengine/verifygate.go`) runs the same plan verify through `verifytree.Verify` into the same verify dir (`verifytree.Dir(anchorPath)` in both `internal/hubgeom/webstergeom.go:38` and `internal/loomcli/wiring.go:558`).
So the first Webster-Burler round diffs from the commit where implementation's verify passed, and every later round diffs from the previous round's pass.
The round gate sees only the fixer's changes and never the cards' growth.
The only check on the cards' growth is the card gate, which Go derives but never runs (`internal/websterengine/cardgate.go` header: "Go never runs it, so a fork that skips it is caught by the plan-level verify gate").
The plan-level verify gate runs no lint at all.
D3's rationale also names "a branch's change since its merge base" as a scope the comment lint already defines, which is true only for darn.
D1's claim that two branches cannot breach the cap therefore rests on a premise no Go-run gate enforces.

**Fix:** Decide where the branch-scoped check runs, and state it in D3.
One option is a round gate that reads the task branch's merge base with the parent for the headroom lint, with the reader darn already has.
Another is a headroom check in webster's plan-level verify gate.
Then align D1's rationale, the Constraints line on `PATTERN-test-speed`'s gate table and the round-gate acceptance line with the chosen base.

### [BLOCKING] Merged-in parent growth blocks the branch after a Publish failure

**Location:** `_lyx/discussion/decision-record.md` D2 ("a branch that does not grow the overview is never blocked, even while main sits above the line after a merge"), D5 rationale.

**Issue:** Publish keeps its merge-in commit on a verify failure (`internal/landingshed/doc.go:22-27`), and the way forward is `lyx loom goto --to Webster-Burler` (`contracts/specs/refusal-spec.md:304`).
The round gate's base is then the last pre-merge pass, which is an ancestor of the merge commit, so the diff from it to HEAD includes everything merged in from main.
Take a branch that never touched `PATTERN.md`, with main above the line after other landings.
Its round gate sees the overview grow from the branch's old content to main's, ending above the line, and fails.
The same happens to any Publish failure, a test-budget collision included, so the headroom lint reintroduces the Publish-time blocking the Goal sets out to stop.
For a merge-only hard-cap breach, the gate's finding demands a cut to the line (D2), not the bytes over the cap that D5's rationale says the fixer needs.
D2's guarantee is false under D3's own wiring.
D5's "exact amount to cut" names the wrong target.

**Fix:** Base the headroom lint on the merge base with the parent branch.
After a merge-in, that base holds main's content, so a non-growing branch passes and a growing branch is held to its own growth.
Or, if the pre-merge base is kept, state in D2 that a merge-in counts as growth, and reconcile D5's rationale with the line the gate enforces.

### [MEDIUM] The no-usable-base pass has no stated bound

**Location:** `_lyx/discussion/decision-record.md` D2 Bound, D3 Bound.

**Issue:** D3 makes the round gate pass the headroom lint when it has no usable base.
In the round form that happens when no pass of the told command is recorded, or when the recorded pass is not an ancestor of HEAD, for example after a rewrite (`internal/impactset/derive.go:43-49`).
In those rounds, growth of any size passes the round gate.
A fork that skips its card gate likewise lets its card's growth through.
D2's Bound lists only "growth that ends at or under the line" and "size change on main made outside a gate".
D3's Bound covers only the verb's writes.
Neither states these bypasses or what bounds them: the hard cap in `TestOverview_RealFilePassesCheck`, which every full verify runs.

**Fix:** Add both bypasses to a Bound, with the hard cap as their limit, and settle them against whatever base F1's fix chooses.

### [MEDIUM] The /proc scope disagrees with its acceptance line

**Location:** `_lyx/discussion/decision-record.md` Scope (In, the `/proc` item), D8, Acceptance criteria ("No `_test.go` file outside `internal/testkit/tmuxkit` names the `"/proc"` path literal").

**Issue:** Scope lists three files and claims every test file outside the kit then reads `/proc` only through the kit.
D8's rationale makes the same repo-wide claim, and the acceptance line checks it with one grep.
Three more test files read `/proc/<pid>/stat` directly today:
`internal/pairteardown/teardown_integration_test.go:201`, `internal/shedverbs/loop_integration_test.go:115` and `internal/gatecli/test_integration_test.go:242`.
With Scope as written, the acceptance line fails.
These are the same parent-pid parser copies D8's `PATTERN-testkit` argument covers.

**Fix:** Decide whether these files are in scope.
If they are, add them to Scope and D8.
If they are not, narrow D8's rationale and the acceptance line to the named files and record the exclusion under Out.

### [MEDIUM] The new round-gate failure has no way forward in the refusal spec

**Location:** `_lyx/discussion/decision-record.md` D3, Constraints.

**Issue:** The headroom finding is a new reason the Webster-Burler round gate fails and, after its attempts, halts.
The "Webster-Burler verify gate exhausted" row (`contracts/specs/refusal-spec.md:296`) lists each cause with its way forward: failing tests, a fixed-column comment, a misplaced guard marker.
`PATTERN-refusal-way-forward` requires the message to name the way forward, the spec row to land in the same commit, and a test to reach it.
The record mentions `refusal-spec.md` only in Notes for the plan writer, as "needs a look".
Notes are optional and non-binding, so nothing requires the change.
The verb's own failure gives no way forward either.

**Fix:** In D3, require the round gate's headroom finding to name its way forward: cut the named bytes from `PATTERN.md`, moving detail into background files.
Add a Constraints line for `PATTERN-refusal-way-forward` that names the spec row update and a test reaching the finding.

### [LOW] A newly created background file is not required to be linked

**Location:** `_lyx/discussion/decision-record.md` D4.

**Issue:** D4 moves detail into an entry's background file, "created when the entry has none".
It never requires the entry to gain its `— [background](pattern/…)` link.
D4 keeps each entry's trigger, requirement and `(test)` mark, but the link is how a reader reaches the moved detail.
Without it, the moved text is orphaned, and D4's "no rule changes" holds only on paper.

**Fix:** Add to D4 that an entry whose detail moves to a background file links that file.

### [LOW] No acceptance line for the guard's parse failure

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, the gofmt guard line.

**Issue:** D7 promises "A file that fails to parse is a failure naming the file".
The acceptance line covers a pass, an unformatted file, the `testdata` skip and the scan floor, but not a parse failure.

**Fix:** Add the parse-failure case to the gofmt guard's acceptance line.

### [NIT] Room left by the trim is stated two ways

**Location:** `_lyx/discussion/decision-record.md` D4 rationale ("leaves room for a few entries"), Auto-mode assumptions ("leave room for about two entries").

**Issue:** The two claims disagree.
The 564 bytes between 15000 and the line, against an average entry of about 220 bytes (16217 bytes over 74 entries), fits about two entries.

**Fix:** Keep one phrasing, in one place.

### [NIT] A perishable file list in D7

**Location:** `_lyx/discussion/decision-record.md` D7 ("today `gofmt -l .` lists …").

**Issue:** D7 says the list is recomputed at implementation time, and the acceptance line `gofmt -l` prints nothing already binds the outcome.
The snapshot gives the plan writer nothing to act on, and support-log.md Q9 already records it.

**Fix:** Drop the six-file list from D7 and keep "the files it flags at implementation time are fixed in the same change".

## Verdicts on the focus entries

- D2/D3 attack surface: no Decision states what the no-base pass lets through; see F3, and F1 for the base itself.
- D1 arithmetic: the claim is true as stated.
  With line L = 15564, cap C = 16384, headroom h = C − L = 820, and each branch's growth g ≤ L − b, a breach needs g₁ + g₂ > C − b.
  That forces L − b > h and each g > h.
  Not a defect in itself.
  The premise that every branch's growth ends at or under the line is the defect, in F1.
- D4: the requirement (trigger, requirement and `(test)` mark kept, nothing dropped or merged, no rule changed) is in D4 itself, so the plan writer has it without support-log.md.
  The missing link requirement is F6.
- D3/D5 against Out: Publish's routing itself is unchanged, but what the existing Webster-Burler way forward demands after a merge-in changes; see F2.
- D7/D8 against acceptance and Constraints: see F4 and F7.
  The Constraints add requirements, such as the gate table rows in `PATTERN-test-speed`'s background and the budget row, and contradict no Decision.
- Relocation: the Open risks, Auto-mode assumptions and Constraints carry no deliberation narrative beyond F8's duplicate.
  The D7 list in F9 is the one relocation.
