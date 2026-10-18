---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:78
    summary: D4 routes the rules move to ly:orch at parent review, but a parent-review request reaches the orch only on an alignment escalation or no verdict, and a PR-Gate approve lands and deploys with no orch turn, so the move has no route and the rules vanish once ORCHESTRATOR.md is deleted
    fix: In D4 replace "a step it takes from this record at parent review" with a route that holds on every landing path, and name it in the record. One that fits the rubric is this task's own run_settings setting loom.alignment_pr_approve to false, set through lyx board before the run reaches PR-Gate (run_settings are writable, unlike the entry's body), so PR-Gate reaches ly:orch, which then takes D4's step after the landed notice. If run_settings do not reach that key, name the operator as the actor instead. Either way, add to the Open risks that until the step runs the role shows the hub-wide placeholder, and that the text survives in this record and in git history.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:80
    summary: D4 step 1 has ly:orch run lyx orch stop, which removes the orchestrator strand, the orch session itself, so it never reaches start --continue or step 2; its premise is also false on Linux, where the watcher re-execs onto a replaced binary at its first idle tick
    fix: Drop step 1 from the orch's steps. Rewrite the Open risks bullet at line 121 so it says that on Linux the watcher re-execs onto the deployed binary at its first restart-safe idle tick, which bounds the refusal window to the time between the stencil reseed and that tick, and that on Windows, where selfexec never re-execs, the operator restarts the orch with lyx orch stop then lyx orch start --continue from the prime. Step 2 is then the orch's only step.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:60
    summary: D3 restores the file on a failed commit as configcli's rollBack does inside its write closure, but configcli restores only on a validation failure inside the closure and reports a failed commit as uncommitted; CommitWritten commits after the closure returns, so a restore runs outside the board write lock, with the new content possibly already staged
    fix: Replace line 60 with configcli's actual behaviour. A failed write restores the prior content inside the write closure (an absent file is removed). A failed commit leaves the new content in place, and the envelope reports it as written but uncommitted for the next board sync to commit, which until then makes the pull-first board writes skip on a dirty Bolt. Change the criterion at line 142 to match, so that a failed write leaves the hub-wide file at its prior content, and a failed commit is reported as uncommitted.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:34
    summary: D1 removes RepoRulesPath, but the watcher's isRequestedNote also uses its directory as the prime worktree root to resolve a relative note path, and neither new path's directory is that root
    fix: Add to D1 that Paths gains a told field for the prime's worktree root, joined in orchPaths from location.WorktreePath(), and that the watcher's relative-note resolution reads it in place of filepath.Dir(RepoRulesPath).
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:98
    summary: D3 replaces the whole hub-wide file with --from's content, and D5 tells the orch to add a rule through lyx orch rules without saying to start from the current file, so a draft that holds only the new rule silently deletes every existing hub-wide rule on Bolt and its remote
    fix: In D5, make the role stencil's placement sentence say the draft starts as a copy of the current hub-wide file at its Bolt path, which the hub-wide placeholder or section names, and that the verb replaces the whole file. Add the same sentence to the verb's help text in D3.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:61
    summary: D3 commits through CommitWritten with no pull first, so when the Bolt copy is behind an upstream change to the rules file, the local commit conflicts at push, and PushRecorded retries only after dropping seed commits, so the next Bolt push does not carry it but hits the same conflict
    fix: In D3, write through Bolt.PullThenCommitWritten with the one rules path in place of CommitWritten. When Bolt cannot be brought up to date, nothing is written and the refusal names the result's SkipDetail as the way forward. Update the plan-writer note naming configcli's shape so it names the pull-first sibling.
  - id: F7
    severity: MEDIUM
    class: decision
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:57
    summary: D3 leaves open whether the unchanged-content path pushes, so a retry after a failed push reports success with no commit while the earlier commit stays unpushed
    fix: In D3, decide that the verb pushes on every successful run, unchanged content included, and that the envelope reports whether Bolt was left with unpushed commits. Add a criterion that a retry after a failed push with the same content pushes the earlier commit.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:64
    summary: D3's bound says no session outside the prime reaches the verb, but the prime-only refusal checks the command's working directory, and no shell-guard row denies lyx orch, so any session that runs it with the prime as cwd can rewrite the text inlined into the orch's role and publish it to Bolt's remote
    fix: Reword the bound's third clause to say what holds. The verb runs only with the prime's worktree as its working directory, a check on the directory and not on the caller, so any session can reach it from there, as it can edit today's ORCHESTRATOR.md in the prime. What bounds the written text is the role preamble D2 moves (the rules add to the role, override none of it, and a contradiction goes to the operator). Keep the Open risk that orch-any-worktree must bound the verb again.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:55
    summary: D3's temporary file sits in <BoardDir>/_lyx/orch/, which no Bolt exclude pattern covers, so a process killed between create and rename leaves a .tmp file that the next whole-repo Bolt.Commit sweeps in, contradicting the bound that the verb writes no other Bolt path
    fix: In D3, state that the temporary file is created beside the rules file and that a killed write can leave it untracked in Bolt, the same exposure configcli's hub writes have, and reword the bound to say the verb commits only the rules path.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:48
    summary: D2 relies on each rules file's headings starting at level three, but nothing that writes a later rule is told so, neither D5's placement wording nor the preamble D2 moves
    fix: Add to D2's moved preamble, or to D5's role-stencil placement sentence, that a rules file's own headings start at level three (three hash marks).
