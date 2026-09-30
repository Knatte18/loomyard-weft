---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/reedengine/panebin.go:6
    summary: New and changed Go doc comments in panebin.go are fixed-column wrapped, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/reedengine/spawn.go:73
    summary: launchStrandLocked's rewritten doc paragraph and the comment above the send are fixed-column wrapped, and the doc's closing clause now dangles
  - id: F3
    severity: LOW
    location: internal/reedengine/replace.go:84
    summary: The new ReplaceStrand comment breaks mid-clause and its Otherwise sentence misstates when the record was never persisted
  - id: F4
    severity: LOW
    location: internal/reedengine/strand.go:317
    summary: removeStrandLocked's changed doc and the Down comment in lifecycle.go break mid-clause
  - id: F5
    severity: LOW
    location: internal/shell/shell.go:1
    summary: shell.go's file header and the new Source/ScriptExt docs in shell.go, posix.go and pwsh.go are fixed-column wrapped
  - id: F6
    severity: LOW
    location: internal/reedengine/doc.go:114
    summary: doc.go's changed pane-binary sentences share lines with other sentences and break mid-clause
  - id: F7
    severity: LOW
    location: internal/reedengine/spawn_test.go:6
    summary: New and changed doc comments in the changed test files are fixed-column wrapped
  - id: F8
    severity: LOW
    location: docs/sandbox-howto.md:24
    summary: The added sandbox-howto line breaks before a participle phrase, which is not an independent-clause boundary
  - id: F9
    severity: NIT
    location: internal/reedengine/launchscript_integration_test.go:23
    summary: readTrimmed does not trim, so its name misdescribes it and every caller trims by hand
  - id: F10
    severity: NIT
    location: internal/reedengine/panebin.go:103
    summary: writeLaunchScript mixes a fail closure with three inline os.Remove branches for the same temp-file cleanup
---

Range: `git diff 85902381e..HEAD`, the merge base of `main` (`product.parent` from `_lyx/shed/reed-launch-script/status.json`) and `HEAD`, five commits matching cards 1–5 one to one.
`go build ./...`, `go vet ./...`, `go test ./...` and `go test -count=1 -tags integration ./internal/shell/... ./internal/reedengine/...` all pass;
`TestLaunchScript_SourcedScriptRunsInThePaneShellScope` ran against a live tmux (not skipped) and passed.

The code follows the plan closely.
Every card-named test exists and asserts what its card says.
The findings are about comment and markdown line breaks, plus two small readability nits.

### [LOW] panebin.go comments are fixed-column wrapped

**Location:** `internal/reedengine/panebin.go:6` (file doc), `:41` (`composePaneLaunchLine`), `:57` (segment constants), `:64` (`launchScriptDir`), `:75` (`stageLaunchScript`), `:87` (`removeLaunchScripts`), `:98` (`writeLaunchScript`)
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention asks for one sentence per line, with further breaks only at independent-clause boundaries.
Every new or changed doc comment here wraps at a column instead.
The file doc packs "to apply it." and a new sentence on one line, then breaks mid-clause ("… written to, so the / pane types …").
`composePaneLaunchLine`'s rewritten opening breaks inside "followed by / launchCmd" and puts two sentences on one line.
Each new function doc breaks mid-phrase ("under the / state dir", "and / returns", "A / missing file", "The file is sourced, never / executed").
**Fix:** Rewrite each new or changed comment one sentence per line, breaking a long sentence only at a `, so`/`, and` or `;` clause boundary.
Rewrap `composePaneLaunchLine`'s whole doc, because its changed sentence runs on into the legacy lines.

### [LOW] launchStrandLocked comments are fixed-column wrapped and the doc's last clause dangles

**Location:** `internal/reedengine/spawn.go:73-81` (doc paragraph), `:139-146` (comment above the send)
**Issue:** Both rewritten comments wrap at a column and break mid-phrase ("see / composePaneLaunchLine", "the pane shell's / own scope", "key name (e.g. "Enter", / "C-c")").
The edit also left the doc's "rather than changing how the pane's shell itself starts" attached to "the pane remains tmux's own default-shell … (pane-start-mode-is-untouched Shared Decision)".
Before the edit, that clause contrasted the prelude riding a typed line with changing the shell's start.
Now it reads as if the split's missing argument were the alternative to changing the shell's start.
**Fix:** Rewrite both comments with semantic breaks.
Reattach the contrast to what it describes: the prelude reaches the shell through a typed source statement instead of changing how the pane's shell starts.

### [LOW] ReplaceStrand's new comment breaks mid-clause and misstates its condition

**Location:** `internal/reedengine/replace.go:84-86`
**Issue:** The first sentence breaks inside "the new / strand's record".
"Otherwise the record was never persisted" reads as the complement of "a nil applyErr".
But when the post-add save succeeded and `applyErr` is non-nil, the record *was* persisted, and the guarded code rightly keeps the script.
The comment's "otherwise" is narrower than it says.
The code is correct: `reconcileApplyPersistLocked` returns nil only after its own `SaveState` succeeds with the new strand still in `st.Strands` (`reconcileLocked` kills panes but never prunes the strand table).
Its early returns (list panes, reconcile, apply layout) all come before that `SaveState`, so they rightly count as never persisted.
**Fix:** Say what the condition actually guards: the post-add save failed and the tail's `SaveState` never succeeded, so the record never reached disk and its script goes.

### [LOW] removeStrandLocked and Down comments break mid-clause

