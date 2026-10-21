---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/fabricengine/taskpatch_integration_test.go:33
    summary: No test runs ReadTaskPatch over a warp whose root .scratch is the fabric-wired link, so nothing pins that the link stays out of Full, FullHash and PatchID
    fix: In taskpatch_integration_test.go add a row to TestFabric_ReadTaskPatch that wires the fixture's warp root .scratch as a link into a directory outside the warp (through the scratch step, or fslink.CreateDirLink plus the /.scratch exclude line), writes a file under .scratch/, and asserts the TaskPatch is byte-identical to the one read before the file was written. Give the same row a second fixture with a real .scratch directory and a /.scratch/ exclude line and assert both fixtures give the same FullHash and PatchID for the same code change.
    origin: lens:code-tests
---

### [LOW] No coverage of ReadTaskPatch over a linked root .scratch

**Location:** `internal/fabricengine/taskpatch_integration_test.go:33`
**Issue:** The parent's `ReadTaskPatch` counts every untracked, non-ignored file, and its skip entries (`patchSkipEntries` in `internal/loomcli/subjects.go`) never name `.scratch`.
The patch therefore stays free of scratch content only because the warp `.git/info/exclude` ignores the root `.scratch` entry, and the task's scratch step now makes that entry a link excluded as `/.scratch`.
By reading, the interaction is correct: `UntrackedFiles` honours `info/exclude`, the pattern has no trailing slash, and a pair wired by an older binary (real directory, `/.scratch/` line) is ignored the same way, so identical code hashes the same on both.
No test pins this, since `newTaskPatchFixture` builds a plain unwired repo and `scratchlink_integration_test.go` never calls `ReadTaskPatch`.
A later change to the exclude pattern or to how untracked files are listed would leak scratch content into the patch hash with no test failing.
The correctness fork could not run the go-git symlink handling and read it from the code only, which is a further reason to pin it.

## Rejected

- Unrecorded `MkdirAll` of the target's parent in `adoptScratchDirectory` (lens:code-correctness): the code lives in the task's own commits alone and no parent change touches it, so it is not this round's subject.
- A correct link whose target path holds a regular file reports nothing in `restoreScratchTarget` (lens:code-correctness): the code lives in the task's own commits alone, outside the interaction.
- Possible row overlap between the `Reconcile*` and wiring rows of `scratchlink_integration_test.go` (lens:code-tests): both sides of the overlap are task tests, and the fork found no clear duplicate.
- No test seen for `PairComplete` and `fabric status` ignoring `.scratch` (lens:code-tests): a task-alone behavior the fork did not confirm missing, and the parent did not touch it.
- `doc.go` Kind list lacks `merge_tracked_staged`, `tag_pushed` and `commits_dropped` (lens:code-affected-set): the gap predates both sides and the merge did not cause it; `path_renamed` is present.
- Package-variable hooks `scratchBeforeLinkHook` and `scratchPathTracked` (lens:code-pattern-fit): they are not durations, so `PATTERN-test-speed` does not bar them, and they are the task's own code.
