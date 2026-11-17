---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 bullet 2 and Notes for the plan writer bullet 2
    summary: D6 makes fabricengine.RegenerateHubWorkspace write a beam start task naming a lym verb into the workspace files, extending fabric's IDE knowledge, which PATTERN-fabric-boundary forbids
    fix: >-
      Rewrite D6's beam start bullet so fabric learns nothing new of IDEs or lym verbs.
      State one design in D6: the beam start tasks reach the workspace files through a general seam fabric offers,
      such as a workspace-content hook installed process-wide the way PairFolderNamer is installed, filled by the ide module,
      or the workspace files' tasks key is written by the ide module after fabric's regeneration and preserved by it.
      Add a Constraints line for PATTERN-fabric-boundary naming that fabricengine gains no knowledge of tasks, verbs or the lym binary.
      Update the Notes bullet on RegenerateHubWorkspace to match, and rewrite the Scope bullet on VS Code tasks to name the writer the design picks.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 bullet 1 and the D5 subdirectory acceptance criterion
    summary: The board subtree's PersistentPreRunE resolves every subcommand with strict lymcwd.Resolve before RunE, so lym board open never reaches ResolveWithin and the subdirectory criterion cannot pass
    fix: >-
      In D5 bullet 1, state that the board subtree's PersistentPreRunE (internal/boardcli/cli.go) resolves the open verb with lymcwd.ResolveWithin
      instead of lymcwd.Resolve, and that it still loads the board config the same way, so the verb reads the board directory as the other board verbs do.
      Drop the contradictory phrasing that implies both resolutions apply.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 bullet 5
    summary: D4's link allowlist keeps any relative path but never defines one, so javascript, data, mixed-case schemes and scheme-relative //host targets have no stated bound
    fix: >-
      In D4 bullet 5, define the kept targets: a target whose scheme, compared case-insensitively after trimming leading whitespace and control characters,
      is http or https, or a target with no scheme that does not start with // or a backslash; every other target renders as plain text.
      State that the HTML escape covers double quotes, so an escaped target cannot leave its href attribute.
      Extend the D4 Review criterion to name the javascript, data and //host cases.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 bullet 2 and Notes for the plan writer bullet 2
    summary: D6 leaves to Plan which lym binary path the beam start tasks run; selfexec.LymBinary refuses under go test and ideengine falls back to bare lym, but the record states neither choice
    fix: >-
      In D6's beam start bullet, state the command's binary: the absolute path selfexec.LymBinary returns in the writing process,
      falling back to bare lym when it errors, as vscode.WriteConfig does for reed attach, and the same for the board open task.
      Replace the Notes sentence that the content must come from inputs every caller has with the settled inputs:
      the prime's and each beam's anchor, which the regeneration already reads through lymcwd.NamedWorktree, and the binary path above, read by the writer F1's design picks.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 Bound
    summary: D6's bound omits that an older lym on the hub rewrites the workspace files, tasks.json and the keybindings block whole, stripping beam start, board open and both keys on every regeneration or seed
    fix: >-
      Add to D6's Bound: a binary older than the change that regenerates the workspace files or seeds or spawns rewrites them without the beam start tasks,
      the board open task and the two keys, and the next regeneration or seed by an up-to-date binary restores them, as D3 states for the board files.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 bullet 1 against D5 bullets 2 and 4
    summary: The board open task never reveals its terminal, so D5's refusal naming lym board rerender and a failed opener's error are invisible when the operator presses alt+shift+9
    fix: >-
      In D6 bullet 1, set the board open task's presentation to reveal its terminal when the verb fails,
      through a reveal setting that brings the terminal forward on a non-zero exit, or state as a bound that a refusal is seen only in Tasks output;
      pick the first and say which presentation value carries it.
  - id: F7
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks bullet 1 and D7
    summary: Parent call needed on the phone goal; the entry body, revised after the landed layer grouping, still records collapsible sections per area and per body as not built, while D7 builds per-section folds only
    fix: >-
      Parent: confirm D7's collapsible Tasks, Notes and Done sections as the phone goal, or ask for per-area collapsible blocks inside each project.
      Once answered, the writer records the call in D7 and drops the Open risks line.
  - id: F8
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks bullet 2 and Scope Out bullet 1
    summary: Parent call needed on the Was ide-prime-setup part; commit 6b037083 dropped the folderOpen chain the entry asks to keep as reed up then reed attach, and the entry's LY window-title form differs from the landed titles
    fix: >-
      Parent: confirm that the prime's folderOpen chain and the entry's window-title form stay out of this task, or name which of them is still wanted.
      Once answered, the writer records the call in Scope and drops the Open risks line.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 Bound
    summary: The bound files the git-tracked tasks.json case under alt+shift+0, but beam start lives in the workspace files; a tracked tasks.json drops board open, so alt+shift+9 is the key that misses
    fix: >-
      In D6's Bound, move the git-tracked .vscode/tasks.json case to a sentence about alt+shift+9 and board open,
      and keep alt+shift+0's list to the task pair's bare-folder window, a bare-folder prime or beam window and a window seeded before the upgrade.
  - id: F10
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 bullet 1 and D4 bullet 3
    summary: D4 orders sections in the README's order, but D1's projection carries no section order and no promise the entry list is in README order, so the viewer must recompute what D1 says it never recomputes
    fix: >-
      In D1's top-level bullet, add the README's section order as a list in the projection, or state that the entry list is in the README's order within each project;
      D4 then groups by that order.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 bullet 4
    summary: D4 links every entry to its design file, but D1 names a design file only when it exists, which is only for an entry with a body
    fix: In D4 bullet 4, change every entry to its design file to an entry to its design file when the data names one.
  - id: F12
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 Bound
    summary: When xdg-open runs a browser in the foreground the board open task stays active, so a second alt+shift+9 meets VS Code's task-already-active prompt; the bound states the wait but not this effect
    fix: >-
      Extend D5's Bound with one sentence saying that while the verb waits on a foreground browser,
      the board open task stays active and a second key press is answered by VS Code's running-task prompt.
  - id: F13
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 bullet 3 and Bound
    summary: A downgrade's render removes both files and its sync commits the removal; the older binary's Claims does not claim them, so that commit replays as a whole-commit cherry-pick, not a by-entry merge, and D3 says it heals
    fix: >-
      In D3's Bound, replace the downgrade clause with: a downgrade removes both files and commits the removal,
      and since the older binary claims neither file, that commit replays as a plain cherry-pick rather than through the by-entry merge, and can conflict.
      In D3 bullet 3, replace dropping them as unclaimed with: a commit touching an unclaimed path is replayed as a plain cherry-pick.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D1 paragraph on encoding
    summary: D1 says a less-than sign is written as a less-than-style escape, which names no escape; Go's default HTML escaping writes the unicode escape for each of the three characters
    fix: In D1, replace written as a `<`-style escape with written as its `<`-style unicode escape (`<`, `>`, `&`).
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Scope Out bullet 1
    summary: The Out bullet carries history the plan writer does not act on, the landed commit ids, while Open risks bullet 2 leans on Scope to name them
    fix: >-
      In Scope Out bullet 1, keep only the excluded items: window titles, the folderOpen chain and launch.json.
      Move the commit ids b0e9f535 and 6b037083 into Open risks bullet 2, which must name the landed commits, until F8 settles it.
