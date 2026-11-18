---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/boardengine/render.go:50
    summary: Removing PathGuard allows configured output paths to overwrite files outside the board directory
    fix: In RenderToDisk, call fsx.PathGuard on each rendered relative path before joining or writing it, returning the existing write-context error on refusal; retain the in-place write after validation.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/boardengine/render.go:50
    summary: Configured nested output paths fail because the replacement writer no longer creates their parent directories
    fix: After validating each relative path in RenderToDisk, create filepath.Dir of its joined output path with os.MkdirAll and mode 0755 before os.WriteFile; propagate directory-creation errors with the output path.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/landing/summary.md:9
    summary: The cause report omits the official VS Code documentation and issue-tracker research required by the goal
    fix: Extend the cause report with relevant official VS Code watcher documentation and issue-tracker evidence, citing the sources and distinguishing what they establish from the installed-code diagnosis and remaining manual confirmation.
    origin: lens:code-scope
  - id: F4
    severity: LOW
    class: consistency
    location: internal/boardengine/render.go:50
    summary: The writer change leaves the concurrency test documentation falsely promising atomic writes of Home.md
    fix: Update the TestConcurrentReadsDuringUpserts comment in internal/boardengine/boardtest/concurrency_test.go to describe atomic persistence of board.json and in-place rendering of derived pages, removing the claim that Home.md is written atomically.
    origin: lens:code-affected-set
---

### [BLOCKING] Restore the output path boundary

**Location:** internal/boardengine/render.go:50

**Issue:** `Render` uses `out.Readme` as an output-map key without validating it.
`LoadConfig` accepts that external configuration and checks only decision-prefix collisions among output settings.
A value such as `../outside.md` now reaches `os.WriteFile` and can truncate a sibling file.
The former `fsx.AtomicWrite` refused this path through `PathGuard` before touching it.
This is a lost boundary check on configuration, not defensive handling of an impossible internal input.
The affected-set lens independently found the same defect; it is recorded once here.

### [MEDIUM] Preserve nested output creation

**Location:** internal/boardengine/render.go:50

**Issue:** The template declares `readme` relative to the board directory, without restricting it to a basename.
For `readme: views/Home.md`, the old writer created `views` through `AtomicWriteBytes`.
The new writer fails when that directory is absent.
`boardCriticalSection` creates only the board root and saves the store before rendering.
Consequently, this regression can leave the mutation saved while its render fails and its sync is never launched.
The same issue applies to nested design prefixes.
The affected-set lens independently found this defect too.

### [MEDIUM] Complete the requested source research

**Location:** _lym/landing/summary.md:9

**Issue:** Goal 1 expressly requires VS Code's documentation and issue tracker.
The report instead attributes all evidence to the installed application and hub.
Its cause analysis and manual procedure are substantive, but neither supplies the requested upstream research.
The official [watcher documentation](https://github.com/microsoft/vscode/wiki/File-Watcher-Internals)
distinguishes recursive and non-recursive watchers.
[Issue 346824](https://github.com/microsoft/vscode/issues/346824) describes stale previews after missed events,
including symlink and outside-workspace cases; those are different setups from this hub's reported rename case.
These sources need to be evaluated, not presented as proof of a matching reproduction.

### [LOW] Correct the affected concurrency comment

**Location:** internal/boardengine/render.go:50

**Issue:** `internal/boardengine/boardtest/concurrency_test.go:27` says the critical section atomically writes `Home.md`.
This change makes that statement false.
The test reads board API data and still checks the authoritative store's consistency;
the stale claim concerns the derived output's write guarantee.
This finding does not ask to change its assertions or repair unrelated historical wording.

## Review coverage

The reviewed range is `9c5bd13101af1686192cd463afa2b7b583b72697..HEAD`, including both commit messages.
README, root and project pages, and design pages all pass through the changed loop.
The manifest is not a preview page, and decision records remain a separate write path disclosed by the report.
No additional defect was established merely from those unchanged atomic writes.

The new regression checks inode identity and updated content at the exported rendering surface.
On Linux, the old temporary file exists alongside the old destination before rename, so they have distinct inodes.
The identity assertion therefore distinguishes the old implementation without duplicating the existing cleanup tests.
The new test has no waits and uses tier 0, parallel execution, and an isolated directory.

Board API reads use the authoritative JSON store, and `commitDirty` acquires the board write lock.
Replay also reads design files for comparison, so the report's statement about never parsing pages is not a claim
that no code ever reads their bytes.
A failed in-place write returns before manifest cleanup; a subsequent successful render rewrites the derived file.
No concrete retry or concurrent staging defect was established beyond the path regressions above.

The report provides a manual preview check after the document lifetime expires.
The upstream [preview source](https://github.com/microsoft/vscode/blob/main/extensions/markdown-language-features/src/preview/preview.ts)
subscribes to filesystem changes when the document is not loaded, supporting that part of the diagnosis.
The new comments are read as rationale for this reported VS Code path, not a universal watcher specification.

## Focus departures

- “Does os.SameFile fail on the old AtomicWrite code on Linux”: checked by tracing the two live files before rename.
  No execution against the old implementation was performed because the review permits no target edits.
  The attempted `lym gate test --tier 0 ./internal/boardengine` reported a read-only runner-registration failure
  and zero discovered tests; it is not a passing test run.
- Mechanical tier, parallelism and line-length checks were inspected only as context.
  The rubric assigns mechanical failures to the gate, so no duplicate lint finding is raised.
