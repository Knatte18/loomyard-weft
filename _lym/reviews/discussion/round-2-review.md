---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7 (lines 170-180), Goal (line 6), Scope In (line 18), Auto-mode assumptions (line 225), Acceptance criteria (line 260)
    summary: The main record still builds D7's README folds and keeps the phone half of the Goal, although the parent's added entry keeps D7 only if another goal needs it, and none does.
    fix: In decision-record.md remove D7 whole, the D7 acceptance criterion, the D7 auto-mode assumption, the Scope In bullet Collapsible Tasks, Notes and Done sections in every rendered board markdown file, and the clause and make the README easier to read on a phone in the GitHub app from the Goal. Leave every Added after Discussion entry untouched. In support-log.md, record under Rejected alternatives that collapsible README sections were dropped once the parent judged today's README good enough on the phone.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 third bullet (lines 138-147) and Constraints (lines 209-210)
    summary: D6 puts the splice of hook members and the refusal of the workspace keys folders and settings inside fabricengine, which extends fabric's existing VS Code knowledge, against the PATTERN-fabric-boundary rule that a change touching such an import moves knowledge out and never extends it.
    fix: In D6's third bullet, state that fabricengine.RegenerateHubWorkspace passes the installed hook's members through unread to internal/vscode, and that the vscode workspace builder (the code behind vscode.BuildWorkspace) splices them after settings and refuses a member named as a key it writes itself, so fabric names no workspace key; the beam file's generated mark, which fabric owns, stays refused in fabric. In the PATTERN-fabric-boundary Constraint, add that fabric's existing vscode import is not extended, with the workspace file's key knowledge kept in internal/vscode.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Open risks (lines 229-235)
    summary: Both Open risks bullets are settled by the parent's added entries, the phone line by The phone goal is met by today's README, which says the line is dropped, and the ide-prime-setup line by The prime's open chain and LY window title stay out.
    fix: Remove both bullets from Open risks in decision-record.md. Leave the section heading in place, with no bullets, unless the writer has another open risk to record.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 first bullet (lines 131-133), D5 (lines 112-118)
    summary: The claim that a reveal silent task comes forward when the verb exits with status 1 rests on VS Code behaviour with no source or version named, and D5 states no exit status for its refusal or a failed opener, so the visibility of a failing board open task is unproven.
    fix: Settle the fact now. Either cite the VS Code source file and release where a silent task's terminal is revealed on a non-zero exit, and add to D5 that its refusal and a failed opener exit with status 1, or change D6's board open presentation to reveal always with focus false and drop the exit-status sentence. Update the D6 acceptance criterion on the presentation to match.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 first and second bullets (lines 107-115)
    summary: D5 presents --board-path as a consequence of the resolved layout, when the flag bypasses resolution and config, and its refusal names an anchored directory that does not exist under the flag.
    fix: In D5's first bullet, replace the sentence ending so the subtree's --board-path flag points the verb at that directory's board.html with two statements, that without the flag the board directory comes from the resolved layout, and that --board-path bypasses resolution and the verb opens board.html in that directory. In the second bullet, state that under --board-path the refusal names lym board rerender alone, since no worktree was resolved.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 first bullet (line 108)
    summary: The verb runs from anywhere in a worktree overclaims, since lymcwd.ResolveWithin refuses a cwd outside the anchored directory of a subpath-anchored worktree.
    fix: In D5's first bullet, replace so the verb runs from anywhere in a worktree with so the verb runs from anywhere inside the worktree's anchored directory or its records sibling.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (line 43)
    summary: D1 names the literal characters as their own unicode escapes, which contradicts its claim that a body holding a script end tag cannot end the script.
    fix: In D1's escaping sentence, write the three escapes as the code spans <, > and &, and the style example as a <-style unicode escape.
---

The parent's added entries and the record's main sections now disagree on D7 and on Open risks, and D6's fabric seam extends fabric's IDE knowledge. Those two defects make the verdict BLOCKING.

Focus questions, settled:

1. D6 and the fabric boundary: the record keeps tasks, lym verbs and the lym binary out of `fabricengine`, and the Constraints name the rule. The splice and key check are another matter (F2).
2. D5's cwd: the record states one resolution, `lymcwd.ResolveWithin` for `open` in the board `PersistentPreRunE`, with every other verb keeping `lymcwd.Resolve`. The task's own cwd, the folder's anchor, resolves under either. The prose around `--board-path` and the "anywhere" claim are wrong (F5, F6).
3. D4's link allowlist: not a defect. The trim and tab/CR/LF removal, the case-insensitive `http`/`https` test and the double-slash-or-backslash rule reject `javascript:`, `data:`, mixed-case schemes, `//host` and `/\host`. Escaping before the subset runs, with double quotes escaped, keeps a target inside its `href`. Entity smuggling such as `&#58;` is defused because `&` is escaped first.
4. D6's Bound: it covers an older binary rewriting the workspace files, `tasks.json` and the keybindings block, and states how each heals. Whether a failing `board open` shows its output is unproven (F4).

### [BLOCKING] D7 survives a parent decision that removes its reason

**Location:** `_lym/discussion/decision-record.md` D7 (lines 170-180), Goal line 6, Scope In line 18, auto-mode line 225, criterion line 260.