---

Round 1 review of the board-viewer discussion artifact pair against the Discussion-Review rubric.
The design is coherent and most claims about existing code hold;
one PATTERN constraint is broken, and several bounds have gaps.

### [BLOCKING] D6 extends fabric's IDE knowledge

**Location:** `_lym/discussion/decision-record.md` D6 bullet 2; Notes for the plan writer bullet 2
**Issue:** `PATTERN-fabric-boundary` says fabric "learns nothing new of IDEs, shuttle, the board or heddle".
It also says an import fabric already holds of such a module is debt, and a change that touches it "moves the knowledge out to the module, and never extends it".
`internal/fabricengine/hubworkspace.go` already imports `internal/vscode` to build the workspace files.
D6 makes `fabricengine.RegenerateHubWorkspace` also write a `tasks` key naming `lym beam start <beam>` and a lym binary.
That is new IDE knowledge and lym verb knowledge inside fabric.
The record's Constraints do not name `PATTERN-fabric-boundary`.
This answers the focus question on D6 in part: the regeneration has the anchors (it reads them through `lymcwd.NamedWorktree`), but placing the task there breaks the constraint.

### [MEDIUM] `lym board open` cannot run from a subdirectory

**Location:** `_lym/discussion/decision-record.md` D5 bullet 1; D5 subdirectory criterion
**Issue:** `internal/boardcli/cli.go:125-152` runs `lymcwd.Resolve(cwd)` in the subtree's `PersistentPreRunE` for every subcommand but `board` and `intake`, and aborts on error before `RunE`.
D5 says the verb resolves with `ResolveWithin` and also reads the board directory "as the other board verbs do"; both cannot hold.
Raised by all three of walkthrough, interactions and bound-breaker; I read the pre-run myself and confirm it.

