---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/09-button-click-binding.md (bindingArgvs, composeButtonCommand) and _lym/plan/08-reed-button-verb.md (runButtonVia step 5, composeConfirmedButtonLine)
    summary: tmux run-shell does not format-expand its -c start directory, so the click branch starts the verb in the server's fallback directory instead of the run directory, and the confirmed line's -c with doubled hash signs names a wrong directory
    fix: "In card 9, drop -c from the button branch so it reads run-shell -b \"#{E:@lym_button_command}\", and have composeButtonCommand start the pinned line with a change-directory statement whose directory word is #{q:@lym_button_dir}, left for the click's expansion as #{q:session_name} is, chained before the discarded verb words and the forced zero status; add to card 1 the Shell form that changes to a directory given as an already-quoted word, with its row in card 1's test. In card 8, drop -c from the confirm-before command (run-shell -b <tmuxQuote(line)>) and have composeConfirmedButtonLine start with sh.ChangeDir(dir), each # doubled like every other pinned token. Card 9's tier2 test keeps its pwd assertion, which now pins this. Ask the parent to record in the decision record that tmux 3.6 run-shell expands its command but not -c, since D7 spells run-shell -b -c '#{@lym_button_dir}'."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/05-third-status-line.md (Edit group)
    summary: card 5 sets status to 3 but misses TestSmokeSelvage, whose subtest asserts readOption("status") reads 2 and fails after the card
    fix: Add `internal/reedcli#TestSmokeSelvage` to card 5's Edit group and a Tests bullet saying its status assertion and the comment above it read 3 (the three-line bar); validate-plan --overlay admits this target.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/04-geometry-button-context.md (Tests, TestGeometryOverRealHub step)
    summary: the tier1 rows setting only parent_worktree to another live pair and to gone-beam never reach buttonBeam's parent-anchor or empty-Anchor branch, since ParentOf answers the prime whenever parent_branch equals the prime's checked-out branch
    fix: In card 4's TestGeometryOverRealHub step, have the live-pair and gone-beam rows also set the origin's parent_branch to a branch the prime has not checked out (the live pair's own branch for the first row, a branch name no worktree holds for gone-beam), and say why in the step.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/06-render-button-row.md (buttonSpec strand field) and _lym/plan/08-reed-button-verb.md (runButtonVia step 3)
    summary: the spec's strand flag is true for every window agents entry, so card 8 refuses or requires a strand name for an agent-window entry that uses no {strand}, wider than D7, which refuses only a {strand} entry
    fix: In card 6, set buttonSpecEntry's strand to true only for an entry whose run or confirm holds {strand} after the static fill, and change the TestPinButtonRowLocked row to a window agents entry with {strand} giving strand true plus a window agents entry without {strand} giving strand false while still drawn only in the agent row. Card 8's step 3 keys on that flag unchanged.
  - id: F5
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-reed-button-verb.md (runButtonVia steps 5 and 6)
    summary: the gone-run-directory check runs after the confirm branch, so a confirm entry whose run directory is gone raises the prompt and refuses only after the operator answers y
    fix: In card 8, move step 6 (run directory not an existing directory) ahead of step 5, so an unconfirmed click on a confirm entry shows the gone-directory message and raises no prompt, and add that case to the removed-run-directory row of TestRunButtonVia.
  - id: F6
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/08-reed-button-verb.md (Tests, TestRunButton_RealTmux)
    summary: two TestRunButton_RealTmux steps repeat TestRunButtonVia rows, the changed-spec confirmed request creating nothing and the pane without @lym_strand_name running nothing
    fix: In card 8, drop from TestRunButton_RealTmux the step that changes the spec's confirm run and repeats the confirmed request (TestRunButtonVia's confirmed-with-another-hash row covers it) and the step on a pane carrying no @lym_strand_name (TestRunButtonVia's empty-read row covers it); keep the real reads, the prompt, the run directory and the exit-code logging.
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/08-reed-button-verb.md (Intent, bound paragraph)
    summary: the bound paragraph names what --confirmed lets through but not --strand, which lets a direct caller fill {strand} with any grammatical strand name
    fix: In card 8's bound paragraph, add that --strand lets a direct caller fill {strand} with any name agentname.Parse accepts, quoted per value so it stays one word, which adds nothing over that caller running the command itself.
---

### [BLOCKING] run-shell -c is not format-expanded, so the click runs outside the run directory

**Location:** `_lym/plan/09-button-click-binding.md`, `bindingArgvs` and `composeButtonCommand`; `_lym/plan/08-reed-button-verb.md`, `runButtonVia` step 5 and `composeConfirmedButtonLine`.

**Issue:** Card 9 binds `run-shell -b -c "#{@lym_button_dir}" "#{E:@lym_button_command}"`, and card 8's confirm-before command is `run-shell -b -c <tmuxQuote(dir)> <tmuxQuote(line)>`, with `tmuxQuote` doubling `#` on the ground that run-shell format-expands its directory and command.
A probe against tmux 3.6 shows run-shell expands its command but not `-c`:
with a global `@g` of `/usr`, `run-shell -c '#{@g}' 'tmux set -g @o "$(pwd)"'` left `@o` at the home directory, while `run-shell 'tmux set -g @o2 "#{@g}"'` gave `/usr`.
The same held inside `if-shell -F` and with `-b`.
So the click starts `lym reed button` in the fallback directory, not the run directory.
The verb's log then lands outside the run directory's anchor, contradicting D7.
Card 9's tier2 test, which records `pwd` as the beam's anchor, fails.
In card 8, the doubled `#` makes a run directory that holds `#` name a path that does not exist.
The decision record prescribes the same `-c` form, so the fix is recorded through the parent too.

### [MEDIUM] Card 5 misses the smoke test asserting status 2

**Location:** `_lym/plan/05-third-status-line.md`, Edit group.

**Issue:** `internal/reedcli/smoke_selvage_keepalive_test.go:338` (in `TestSmokeSelvage`, `tier2`) asserts `readOption("status") != "2"` fails the test, and its comment says the bar has two lines.
Card 5 changes the session's `status` pin to 3 and lists no edit to this test, so the card's own gates fail on it.

### [MEDIUM] Card 4's tier1 parent rows never leave the prime branch of ParentOf

**Location:** `_lym/plan/04-geometry-button-context.md`, Tests, `TestGeometryOverRealHub`.

**Issue:** `ParentOf` (`internal/hubgeom/parentof.go:38-44`) returns `""` when the origin's `parent_branch` equals the main worktree's branch, before it reads `parent_worktree`.
The existing step this extends (`internal/hubgeom/geometry_integration_test.go:94-100`) rewrites only `ParentWorktree` and keeps hubforge's prime branch.
The planned rows that set `parent_worktree` to another live pair and to `gone-beam` therefore read the prime, and they never reach the parent-anchor branch or the empty-`Anchor` branch D4 requires.
The rows would fail on their own expectations.

### [MEDIUM] The spec's strand flag covers every agent-window entry, not only {strand} ones

**Location:** `_lym/plan/06-render-button-row.md`, `buttonSpec`; `_lym/plan/08-reed-button-verb.md`, step 3.

**Issue:** Card 6 sets `strand` "true for a `window: agents` entry".
Card 8 step 3 reads the clicking pane's `@lym_strand_name` for every `strand` entry and refuses when it is empty.
It also requires a confirmed call to carry a name `agentname.Parse` accepts.
D7 refuses only "a `{strand}` entry clicked while the active pane carries no `@lym_strand_name`".
D2 allows `window: agents` without `{strand}`.
Such an entry would be refused on a pane an older lym launched, though its command needs no name.

### [LOW] The gone-directory check comes after the prompt

**Location:** `_lym/plan/08-reed-button-verb.md`, `runButtonVia` steps 5 and 6.

**Issue:** Step 5 raises the prompt and returns before step 6 checks the run directory.
On a confirm entry whose beam was removed after the render, the operator answers a prompt for a click that then only shows the gone-directory message.

### [LOW] TestRunButton_RealTmux repeats two tier0 rows

**Location:** `_lym/plan/08-reed-button-verb.md`, Tests.

**Issue:** The real-tmux test's changed-spec step repeats `TestRunButtonVia`'s "confirmed call with another hash runs nothing" row.
Its no-name pane step repeats the "`{strand}` entry whose read answers empty" row.
Each runs the same `runButtonVia` branch, and the step before each already exercises the real tmux read, so the repeats add no behaviour.
The other real-tmux steps hold their place: the real `display-message -c` read of the active pane, the real prompt, the run directory, and `runButtonScript`'s Info logging, which no tier0 row reaches.

### [LOW] The --strand flag has no stated bound

**Location:** `_lym/plan/08-reed-button-verb.md`, Intent.

**Issue:** The attack-surface rule asks each new flag's card to state what it lets through and what bounds it.
The bound paragraph covers `--confirmed` and the prompt but not `--strand`.
The grammar check in step 3 bounds the value but does not say what a direct caller gains.

## Rejected

- Header comments edited outside the targeted members' spans, in cards 1, 4, 5, 7 and 9 (lens:repo-invariants finding 1, lens:affected-set finding 2): `planglyph.ScopeGuard` skips a changed file that holds a changed symbol or a covering target (`internal/planglyph/scope.go:113-115`), and its finding is informational. Each of these files has a targeted member the card changes, so the header edit is in scope.
- Card 9's "neither strand-name variable set" assertion (lens:guard-reach finding 3): it pins the run environment, which D7's acceptance criterion names as an outcome. An outcome assertion that a fixture also guarantees is no defect, and it costs no extra test.
- `shell_test.go`'s header "One test per Shell method" (lens:repo-invariants finding 2): `TestShell_ScriptExtAndDiscardLineKey` already breaks it, so the plan does not falsify it.
- `reedcli/cli.go`'s header naming `switch` as the one pre-run exception (lens:repo-invariants finding 3): it was already stale, since `follow` and `window` are exempt too (`cli.go:142`).
- Granularity of `tmuxQuote` and `buttonRunHash` (focus directive): both are private helpers serving card 8's verb alone, and they already sit in that card, so the granularity rule is met.
