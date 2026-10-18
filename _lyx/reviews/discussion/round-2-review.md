---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D4 says a card cannot write Bolt, but lyx board writes Bolt from a task pair with no prime-only refusal, so the step could be carried by a board entry and the no-reminder risk could be closed
    fix: "In D4, replace the sentence starting A card cannot write either file with: A card cannot write either file: the only route that commits the hub-wide file is `lyx orch rules`, which this task ships, which is in production only after the deploy, and which runs only from the prime; the prime's own file is on `main-weft`, and weft content never lands from a task pair. After the sentence starting So the operator moves the rules, add: A card files a backlog board entry through `lyx board`, slug `orch-rules-move`, whose body holds this step and the text below verbatim, so the step is on the board when the task lands. In Open risks, delete the sentence starting Nothing at landing reminds the operator. In support-log.md's Rejected alternatives, reword the bullet starting A card writes the moved rules to: a card can reach Bolt only through `lyx board`, and the verb that commits the rules file is not deployed until the task lands."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:70
    summary: D3 sends a failed commit to the next board sync, which is the sweep into an unrelated commit that D3's rationale rejects, and it claims pull-first board writes skip on any dirty Bolt, which pullFirst does not do
    fix: "In D3, replace the sentence starting A failed commit leaves the new content in place with: A failed commit leaves the new content in place, and the envelope reports it as written but uncommitted, with re-running `lyx orch rules --from <file>` as the way forward, since that run commits the one path under the verb's own message. In Acceptance criteria, extend the criterion starting A failed write leaves the hub-wide file at its prior content so its second half reads: and a failed commit is reported as written but uncommitted, naming the re-run as the way forward."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:69
    summary: D3's restore inside the write closure guards nothing, because a temp-file-and-rename write that fails has not replaced the file, and no step after the rename can fail inside the closure
    fix: "In D3, delete the sentence starting A failed write restores the prior content inside the write closure. In Notes for the plan writer, reword the first bullet to: `reportHubWrite` in `internal/configcli/hubwide.go` shows the uncommitted report; D3 commits through `PullThenCommitWritten`, the pull-first sibling of the `CommitWritten` that `newHubCommit` uses. Keep the criterion that a failed write leaves the hub-wide file at its prior content."
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:51
    summary: D2 states the level-three heading rule twice, and the second time it can be read as the render shifting headings
    fix: "In D2, change the sentence starting Each file's text is inlined verbatim to: Each file's text is inlined verbatim."
---

### [MEDIUM] D4 rests on a false premise about Bolt and leaves the reminder gap open

**Location:** `_lyx/discussion/decision-record.md:89`, `_lyx/discussion/decision-record.md:139`, `_lyx/discussion/support-log.md:63`

**Issue:** D4 says "A card cannot write either file: the hub-wide file is on Bolt", and the support log's rejected alternative says a task pair "reaches neither Bolt nor `main-weft`".
`lyx board` writes Bolt, and nothing stops it from running in a task pair: `internal/boardcli` has no prime check, and `internal/shellguard/rows.go` has no board row.
D4's conclusion still holds, but for a different reason.
The only route that commits `<BoardDir>/_lyx/orch/rules.md` is `lyx orch rules`.
This task ships that verb, so it is not in production until the deploy, and the pre-run in `internal/orchcli/cli.go` refuses it outside the prime.
The false premise hides a mitigation.
The rubric does not bar a run from writing another board entry.
So a card can put the step and its verbatim text on the board, and the Open risks line "Nothing at landing reminds the operator of the step" no longer has to stand.
Round 1's fixer left that line in as a disclosure because it found no write surface for a landing hook.
A board entry needs no landing hook.

On the focus question: the operator route holds on every landing path, PR-Gate approve included, because it needs no orch turn.
The orch does get a turn at each landing, through the `<slug>: landed` notice that `## After landing` in `contracts/stencils/orch/orch-template-role.md` describes, but that notice carries no record, so D4 is right that no landing path gives `ly:orch` a turn with this record.
No step has the orch run `lyx orch stop`.
The one stop/start sequence is in Open risks, and the operator runs it from the prime.
`start --continue` exists in `internal/orchcli/start.go`.