### [MEDIUM] D4's link allowlist leaves "relative path" undefined

**Location:** `_lym/discussion/decision-record.md` D4 bullet 5
**Issue:** The focus asked whether the escaping order and the link allowlist close `javascript:` and `data:` targets.
The order (escape, then subset) is stated and sound, and D1's `</script>` handling holds.
The allowlist has a gap: "a relative path" is undefined.
A naive test lets `javascript:` or `data:` through, a case-sensitive test passes `JavaScript:`,
and `//host/share` resolves to a `file://` UNC target on Windows from a page opened at `file://`.

### [MEDIUM] D6 leaves the beam start command's binary to Plan

**Location:** `_lym/discussion/decision-record.md` D6 bullet 2; Notes bullet 2
**Issue:** D6 says `board open` runs the absolute lym binary, but says only that `beam start` "runs `lym beam start root`".
The Notes hand Plan the rule that "the tasks' content must come from inputs every caller has" without naming those inputs.
That is a deferred fact the design rests on.
The facts are findable now:
- `selfexec.LymBinary` gives the path and refuses a test binary (`internal/selfexec/binary.go:19`);
- `ideengine` falls back to bare `lym` on that error (`internal/ideengine/spawn.go:231`, `internal/vscode/config.go:71-73`);
- the regeneration already reads every anchor.

The existing-paths lens reported the same gap at LOW; it is kept here at MEDIUM as a deferred research question.

### [MEDIUM] D6 has no mixed-version bound

**Location:** `_lym/discussion/decision-record.md` D6 Bound
**Issue:** Every `RegenerateHubWorkspace` caller rewrites the workspace files whole, and every seed or spawn rewrites `tasks.json` and the keybindings block whole.
The callers include fabric verbs through `regenerateAfterVerb`, `anchorchange.go:539` and `loomcli/pawlwiring.go:116`.
An older binary on the hub therefore strips the new tasks and keys.
D3 bounds the same exposure for the board files, but D6 does not bound it.
Raised by existing-paths and bound-breaker.

### [MEDIUM] Board open's refusal is invisible from the key

**Location:** `_lym/discussion/decision-record.md` D6 bullet 1 against D5 bullets 2 and 4
**Issue:** D6's `board open` task does not reveal its terminal.
D5's refusal (no `board.html`, way forward `lym board rerender`) and a failed opener's error then print to a terminal the operator never sees.
Neither decision bounds this.

### [MEDIUM] Parent: the phone goal

**Location:** `_lym/discussion/decision-record.md` Open risks bullet 1; D7
**Issue:** The focus asked whether each Open risks line is a parent question.
This one is: it is doubt about what the task is for.
Its premise needs a correction, though.
The entry body was revised after the landed work it lists, and it still records that "the collapsible `<details>` sections per area and per body are not built".
So per-area folding is not plainly outdated.
Per-body folding is moot, since bodies moved to `design-<slug>.md`.
D7 rests on no unverified fact about GitHub app `<details>` rendering:
the operator asked for `<details>` for the app, and the README already wraps each project in `<details open>` (`internal/boardengine/render.go:4`).

### [MEDIUM] Parent: the `Was ide-prime-setup` remainder

**Location:** `_lym/discussion/decision-record.md` Open risks bullet 2; Scope Out bullet 1
**Issue:** This is a parent question too.
Commit `6b037083` ("drop the folderOpen chain") contradicts the entry's "Prime folderOpen chain: `reed up` then `reed attach` only".
The entry's window title `LY: prime` differs from the landed titles: `b0e9f535` covers console titles, and `consoletitle.Workspace` writes `[<shortname>] <name> workspace`.
A landed commit contradicting the entry is the parent's call.