**Location:** `internal/reedengine/strand.go:317-318`, `internal/reedengine/lifecycle.go:779-780`
**Issue:** The changed `removeStrandLocked` line holds the tail of one sentence and the head of another, breaking inside "every removed / strand".
The new `Down` comment breaks inside "has a strand left to / belong to" and shares a line between two sentences.
**Fix:** Put the new sentence on its own line in `removeStrandLocked`'s doc, and write the `Down` comment one sentence per line.

### [LOW] internal/shell comments are fixed-column wrapped

**Location:** `internal/shell/shell.go:1-4` (file header), `:45-50` (`Source`/`ScriptExt` interface docs), `internal/shell/posix.go:53`, `internal/shell/pwsh.go:55-59`
**Issue:** The shell.go header edit left a mixed shape: one over-long changed line and one legacy-width line, both breaking mid-sentence.
The interface docs and both `Source` docs break mid-phrase ("in the file at path in the / current shell's own scope", "so no / ExecutionPolicy").
**Fix:** Rewrite the header sentence and each new doc one sentence per line, with breaks only at clause boundaries.

### [LOW] doc.go's changed paragraph is fixed-column wrapped

**Location:** `internal/reedengine/doc.go:114-118`
**Issue:** The changed lines put "…as shell statements." and "The composed line, …, is" on one line, then break inside "is / written" and "the pane shell's / own scope".
**Fix:** Rewrite the changed sentences one per line.
Leave the untouched legacy lines after them as they are.

### [LOW] Test-file doc comments are fixed-column wrapped

**Location:** `internal/reedengine/spawn_test.go:6-8` (file header) and the new helper/test docs (`launchFake`, `readLaunchScript`, `breakSaveState`, the retargeted `TestLaunchStrandLocked_SendsThePreludeAheadOfTheStrandCommand`, and the seven new tests); `internal/reedengine/launchscript_cleanup_test.go:1-2` and `newCleanupEngine`; `internal/reedengine/launchscript_integration_test.go:3-7` and the test doc; `internal/reedengine/panebin_test.go:6`; `internal/reedengine/emptycmd_integration_test.go:10-15`
**Issue:** Same convention: these new and changed comments wrap at a column, break mid-phrase, or share a line between two sentences.
The spawn_test.go header mixes two over-long changed lines with a mid-sentence break.
**Fix:** Rewrite each new or changed comment with semantic breaks.

### [LOW] sandbox-howto.md breaks before a participle phrase

**Location:** `docs/sandbox-howto.md:24`
**Issue:** The repo's markdown rule allows a break inside a sentence only at an independent-clause boundary.
The added line starts "writing those statements …", a participle phrase with no subject of its own.
**Fix:** Join the participle phrase onto the preceding line, keeping the existing, legitimate break before ", so a nested `lyx …`".

The new CONSTRAINTS.md Pane Binary Resolution bullet breaks at two semicolons, each followed by a clause with its own subject and verb, so it complies.

### [NIT] readTrimmed does not trim

**Location:** `internal/reedengine/launchscript_integration_test.go:23-29`
**Issue:** `readTrimmed` returns the raw file content.
Its doc says "returns path's content", and every caller trims by hand (`strings.TrimSpace`, `strings.TrimRight`), so the name promises something the helper does not do.
**Fix:** Rename it to `readOrEmpty`, which says what it does.

### [NIT] writeLaunchScript's temp-file cleanup is spelled two ways

**Location:** `internal/reedengine/panebin.go:103-132`
**Issue:** The `fail` closure handles only the write error; the close, chmod and rename errors each repeat `_ = os.Remove(tmpName)` inline.
Every path is covered, so this is a readability issue, not a leak.
**Fix:** Use one deferred cleanup that removes the temp file unless the rename succeeded, and drop the closure and the inline removes.

## Focus directives settled without a finding

- **ReplaceStrand `postAddSaveFailed && applyErr != nil`:** correct on every branch (see F3 for the reasoning); only the comment is off.
- **removeStrandLocked deletes before persisting:** if `RemoveStrand`'s persist or `ReplaceStrand`'s tail fails, the removed GUID can stay in `reed.json` without its script.
  That is harmless.
  The CONSTRAINTS invariant says only that a script exists *only while* its GUID is in the table, which forbids orphaned scripts but allows a scriptless GUID.
  Every launch path (`Resume`, `--if-absent` relaunch, `UpdateStrand`'s surface) calls `stageLaunchScript`, which rewrites the script before typing its source statement.
- **Per-card mechanical checks:** card 1's `Shell` has no implementer outside `internal/shell` (grep for its method signatures finds only `posix.go`/`pwsh.go`).
  `removeLaunchScripts` returns nothing, and every call site adds it beside an existing return, so cards 3 and 4 change no op's result or error.
  For the Create groups, no reed state-dir `reed`/`launch` subpath constant existed before.
  `fabricengine/gitexclude.go`'s unexported `writeFileAtomically` is a near-equivalent 0o644 temp-and-rename helper (without `MkdirAll`) that card 2 did not record beside the `fsx.AtomicWriteBytes` exclusion.
  Card 2 mandates "a private helper in this file", so lifting a shared helper would be a plan-authoring point, not a diff defect, and is not flagged.
- **Windows `os.Rename` over an existing script:** Go's `os.Rename` on Windows replaces an existing target (`MOVEFILE_REPLACE_EXISTING`).
  pwsh's `Get-Content -Raw` closes its handle once it has read, and a script is rewritten only on a later launch of a new pane.
  A rename racing an open read would fail and degrade to typing the full line, never fail the launch.
  Not a defect.
