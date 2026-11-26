---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-affected-set
    location: internal/lymcwd/anchor.go:115
    summary: The drift capture child, which lym starts in the anchored directory, embeds the gate's new kept-cd cause and its cd-and-run-again way forward in drift's halt reason, so that reason holds two way-forward clauses that contradict each other.
    fix: "Keep the kept-cd cause and the cd way forward out of the error a lym-spawned child reports. One way is to have checkCwdAnchorGate return the old two-path text and add the cause and way forward where an interactive verb surfaces the error, such as preflight.ResolveMode's ErrCwdOutsideAnchor branch in internal/preflight/predicates.go. The other is to have driftcli.runCaptureChild (internal/driftcli/cli.go:260) report only the opening text. Keep errors.Is(err, ErrCwdOutsideAnchor) true on both paths, and update TestCheckCwdAnchorGate's wantMessage row to match wherever the text ends up."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/lymcwd/anchor.go:117
    summary: The way forward prints cd followed by the anchored directory unquoted, so a path that holds a space fails when the command is run as written.
    fix: "In checkCwdAnchorGate's format string, wrap the second anchorAbs in double quotes so the clause reads way forward: cd \"<dir>\" and run the command again. Double quotes work in bash, PowerShell and cmd. Do not use %q, because it doubles Windows backslashes. Update the wantMessage of the subdirectory row in internal/lymcwd/gate_test.go to carry the quoted path."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/lymcwd/gate_test.go:77
    summary: No row pins the way forward's cd target for a subpath anchor, so a message that named worktreePath instead of anchorAbs would pass every row.
    fix: "Give the row named worktree root above a subpath anchor errors a wantMessage holding the whole refusal. Its cwd is home/user/repo, its anchored directory is filepath.Join(\"home\", \"user\", \"repo\", \"backend\"), and its way forward reads cd followed by that backend path."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/heddleengine/shipped.json:21
    summary: The lym-shell-directory entry says a lym verb run after a cd is refused, which is broader than the code, because ResolveWithin verbs accept a subdirectory and a plain repo's subdirectory resolves as standalone.
    fix: "In the lym-shell-directory text, narrow the first bullet so it says a lym verb run after the cd may be refused, far from the cd that caused it. Leave the heading and the second bullet as they are."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/heddleengine/store_test.go:188
    summary: The new ShippedEntriesArePinned row asserts the entry's heading wording, a new assertion on static prose of a shipped instruction.
    fix: "Change the lym-shell-directory row to {\"lym-shell-directory\", \"agents-file\", \"## \", true}, as the lym-glossary row does, so it pins slug, order, target and heading level but not the wording."
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:31
    summary: The merge-danger blast radius names lymcwd alone, but the diff also changes the shipped agent instructions heddleengine renders into every worktree's agent files.
    fix: "Set the Blast radius line in _lym/landing/summary.md to one word covering what breaks, such as agents. Its value must stay a single word, as loom-final-summary-spec requires."
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/lymcwd/gate_test.go:18
    summary: TestCheckCwdAnchorGate's doc comment still describes only the accept and reject triples and the ungated ResolveWorktree, not the whole refusal text the test now pins.
    fix: "Add one sentence to the TestCheckCwdAnchorGate doc comment saying that a row with wantMessage also pins the refusal's whole text, cause and way forward included."
---

The diff makes the change the board entry asks for.
It takes the entry's first remedy: the refusal names the anchored root and a likely earlier `cd`.
It ships agent guidance that a `cd` persists and that absolute paths or `git -C` avoid it.
Creel gets the new text verbatim through `preflight.ResolveMode` and `hubClient`.
The findings are all LOW or NIT, so the verdict is APPROVED.

### [LOW] Drift's capture child carries a shell-cd diagnosis that does not apply to it

**Location:** `internal/lymcwd/anchor.go:115`

