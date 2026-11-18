---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/boardengine/render.go:49
    summary: Removing PathGuard lets configured render paths overwrite files outside the board
    fix: Restore fsx.PathGuard on each relPath before opening its output in RenderToDisk, retaining the in-place write. Add a RenderToDisk regression case with a parent-directory path and assert rejection and unchanged outside content.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/boardengine/render.go:49
    summary: Configured outputs in missing subdirectories now fail after the authoritative board save
    fix: After validating relPath, create the output parent with os.MkdirAll and mode 0755 before os.WriteFile, propagating errors with the output name. Extend the existing RenderToDisk tests with a nested configured README or design prefix whose parent does not exist.
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/boardengine/render.go:38
    summary: The writer change omits the upstream research explicitly required to establish its cause
    fix: Complete Goal 1 by consulting VS Code documentation and its issue tracker, and record the sources and their applicability to the installed watcher and preview behavior in the change report. Keep or revise the writer change and its explanation according to that evidence, distinguishing the installed configuration from universal watcher behavior.
    origin: lens:code-scope
---

### [BLOCKING] Restore the output-path guard

**Location:** internal/boardengine/render.go:49

**Issue:** A configured `readme: ../outside.md` now escapes the board directory and truncates the sibling file.
`Render` uses `Outputs.Readme` directly as a map key at `render.go:121`.
`LoadConfig` checks the design prefix against decision-record names, but performs no relative-path guard.
The old `fsx.AtomicWrite` called `PathGuard`, which rejects parent-directory components before opening a file.
The replacement removes that boundary without replacing it elsewhere.
The correctness and affected-set lenses independently identified this regression.
The tests lens confirmed that its existing filesystem-helper tests no longer protect the render call.

### [MEDIUM] Keep support for nested configured outputs

**Location:** internal/boardengine/render.go:49

**Issue:** `readme: views/Home.md` fails when `views` does not yet exist.
The same applies to a nested design prefix.
The configuration template documents a relative README path and imposes no basename-only restriction.
Previously, `AtomicWriteBytes` created the parent directory before writing.
The new call does not.
`boardCriticalSection` creates only the board root, saves the store, and then renders.
The regression therefore returns an error after the mutation is saved and prevents the subsequent sync launch.
The correctness lens independently found this defect; its missing regression coverage is folded into this fix.

### [MEDIUM] Complete the required source investigation

**Location:** internal/boardengine/render.go:38

**Issue:** Goal 1 explicitly requires VS Code documentation and issue-tracker research.
The change report instead declares that all evidence comes from the installed executable and this hub.
It provides no upstream documentation or issue evidence.
The installed preview code supports the described change-event subscription,
and the installed watcher maps Parcel creation events separately from update events.
Those observations justify investigating the writer, but do not complete the requested source comparison.
The manual confirmation procedure is present, so there is no separate omission for Goal 3.

The handler consulted the [watcher internals documentation](https://github.com/microsoft/vscode/wiki/File-Watcher-Internals)
and [issue 346824](https://github.com/microsoft/vscode/issues/346824).
The documentation distinguishes watcher implementations.
The issue describes stale previews after missed events under different conditions and versions;
it does not by itself prove this hub's rename diagnosis.
The change report should make that applicability assessment rather than claim universal watcher behavior.

## Review coverage

Reviewed the supplied diff for `9c5bd13101af1686192cd463afa2b7b583b72697..HEAD` and both commit messages.
All rendered README, project and design files pass through the changed loop.
The manifest remains atomic and is not a markdown preview output.
Decision records remain outside that loop; the report discloses their stale-preview risk.
The output-format and manifest-cleanup behavior is otherwise unchanged.

Normal board reads use the store, and sync commits take the board write lock.
A partial rendered view on write failure remains recoverable from the saved store on a later successful render.
The manifest is not advanced on that failure.
No additional concrete corruption of authoritative data was established from the loss of atomic writes alone.

The inode test distinguishes the old implementation on Linux:
the replacement temporary file is allocated while the original still exists.
Its identity assertion adds coverage beyond the existing content and cleanup tests.
It uses tier 0, parallel execution and a fixed render time, with no waits or subprocesses.
Line width and mechanical comment wrapping remain the gate's subjects.
The changed comments explain the intended write mechanism; their watcher explanation needs the source qualification in F3.

## Validation

Attempted `timeout 3600 lym gate test --wait --tier 0 ./internal/boardengine -- -run 'TestRenderToDisk' -count=1`.
It failed before running tests because the gate could not create its holder record under `_bolt/.lym/gate`:
that directory is read-only in this session.
The reported defects are code traces, not claimed runtime reproductions.
No target files were changed.

## Rejected

- A separate test-coverage finding duplicates F1 and F2; their fixes include the missing regression cases.
- The duplicate correctness and affected-set findings are consolidated into F1 and F2.
- A separate atomicity finding lacks an established broken authoritative-data consumer or unrecoverable retry path.
- A blanket requirement to convert every atomic markdown write is not established by this diff's render-specific diagnosis.
