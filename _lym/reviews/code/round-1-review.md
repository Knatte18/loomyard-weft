---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/boardengine/render.go:49
    summary: Swapping fsx.AtomicWrite for os.WriteFile drops the PathGuard check and the parent-directory creation on every rendered output path, so a nested readme or design_prefix now fails every board write and a ../ or absolute readme writes outside the board directory
    fix: In RenderToDisk's write loop in internal/boardengine/render.go, before os.WriteFile, call fsx.PathGuard(relPath) and return its error wrapped as write <relPath>, then call os.MkdirAll(filepath.Dir(filepath.Join(boardPath, relPath)), 0o755) and return its error the same way; keep the in-place os.WriteFile. Add rows to TestRenderToDiskRewritesAnExistingFileInPlace or a table beside it in internal/boardengine/render_test.go - an Outputs with Readme "views/Home.md" renders and the file exists under views/, and an Outputs with Readme "../outside.md" returns an error and creates no file outside the temp board dir.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/boardengine/boardtest/concurrency_test.go:27
    summary: The TestConcurrentReadsDuringUpserts doc comment still says each write performs an AtomicWrite of Home.md, which this diff makes false
    fix: In internal/boardengine/boardtest/concurrency_test.go, change the doc-comment line on TestConcurrentReadsDuringUpserts that reads "Each write runs Board.boardCriticalSection, which performs an AtomicWrite of board.json, Home.md and _Sidebar.md." so it says the write saves board.json atomically and rewrites the rendered board files in place; change no assertion.
    origin: assist:assist
  - id: F3
    severity: NIT
    class: consistency
    location: internal/boardengine/render.go:38
    summary: The RenderToDisk doc comment states as a general fact that an editor's file watcher reports a rename-over as a creation, while the evidence covers VS Code's watcher on Linux only
    fix: In the RenderToDisk doc comment in internal/boardengine/render.go, replace "An editor's file watcher reports a file replaced by rename as created, not changed." with "VS Code's file watcher reports a file replaced by rename as created, not changed."
    origin: handler
---

### [BLOCKING] Rendered output paths lost their guard and parent-directory creation

**Location:** internal/boardengine/render.go:49

**Issue:** `fsx.AtomicWrite` ran `PathGuard` on `relPath` and `AtomicWriteBytes` ran `os.MkdirAll` on the parent directory (`internal/fsx/fsx.go:53`, `:80`).
The replacement `os.WriteFile(filepath.Join(boardPath, relPath), ...)` does neither.
The output names come from the `readme` and `design_prefix` config keys, which `template.yaml:1` documents as relative to the board dir and which `LoadConfig` checks only against the `decision-` clash (`config.go:198-213`); `readme` is not validated at all, and both can be set by environment variable.
So `readme: views/Home.md` or `design_prefix: designs/`, which worked before, now fails every render with ENOENT, after `board.json` is already saved and before `spawnSync` runs (`board.go:186-197`).
A `readme` of `../outside.md` or an absolute path, which `PathGuard` refused, now truncates and overwrites a file outside the board directory.
The summary's "The change" section does not mention either loss.
The code-affected-set lens, the code-tests lens (missing test) and the assistant reported the same defect; it is recorded once here.
It is raised to BLOCKING over the lenses' MEDIUM because it silently removes an existing boundary check and the fix is two calls.

### [LOW] Concurrency test comment still claims an atomic Home.md write

**Location:** internal/boardengine/boardtest/concurrency_test.go:27

**Issue:** The comment says each write "performs an AtomicWrite of board.json, Home.md and _Sidebar.md".
After this diff, `Home.md` (the configured readme) is rewritten in place by `os.WriteFile`, so the comment is false for it.
The test's assertions read the board through the API and are unaffected.

### [NIT] Doc comment generalises the watcher behaviour beyond the evidence

**Location:** internal/boardengine/render.go:38

**Issue:** The summary's evidence is VS Code's parcel watcher on Linux, where inotify `IN_MOVED_TO` maps to `create`.
"An editor's file watcher" claims this for any editor and platform, which nothing in the change supports.

## Rejected

- code-correctness F2 and code-tests F2, a failed in-place write leaving a truncated file: the summary's design decision accepts this explicitly. The files are a derived view lym does not parse back, the render runs under the board lock after `board.json` is saved, and the next render rebuilds them. Nothing in the diff contradicts that reasoning.
- code-tests F1, no test for nested or escaping output paths: merged into F1, whose fix adds the test rows.
- code-scope F1, decision record files still written by temp-and-rename: the summary names this under Open risks and gives a sound reason, since lym parses records back and needs the atomic write. The observed symptom in the goal is the rendered board files. Moving records to in-place writes would trade a crash-safety property for preview refresh, a change the goal does not ask for.
- assistant F3, the report cites no VS Code documentation or issue-tracker sources: the goal's requirement is that the report state the cause with evidence, and it does so from the shipped VS Code source, which is primary evidence. Naming sources is a means, not a defect in the diff, and the summary is judged here only for claims the diff does not bear out.
- assistant F1 and F2 as separate findings: they share one cause and one fix site, so they are merged into F1.
