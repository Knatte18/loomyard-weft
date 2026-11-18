---
round: 1
focus:
  - "internal/boardengine/render.go RenderToDisk: the diff swaps fsx.AtomicWrite for os.WriteFile. AtomicWrite also ran PathGuard on relPath and MkdirAll on the parent directory. Does any relPath that `files` can hold need a parent directory that now fails to exist, and is the PathGuard check on the render's output paths still enforced somewhere?"
  - "internal/boardengine/render.go RenderToDisk: os.WriteFile truncates then writes, so a reader (lym board reads, git, a preview) can see an empty or partial file. Does any code in the repo read the rendered files while a render may run, and does the new failure path leave a half-written file that the manifest cleanup or a retry mishandles?"
  - "Scope against the decision record's Goal: the operator asked to find the cause first, expecting it in VS Code settings, and to change how lym writes board files only if the cause is there. Does the diff or its commit message give evidence for the rename cause, and does it say how the operator confirms the fix by hand? Check the claims in the commit message and in the new RenderToDisk and doc.go comments against what the diff supports."
  - "Other callers of fsx.AtomicWrite or temp-and-rename writes for board output (for example the manifest file, README, project files) that the in-place rewrite leaves unchanged: does the fix cover every file the VS Code preview opens, or only the ones in the `files` loop?"
  - "internal/boardengine/render_test.go TestRenderToDiskRewritesAnExistingFileInPlace: does os.SameFile fail on the old AtomicWrite code on Linux, so the test pins the behavior? Check the repository's test conventions (tiergate, parallel use, line length of the Task literal) and the test economy bullets."
  - "internal/boardengine/doc.go and the RenderToDisk doc comment: do the new sentences follow the repository's comment conventions and state something that is true of the code, including the claim about a watcher reporting a rename as a creation?"
---

Seed focus for round 1. The diff is small: one write call changed in `RenderToDisk`, a test, and two comment additions.
The main risks are the dropped `PathGuard` and `MkdirAll`, non-atomic writes, and whether the change matches the record's find-the-cause-first scope.