### [LOW] D6's bound misattributes the tracked tasks.json case

**Location:** `_lym/discussion/decision-record.md` D6 Bound
**Issue:** `ideengine` skips a tracked `tasks.json` (`internal/ideengine/spawn.go:246-255`), which drops `board open` and `reed attach`.
That affects `alt+shift+9`, not `alt+shift+0`, whose task lives in the workspace files.

### [LOW] The projection carries no section order

**Location:** `_lym/discussion/decision-record.md` D1 bullet 1; D4 bullet 3
**Issue:** D4 groups "by section in the README's order", but D1 hands the viewer only section names and the type-label order.

### [LOW] D4 links design files that may not exist

**Location:** `_lym/discussion/decision-record.md` D4 bullet 4
**Issue:** A design file exists only for an entry with a body (`internal/boardengine/render.go:2`), and D1 names it only when it exists.

### [LOW] A second key press while xdg-open blocks

**Location:** `_lym/discussion/decision-record.md` D5 Bound
**Issue:** While the verb waits on a foreground browser, the task stays active, and VS Code answers a second run with its running-task prompt.

### [LOW] D3's downgrade bound misstates the replay

**Location:** `_lym/discussion/decision-record.md` D3 bullet 3 and Bound
**Issue:** `internal/fabricengine/replay.go` sends a commit through the by-entry merge only when every path it touches is claimed; otherwise the commit joins the cherry-picks.
On a downgrade, the older binary's render removes both files through the manifest, and its sync commits the removal.
That binary's `Claims` does not claim them, so the commit replays as a cherry-pick, which can conflict.
D3 says each case heals on the next write and calls unclaimed paths "dropped".
The bound-breaker's own counterexample (machine A's commits on origin) is wrong: classification runs over the local commits only.
The downgrade path is the one that holds, so the finding is kept at LOW with that counterexample.

### [LOW] D1 names no escape

**Location:** `_lym/discussion/decision-record.md` D1 paragraph on encoding
**Issue:** "written as a `<`-style escape" names no escape.
Go's `encoding/json` default writes `<`, `>` and `&`.

### [NIT] The Out bullet carries history

**Location:** `_lym/discussion/decision-record.md` Scope Out bullet 1
**Issue:** The commit ids are deliberation evidence.
The plan writer needs only the excluded items.
Open risks bullet 2 needs the ids, since a parent question names its commits.

## Focus verdicts

- D6 and its Notes: F1 and F4.
- D5: the path is bounded.
  It is the board directory's `board.html` only, `--board-path` is hidden and internal, and the opener gets only that absolute path.
  The cwd resolution is broken (F2).
- D4 and D1: escape order and `</script>` handling hold; the link allowlist has a gap (F3).
- Open risks: both lines are legitimately `decision` (F7, F8); D7 rests on no unverified fact.
- D3: the owner-set claim holds.
  `renderOwnerWrite` returns every name in `Render`'s map, and `CommitWritten` commits those paths (`internal/boardengine/ownerset.go:130-155`, `286-321`).
  `FabricReplay.Claims` is the replay hook (`internal/fabricengine/boltreplay.go:147`).
  The claim is not deferred.
  Its downgrade bound is F13.
- Relocation: only the Out bullet's history (F15).
  The D3 Bound is a stated bound, not narrative.
  The Open risks commit list is required, since a parent question names its commits.
- Acceptance criteria: every id from D1 to D7 appears as a whole word.
  Each criterion is checkable behaviour; D4's is a review check and carries `Review:`.

## Rejected

- Bound-breaker F1 as filed (machine A's commits carry unclaimed paths for machine B): replay classifies only the local commits, so origin's commits are never classified by B; the downgrade case is kept as F13.
- Bound-breaker note that `lym board rerender` does not commit under SkipGit or SkipPush: a test or offline configuration, not the hub default the way forward serves; the fork did not file it either.
- Interactions finding 7 (Scope names only `lym ide seed` and `lym ide spawn` as writers): merged into F1, whose fix rewrites that Scope bullet to name the writer the design picks.
- Existing-paths finding 2 (binary path for beam start): merged into F4.