### [MEDIUM] D3's failed-commit path contradicts its rationale and overstates the pull skip

**Location:** `_lyx/discussion/decision-record.md:70`

**Issue:** D3's rationale says a hand-edited Bolt file is a problem because `Bolt.Commit` "stages every change in Bolt's repo, so the edit would be swept into an unrelated board or sync commit under another message".
The failed-commit path then names "the next board sync" as the recovery, which is that sweep.
Re-running the verb is a cleaner recovery that keeps the verb's own message.
The run first pushes with `Push`, and `PushCoalesced` returns early with nothing unpushed.
`pullFirst` then fast-forwards over the modified file unless upstream touched it.
Then `StageAndCommit` commits the one path.
The clause "until then the pull-first board writes skip on a dirty Bolt" is also false.
`pullFirst` in `internal/fabricengine/bolt.go` returns early when Bolt is not behind its upstream, and it skips as dirty only when `MergeFFOnly` or the seed drop fails.
A dirty Bolt that is not behind writes normally.

The other D3 points the focus names hold:
- Unchanged content: `StageAndCommit` returns `committed == false` when `git diff --cached --quiet` exits 0, so it commits nothing and is no error.
- A retry after a failed push: the first `Push` sends the earlier commit, because `HasUnpushed` sees it ahead of `@{u}`.
- The failed-write restore: `rollBack` in `configcli` runs inside the closure, and `PullThenCommitWritten` returns write and commit errors after the closure.

### [LOW] D3's restore on a failed write is a dead path

**Location:** `_lyx/discussion/decision-record.md:69`, `_lyx/discussion/decision-record.md:167`

**Issue:** D3 already chooses a temp-file-and-rename write.
`fsx.AtomicWriteBytes` in `internal/fsx/fsx.go` creates the temporary file, writes and closes it, and then renames it over the target, so every failure before the rename leaves the target as it was.
Unlike `setHubWide`, the rules write has no validation or loader step after the rename that could fail inside the closure.
"Restores the prior content … and removes the file when it was absent" therefore describes a scenario the write cannot produce, and the plan writer would build restore code and fake-seam tests for it.
The acceptance criterion holds through the atomic rename alone and stays.

### [NIT] D2 states the heading rule twice

**Location:** `_lyx/discussion/decision-record.md:48`, `_lyx/discussion/decision-record.md:51`

**Issue:** Line 48 gives the moved preamble the sentence "a rules file's own headings start at level three (`###`)".
Line 51 repeats it after "inlined verbatim", where it can read as the render demoting headings.
The preamble already tells the writer the rule, which settles the focus question for both files.
On the other half of that question, D3's help text and D5's placement sentence both say the verb replaces the whole file.

## Rejected

- D1's `PrimeRoot`: `filepath.Dir(RepoRulesPath)` is `location.WorktreePath()` today (`internal/orchcli/paths.go`), so a told `PrimeRoot` joined from `WorktreePath()` keeps `isRequestedNote` resolving exactly as it does now. Not a defect.
- D3's bound: it names the real mechanism, the working-directory check `refuseNonPrime` behind `RequireDrivableWorktree` in the orch pre-run, and claims no shell-guard row. The temporary file is covered, because the bound is about what the verb commits, and D3 states that a killed write can leave the temporary file behind. `configcli`'s editor route (`copyStagedToHub`) and its `rollBack` restore already have the same exposure, writing Bolt config files through `fsx.AtomicWriteBytes`, which puts its `.tmp-` file beside the target. The temp-root scan bans only system temp directories, so no PATTERN is violated. Not a defect.
- The Linux re-exec bound in Open risks matches `internal/orchengine/watchloop.go`: the binary check runs only after a tick that succeeded, and the consecutive tick-error cap ends the watcher. Not a defect.
