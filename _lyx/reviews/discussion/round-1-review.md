---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:63
    summary: D2 vets and lists deps only under the integration tag so a tmux or llm tagged change passes the round gate uncompiled, which contradicts D1's any-gate llm row and is not stated as a bound
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D4 publish_verify runs an arbitrary shell command read from the task's own per-worktree landing config, and the Decision does not state that a task diff can empty or replace it, which silently drops the tmux gate
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:107
    summary: D5 makes the round gate skip every unmarked repo-scanning test and states no bound on a missed marker, and no acceptance criterion requires the existing scan guards to be marked
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:129
    summary: D7 scopes the lint to whole blocks containing a changed line and flags any line of the block, so untouched legacy wrapped lines are flagged against the rule that untouched lines are never rewrapped
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:132
    summary: D7's finding rule treats a colon line end as a sentence end though D6 allows only semicolon and comma-conjunction breaks, and a sentence end followed by a closing paren or quote is flagged as a false positive
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:59
    summary: The diff scope leaves renamed and deleted files unmapped in D2, and D7's round-gate lint has no base when D2 falls back to the full verify
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:215
    summary: D2's empty-set behavior has no matching acceptance criterion
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:84
    summary: D3 states no retention rule for per-command entries, whose round-command keys differ every round, and its exposed latest pass sentence does not say what is selected or returned
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:63
    summary: D2 builds one shell command that lists the impacted set four times, which can pass the Windows cmd length limit for a large set, and the command text is also the record key
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:207
    summary: The requirement that the publish_verify value lands only after the tmux tier is green sits in Open risks rather than in D4 or Constraints
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:51
    summary: Several Rationale, Auto-mode and Open risks lines are unsupported tallies, incident narrative or duplicates of other sections and carry no requirement for the plan writer
---

### [MEDIUM] Round gate compiles only the integration tag

**Location:** `_lyx/discussion/decision-record.md:47`, `:63`, `:69-77`.
**Issue:** D2's command order runs `go vet -tags integration` and `go list -deps -test -tags integration`.
`go build ./...` compiles no test file and no tagged file, so a change to a `tmux`- or `llm`-tagged file, or a change that only a `tmux`/`llm` test file imports, is compiled by no round-gate step.
Today the plan verify vets every tag at each round (`PATTERN-test-speed`: each tag compiles on its own); the plan verify of D4 keeps that, but only Webster's gate and Publish run it, and Webster's gate precedes the Burler rounds.
So a compile break in a tagged file written during a round surfaces first at Publish.
D1's last row ("Any gate: never the `llm` tier, which only `go vet -tags llm` compiles") reads as if every gate vets `llm`, which the round gate does not.
D2's closing bound names only what the import graph cannot see, not this gap.
The cost is one rework round, as D2 says for the graph gap, but the Decision does not state it for tags.
**Fix:** Either run `go vet` over the set under `integration`, `tmux` and `llm` (vet compiles without running, so it is cheap) and compute the set with `go list -deps -test` under all three tags, or add the tagged-file gap to D2's bound next to the graph gap.
Reword D1's last row so it says which gates vet `llm`.

### [MEDIUM] `publish_verify` has no stated bound on who sets it or what it runs

**Location:** `_lyx/discussion/decision-record.md:94-103`.
**Issue:** D4's bound is "the key runs only at Publish, and an empty value runs nothing".
That does not say what the key can run or who can change it.
`landing` is not `HubWide` in `configreg`, so `landing.yaml` is the task worktree's own tracked `_lyx/config/landing.yaml`, loaded from the anchor path in `internal/loomcli/wiring.go`.
A task's own diff can therefore set `publish_verify` to empty (the tmux gate is silently dropped) or to any shell command, run by `verifyrun` through `sh -c` with the worktree's rights.
The plan's `## verify:` is the same kind of command, but Plan-Review reviews it; no step reviews a config edit before Publish reads it, and Finalize's config-change notice comes after the landing.
**Fix:** State in D4 what the key can run (any shell command in the task worktree, the same trust class as `## verify:`) and what bounds a task-edited value, for example reading it from the parent branch's copy, or naming the review that sees a `landing.yaml` change.
If the accepted bound is that a task can empty it, say so.

### [MEDIUM] `//lyx:guard`: a missed marker is unbounded and unenforced

**Location:** `_lyx/discussion/decision-record.md:107-112`, `:213-229`.
**Issue:** The round gate used to run the plan verify, which ran every scanning guard.
D5 makes a scanning guard run in a round only when marked, and states the control as "review discipline".
It does not state what an unmarked guard lets through (a rule violation in the round's change passes every round and is caught at Webster's gate or Publish, one rework round later) or what bounds it.
D2's closing paragraph covers graph-invisible reads generally, not the marker.
The parenthetical list in D5 is not exhaustive: scanning tests also sit in `cmd/lyx`, `internal/fabricengine`, `internal/pairteardown` and `internal/testkit`, and no acceptance criterion requires the existing ones to be marked, so the plan writer can mark only the listed files.
**Fix:** State the bound in D5 (caught at Publish, never on `main`).
Add an acceptance criterion that every existing repo-scanning guard carries the marker, and name how the plan writer finds them (a test that reads repository files outside its own package directory).
State which build tag a marked test runs under, since a marked guard in a tagged file is not run by the gate's untagged and `integration` commands.

### [MEDIUM] D7 flags untouched lines of a touched block