---

The record follows the board entry's operator decisions: two layers, hub-wide first, with `ORCHESTRATOR.md` deleted.
The blocking defects are in D4, the post-landing move.
It is routed through a parent-review turn the orch normally never gets (F1), and its first step ends the orch's own session (F2).
If both stand, the landing deletes the two operator rules and nothing ever writes them to Bolt.
D3's failure paths are next: they cite configcli for behaviour configcli does not have (F3), skip the pull-first write Bolt offers for this case (F6), and leave the retry-after-failed-push path undecided (F7).

### [BLOCKING] D4 has no route that reaches the orch after landing

**Location:** `_lyx/discussion/decision-record.md:78`

**Issue:** The role stencil says "A parent-review request reaches you only when the alignment agent escalated or gave no verdict" and sends that request to a one-shot fork (`contracts/stencils/orch/orch-template-role.md:41,43`).
`CLAUDE.md` says the Land row deploys after "one the alignment agent's approve at PR-Gate makes with no orch turn".
On that normal path, nothing gives the orch this record or D4's step.
The support log's Turn 3 "fixed" entry for this step rests on the same premise.

### [BLOCKING] D4 step 1 removes the orch that runs it

**Location:** `_lyx/discussion/decision-record.md:80`

**Issue:** `stopCmd` removes the recorded orchestrator strand (`internal/orchcli/stop.go:40-44`), which is `ly:orch`'s own session, so steps 1b and 2 never run.
The restart is also unneeded on Linux.
`lyx orch watch` re-execs onto a replaced binary at a restart-safe idle tick (`internal/orchengine/doc.go:397-407`, `internal/orchengine/watchloop.go:211-212`).
It never re-execs on Windows (`internal/selfexec/identity_windows.go`), so the restart is the operator's step there.
Open risks line 121 inherits the same false premise.

### [MEDIUM] D3's restore on a failed commit cites behaviour configcli does not have

**Location:** `_lyx/discussion/decision-record.md:60`

**Issue:** `setHubWide` calls `rollBack` only inside its write closure, after a validation failure (`internal/configcli/hubwide.go:206-226`).
A commit error goes to `reportHubWrite` as "uncommitted ... the next board sync commits it" (`hubwide.go:186-193`).
`CommitWritten` stages and commits only after `write` returns, and releases the lock when it returns (`internal/fabricengine/bolt.go:48-63`).
So a restore after a commit failure runs unlocked, a concurrent whole-repo `Bolt.Commit` can sweep the new content in first, and a staged index keeps it.
Under the Fabric-git constraint, the criterion at line 142 cannot be met.

### [MEDIUM] D1 drops the field the watcher uses as the prime root

**Location:** `_lyx/discussion/decision-record.md:34`

**Issue:** `isRequestedNote` resolves a relative note path against `filepath.Dir(w.paths.RepoRulesPath)` (`internal/orchengine/watcher.go:586-595`), which is `location.WorktreePath()` (`internal/orchcli/paths.go:39`).
That is a second role of the field, not a call-site count.
The directory of `OwnRulesPath` is `<anchor>/_lyx/orch` and that of `HubRulesPath` is in `_board`, so a mechanical swap misresolves relative replies and leaves the cycle's note unmatched.

### [MEDIUM] Adding a rule through the verb deletes the existing ones