**Issue:** The parent's entry "The phone goal is met by today's README" (lines 194-198) says the task builds no new README folds for the phone, and D7's sections stay only if another goal of the entry needs them. The board entry has two goals, the interactive viewer and the phone README. The viewer needs no README folds; D7's own rationale cites only the GitHub app and the phone scroll. The arbiter entry at lines 182-186 also kept D7 only "until the parent answers", and the parent has now answered. As written, Plan reads the main record and builds D7. Rejected alternatives in `decision-record.md` are by design absent, so the dropped design goes to `support-log.md`.

### [BLOCKING] D6's workspace seam extends fabric's VS Code knowledge

**Location:** `_lym/discussion/decision-record.md` D6 third bullet (lines 138-147), Constraints (lines 209-210).

**Issue:** `pattern/PATTERN-fabric-boundary.md`, Fabric isolation, says fabric gains no new knowledge of IDEs. It also says "An import fabric already holds of such a module is debt. A change that touches it moves the knowledge out to the module, and never extends it." `internal/fabricengine/hubworkspace.go` imports `internal/vscode` and leaves the workspace file's shape to `vscode.BuildWorkspace` and `vscode.WorkspaceSettings`; fabric names no VS Code key except its own `lym.generated` mark. D6 has `RegenerateHubWorkspace` itself splice raw members "after `settings`" and refuse members named `folders` or `settings`. That teaches fabric the VS Code workspace schema, which extends the debt. The no-tasks, no-verbs, no-binary half of the Constraint holds, and the seam's inputs (hub path, beam slug, anchor) are general. Only the placement of the splice and the key check breaks the rule.

### [MEDIUM] Open risks keeps two lines the parent has settled

**Location:** `_lym/discussion/decision-record.md` Open risks (lines 229-235).

**Issue:** The rubric says that a writer's Open risks line settled by an `Added after Discussion` answer is a consistency finding, and its fix drops the line. The phone bullet is settled by lines 194-198, which also say outright that the line is dropped. The `Was ide-prime-setup` bullet is settled by lines 200-204 and by the arbiter's launch.json entry (lines 188-192).

### [MEDIUM] Failure visibility of the board open task rests on an unsourced VS Code fact

**Location:** `_lym/discussion/decision-record.md` D6 first bullet (lines 131-133); D5 lines 112-118.

**Issue:** D6 sets `"reveal": "silent"`. It claims the terminal comes forward when the verb fails with exit status 1, so that D5's refusal and opener errors are seen. The record names no VS Code source or release for that behaviour. VS Code's task documentation words `silent` differently, as revealing when the output is not scanned for errors and warnings. No existing lym task uses `silent`: `writeAttachTask` in `internal/vscode/config.go` uses `always`. D5 states no exit status for its refusal or its failed opener. The discussion spec (stencil Step 2) requires such a fact to be settled now, with evidence named by source and version. This is the focus's fourth question, and the record leaves it unproven. The round-1 fixer report (fix-proposal 4) itself calls the VS Code rule unconfirmed. It argues the wording "holds under either reading", but under the documented reading a task with no problem matcher may never be revealed on failure, so the claim still rests on a fact nobody has checked. The fixer cites `clihelp.Abort(ctx, 1)` for the exit status, but D5 does not state that status.

### [LOW] D5 misstates how --board-path relates to the resolved layout

**Location:** `_lym/discussion/decision-record.md` D5 first and second bullets (lines 107-115).

**Issue:** In the board `PersistentPreRunE` (`internal/boardcli/cli.go`), `--board-path` takes the branch `cfg = boardengine.Config{Path: *boardPathFlag}` and resolves no layout, so `location` stays nil. D5 says that the layout sets the board directory "so" the flag points the verb at `board.html`, which reverses the relationship. D5's refusal names "the worktree's anchored directory", and under the flag no such directory has been resolved.

### [LOW] D5 overclaims "anywhere in a worktree"

**Location:** `_lym/discussion/decision-record.md` D5 first bullet (line 108).

**Issue:** `lymcwd.ResolveWithin` (`internal/lymcwd/lymcwd.go`) returns `ErrCwdOutsideAnchor` when the cwd lies in a worktree but outside its anchored directory. A subpath-anchored worktree's root is one such cwd, so the verb refuses there.

### [LOW] D1's escape sentence names the characters as their own escapes

**Location:** `_lym/discussion/decision-record.md` D1 (line 43).

**Issue:** The raw bytes of line 43 read "written as its `<`-style unicode escape (`<`, `>`, `&`)". That says each character is written as itself. It contradicts the sentence's own conclusion and the D1 acceptance criterion at line 242. Go's HTML escaping writes `<`, `>` and `&`. This is round 1's F14 again: the fixer report marks it "fixed as proposed", yet the backslash escapes did not reach the file. The fixer should check the raw bytes after the edit.

## Rejected

- The bound-breaker's NIT on D1's escape text is the same defect as the interactions lens's LOW, merged into F7 at LOW: the sentence states something false, not something merely untidy.
- The interactions lens's LOW that D5 states no exit status is folded into F4, since it is a premise of the same visibility claim.
- The existing-paths lens reported no findings; its traces agree with the record.