**Location:** `_lyx/discussion/decision-record.md:129-133` against `:118-121` and `pattern/PATTERN-comment-line-breaks.md:9`.
**Issue:** D7's scope is blocks "that contain at least one added or changed line", and its finding is "a line of a block" that breaks wrongly, so any line in the block.
A one-word edit in a legacy fixed-column-wrapped comment makes every wrapped line of that block a finding, and the fixer must rewrap lines nobody changed.
The PATTERN line D11 keeps ("untouched lines are never rewrapped: the rule governs new and changed comments only") and D7's own "untouched blocks are never flagged" say the opposite of that outcome at line level.
**Fix:** Flag a line only when the line or its successor is added or changed in the diff, or state in D7 and D11 that a touched block is rewrapped whole and amend the PATTERN line to say so.
Pick one, since D6 and D7 land together.

### [LOW] Finding rule's terminators disagree with D6 and misfire on closing punctuation

**Location:** `_lyx/discussion/decision-record.md:132` against `:118`.
**Issue:** D7 counts a line ending in `:` as a sentence end, so a break after a colon passes.
D6 allows a break inside a sentence only after a semicolon or a comma before a conjunction, and says nothing of a colon.
D7 also lists `.`, `!` and `?` as the only sentence ends, so a sentence ending `.)`, `."` or `.` followed by a closing backtick is flagged, though D7's own rationale rejects false positives that block a correct comment.
**Fix:** Say in D6 whether a colon break is allowed, and in D7 allow a terminator followed by closing punctuation.

### [LOW] Diff mapping for renamed, deleted and base-less cases is open

**Location:** `_lyx/discussion/decision-record.md:59`, `:129-131`.
**Issue:** D2 maps "each changed `.go` file" to its directory's package.
A renamed file changes two packages when it moves between directories, and a deleted file's directory may no longer be a package, so "nearest enclosing directory that is a Go package" and the fifth fallback condition do not say what happens.
D7 counts "added or changed" lines, so a pure rename without rename detection makes every comment line of the moved file an added line.
At the round gate D7's diff is D2's base to HEAD, but D2 falls back to the full verify exactly when no base exists, and then the lint has no diff.
**Fix:** State that the diff uses rename detection and maps both sides of a rename, that a deleted file maps to its nearest surviving package directory or to the fallback, and what the round-gate lint diffs against when D2 has no base (pass vacuously, or diff against the parent merge-base).

### [LOW] Empty-set behavior has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:66`, `:215-228`.
**Issue:** The fallback conditions are matched by the third criterion, and the order and the skip rule have criteria, but the empty set (no changed package, no fallback: only `go build ./...` and the guard tests) has none.
**Fix:** Add a criterion: a diff that touches only a `.md` file outside any package runs `go build ./...` and the guard tests and no package tests.

### [LOW] D3 retention and the exposed lookup are unstated

**Location:** `_lyx/discussion/decision-record.md:84-87`.
**Issue:** The record keeps one entry per command.
The round command changes with the impacted set, so each round writes a new key; D3 says a round pass never evicts the plan-verify entry but gives no rule for any other entry, so the record grows without bound or evicts in an unspecified way.
"The latest pass of a given command, tree and commit" does not say whether the lookup is keyed by command and returns the tree and commit, or is keyed by all three.
**Fix:** State the retention rule (for example one entry per command, round entries replaced by the latest, plan-verify entries kept) and reword the lookup as "keyed by command, returns the latest entry's tree and commit".

### [LOW] D2's command line grows with the set

**Location:** `_lyx/discussion/decision-record.md:63`.
**Issue:** The set is listed in each of four `go` commands, joined into one shell command that is also the `Verify` record key.
This module has about 120 packages whose import paths total about 6 KB, so a large set repeated four times exceeds `cmd /C`'s length limit on Windows, where `internal/fslink` and `verifyrun` both support running.
**Fix:** State that a set covering the whole module (or above a stated size) runs as `./...`, so the command stays bounded.

### [LOW] The tier-green ordering is an Open risk, not a constraint

**Location:** `_lyx/discussion/decision-record.md:207`.
**Issue:** "The Plan must not land D4's config value before the tier is green" is a requirement on the plan writer's card order, and it sits in Open risks, whose bullets describe what might go wrong.
D4 states that the tier is made green in this task but not that the config value comes after.
**Fix:** Move the ordering into D4 as a sentence ("the card that sets `publish_verify` follows the card that makes the tier green"), and keep Open risks to the risk.

### [LOW] Narrative and duplicated lines in the decision record

**Location:** `_lyx/discussion/decision-record.md:51`, `:143`, `:166`, `:198`, `:201`, `:209`, `:211`.
**Issue:** None of the following carries a requirement the plan writer needs, and `support-log.md` already holds or can hold each.
- D1's "A full suite count per run drops from 7–9 to 2–3" is a tally stated nowhere else, with no derivation.
- D7's "three bugfix runs ended Webster review unconverged" and D9's "67–90 s versus 60 s idle" are incident and measurement narrative; the support log carries the second.
- The first Auto-mode assumption repeats D2's first fallback and D7's last bound.
- The fourth Auto-mode assumption repeats D9's own-measure statement.
- The fourth Open risk is Q10 of the support log (an operator question outside the repo); its clause "PATTERN wins inside lyx's prompts" is the only requirement in it.
- The fifth Open risk repeats D2's last paragraph.
Completeness check: the only requirement in this set is the PATTERN-wins clause.
**Fix:** Extract "PATTERN's reading governs lyx's reviewers, fixers and lint" into D6 as a bullet, then drop the fourth and fifth Open risks, the repeated Auto-mode assumptions, the D1 tally and the D7 incident clause; the D9 numbers move to the support log, where the rejected-alternatives entry already has them.