**Location:** `_lyx/discussion/decision-record.md:98`

**Issue:** D3 copies `<file>`'s whole content over the hub-wide file.
D5 tells the orch to put a new workaround there "written through `lyx orch rules`".
A draft holding only the new rule replaces the Production and Watching-runs rules, then commits and pushes the result with no warning.

### [MEDIUM] D3 writes over a stale Bolt copy

**Location:** `_lyx/discussion/decision-record.md:61`

**Issue:** `PullThenCommitWritten` is documented as "CommitWritten's pull-first sibling for callers that must not write over a stale copy" (`internal/fabricengine/bolt.go:170-171`).
`PushRecorded` retries a conflicting push only after dropping seed commits (`bolt.go:82-94`).
A rules commit that conflicts with an upstream change to the same file is no seed commit, so "for the next Bolt push to carry" is false: every later Bolt push fails on the same conflict.

### [MEDIUM] Unchanged content after a failed push

**Location:** `_lyx/discussion/decision-record.md:57`

**Issue:** `CommitWritten` never pushes, and `Push` is a separate call (`internal/fabricengine/bolt.go:48-67`).
D3 says unchanged content "commits nothing and reports success" but not whether it pushes.
After a failed push, the natural retry with the same draft takes that path: it can report success with no commit while the earlier commit stays unpushed.

### [MEDIUM] The prime-only bound is a working-directory check

**Location:** `_lyx/discussion/decision-record.md:64`

**Issue:** The pre-run resolves the location from the command's cwd and compares worktree names (`internal/orchcli/cli.go:124-139`, `internal/orchcli/refusal.go:13-24`).
No shell-guard row denies `lyx orch` (`internal/shellguard/rows.go`).
The bound's clause "no session outside the prime reaches it" overstates what bounds the verb.
Under the rubric's attack-surface rule, the Decision must state the actual bound and what it lets through.

### [LOW] The temporary file is a second Bolt path

**Location:** `_lyx/discussion/decision-record.md:55`

**Issue:** `fsx.AtomicWriteBytes` creates `.tmp-*` beside the target and removes it only through a `defer` (`internal/fsx/fsx.go:60-65`).
Bolt's `info/exclude` and `.gitignore` match neither the rules file nor the temporary file, and `Bolt.Commit` stages everything (`internal/fabricengine/recordsgit.go:362`).

### [NIT] The `###` heading rule has no carrier

**Location:** `_lyx/discussion/decision-record.md:48`

**Issue:** Files are inlined verbatim under the two H2 sections, so a `##` heading in a later rule breaks the role file's section structure.
D4's draft follows the `###` rule, but nothing tells a future writer to.

## Focus verdicts

- D3's bound. Publishing any readable file is stated in the bound and holds as stated. The empty-file clear is stated, and D2 and its criterion agree with it. The temporary-file write is not bounded (F9). The prime-only clause misstates its own mechanism (F8).
- D4's Production exception. Not a defect. The rule is the operator's own text, moved verbatim, so the move adds no new escape hatch. Its own text bounds it to "after your own pushed fix, on a clean `main`", and only the worktree on `main` pushes to `main`, so widening it to every orch of the hub lets no other thread deploy. The Open risk leaves whether it still holds to the operator.
- D2 against D1 and the criteria. The placeholder, blank-file and unreadable-file behaviours and the marker names `hub_rules` and `orch_rules` are consistent across D1, D2, D3 and the criteria. Open risk 1's claim about an edited board copy holds, since that copy lacks the new markers.
- Relocation. No requirement the plan writer needs sits only in `support-log.md`, and no deliberation narrative in `decision-record.md` needs to move.
- Own board entry. No Decision changes this task's board entry's body or scope. D5 edits `CLAUDE.md` and the role stencil, and D4 writes a Bolt file outside the board. F1's suggested fix writes only the entry's `run_settings`, which the rubric permits.

## Rejected

- No limit on the content or size of `--from` (bound-breaker, LOW): the input is the operator's or their delegate's draft, and today's `ORCHESTRATOR.md` was inlined with the same lack of a limit. A size bound designs for a requirement nobody asked for.
- Pull-first board writes skip on a rules file left dirty after a failed commit (existing-paths, LOW): folded into F3, whose fix names it.
- Windows watcher never re-execs (existing-paths, LOW): folded into F2, whose fix makes the Windows restart the operator's step.