**Issue:** `StartCapture` starts `drift capture-child` in the anchored directory (`internal/driftengine/spawn.go:77`), so no agent shell or `cd` is involved.
When that child still hits the gate, `runCaptureChild` passes `err.Error()` on as a lasting fault (`internal/driftcli/cli.go:260-262`).
`startFailure` (`internal/driftengine/spawn.go:168`) and the baseline row (`internal/driftengine/baseline.go:177`) then embed it in a halt reason that says resuming cannot change the error.
That reason now also says an earlier `cd` likely moved the shell and that `cd` followed by a re-run is the way forward.
The path is rare, since lym itself sets the child's cwd, so this is LOW.

### [LOW] The cd way forward is not quoted

**Location:** `internal/lymcwd/anchor.go:117`

**Issue:** `cd %s` with a bare `anchorAbs` gives `cd /home/John Doe/hub/w` for a hub under a directory whose name holds a space.
Bash then reports too many arguments, so the way forward cannot be run as written.

### [LOW] The way forward's target for a subpath anchor is untested

**Location:** `internal/lymcwd/gate_test.go:77`

**Issue:** The only row with `wantMessage` uses anchor `"."`, where `anchorAbs` equals `worktreePath`.
The subpath rows check `errors.Is` alone, so a swap of the way forward's argument to `worktreePath` would pass.

### [LOW] The shipped entry claims more than the gate does

**Location:** `internal/heddleengine/shipped.json:21`

**Issue:** "a `lym` verb run after it is refused" holds for verbs behind `Resolve`.
`ResolveWithin` accepts a cwd at or below the anchor (`internal/lymcwd/lymcwd.go:141-160`).
`ResolveMode` resolves a plain repo's subdirectory as standalone (`internal/preflight/predicates.go:85-93`).

### [LOW] The new pinned row asserts the heading's wording

**Location:** `internal/heddleengine/store_test.go:188`

**Issue:** The row pins `## Keep the shell where it started`, static prose of a shipped instruction.
The row is needed for slug, order and target, but `levelOnly` keeps those without fixing the words.

### [LOW] The blast radius omits the rendered agent instructions

**Location:** `_lym/landing/summary.md:31`

**Issue:** The new shipped entry renders into the agent files of every worktree of every hub on the next lym build.
A wrong entry misguides every session, which `lymcwd` does not name.
The `two-way` door holds: a revert removes the entry and the next render drops it.

### [NIT] The gate test's doc comment omits the message pin

**Location:** `internal/lymcwd/gate_test.go:18`

**Issue:** The doc comment lists the triples and the ungated `ResolveWorktree` check, but not the whole-text check the `wantMessage` field adds.

## Rejected

- Scope: `env -C` in the shipped entry is not unrequested scope, since it is the same advice to avoid `cd` that the entry asks for.
- Root cause: the `Root-Cause:` trailer names the message's missing cause and way forward, which is the defect the diff removes.
- Refusal rules: `lymcwd` is not bound by `PATTERN-refusal-way-forward`, and the new trailing `way forward:` clause names no denied form either way.
- Shell guard: the suggested `cd` to the anchored directory does not conflict with the steer against a `cd` into the directory already worked in.
- Docs: no doc, spec, stencil or PATTERN file quotes the old refusal text or lists the shipped entries, so none needed an update.

## Focus departures

- "The way forward in that refusal and in the new shipped entry: is each suggested command usable as written, for example `cd <path>` with a path holding spaces or Windows separators, and `env -C` on the platforms the repository supports?"
  The `cd` part is settled by F2.
  I did not settle whether `env -C` exists on macOS's BSD `env` or on Windows outside Git Bash, because no evidence for either could be read on this Linux machine, and I filed nothing on that point.
- "Check cwds other than a subdirectory (parent, sibling, another worktree)."
  The hub directory above a worktree is not a git worktree, so it returns `ErrNotAGitRepo` before the gate, and another worktree resolves to its own root.
  Neither reaches the new text, so neither raises a finding.
