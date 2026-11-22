# Card 18 — orch-pointer-autonomy-line

**Create:**
- `plan:internal/orchengine#WithLaunchFacts` -> `func WithLaunchFacts(p Paths, pointer string) string`
- `plan:internal/orchengine#autonomyOnLine` -> `const autonomyOnLine = " Autonomy is on: settle every halt yourself and reach the operator only for a severe halt or a merge-ready beam, as role-autonomy.md beside your role says."`
- `plan:internal/orchengine#autonomyOffLine` -> `const autonomyOffLine = " Autonomy is off."`
- `plan:internal/orchengine#TestWithLaunchFacts` -> `func TestWithLaunchFacts(t *testing.T)`

**Edit:**
- `internal/orchengine#Paths`
- `internal/orchengine#ResumeContext`
- `internal/orchengine#Watcher.startClearing`
- `internal/orchengine#Watcher.startAutoReload`
- `internal/orchengine#Watcher.reloadAfterCompaction`
- `internal/orchcli#orchPaths`
- `internal/orchcli#orchCLI.launchOrch`
- `internal/orchcli#orchCLI.resumeOrch`
- `internal/orchengine#TestResumeContext_RendersPointerPerPhaseAndWritesMark`
- `internal/orchengine#TestWatcher_HandoffCompleteClearsThenResumes`
- `internal/orchengine#TestWatcher_AutoCompactionReloadsPluginsThenPointer`
- `internal/orchengine#TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped`
- `internal/orchcli#TestStart_ResumesTheRecordedSession`

**Uses:**
- `internal/orchengine#WithPresence`
- `plan:internal/hubgeom#Autonomy`
- `internal/orchengine#TestRenderPointers_OneLineNamingTheirPaths`
- `internal/orchengine/doc.go`
- `internal/orchcli/doc.go`

**Intent:** D9: the start, resume and reload pointers and the pending resume the watcher types after a handoff each carry one line naming the orch's autonomy, read then, whatever the presence.
The line informs only; the orch acts on a notice's value and on `lym config beam --resolved`.

- `Paths` gains `Autonomy func() bool`, read each time a pointer is built; nil reads off.
  `orchPaths` fills it with `hubgeom.Autonomy(location, location.WorktreeName)`, the orch's own worktree.
- `WithLaunchFacts` returns `WithPresence(p, pointer)` with `autonomyOnLine` or `autonomyOffLine` appended to the same line,
  so the autonomy line stands beside the presence step, never inside it, and appears under a terminal presence too.
- Every pointer site that called `WithPresence` calls `WithLaunchFacts`:
  `launchOrch` (start), `resumeOrch` (reload), `ResumeContext` (the hook's resume and reload),
  and the watcher's pending resumes in `startClearing`, `startAutoReload` and `reloadAfterCompaction`.

Tests, tier0:

- `TestWithLaunchFacts`: autonomy on and off, each under a slack presence, a terminal presence and no presence path;
  the result is one line, holds the presence line only under slack, and ends with the autonomy line.
- `TestResumeContext_RendersPointerPerPhaseAndWritesMark` gains rows with `Paths.Autonomy` on and off under slack and terminal, asserting the rendered pointer ends with the matching line.
- `TestWatcher_HandoffCompleteClearsThenResumes` (`startClearing`), `TestWatcher_AutoCompactionReloadsPluginsThenPointer` (`startAutoReload`)
  and `TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped` (`reloadAfterCompaction`) each set `Paths.Autonomy` on
  and assert the typed or pending pointer ends with the literal `autonomyOnLine`, under each presence row they have.
- `TestStart_ResumesTheRecordedSession` asserts, for its fresh, reload and last-handoff sources alike, that the launched prompt holds the literal `autonomyOffLine` with no autonomy seam,
  and `autonomyOnLine` with a seam reading on.

**ImpactSummary:** Six pointer sites swap one wrapper for another and one Paths field is added; pointer text grows by one sentence on its one line, and every test that pins it is on this card.
