---
round: 1
focus:
  - "internal/proc/tasklimit.go: does the ancestry walk pick the level with the fewest free tasks, skip `max`, missing files and malformed numbers per level, and stop at the `/sys/fs/cgroup` root without escaping it or looping on a odd path (D1, D2)?"
  - "internal/dobbyengine/starts.go: is the hold checked right before a task's first pass is spawned, with no refused-start record, notice or backoff, and do a nil seam, `task_headroom: 0` and a not-found reading start as before (D4)?"
  - "internal/dobbyengine/starts.go and loop.go: is the hold episode state kept in process, does the hold log line carry current, max, cgroup and margin, and does the resume line fire once on the first start the reading lets through (D5)?"
  - "internal/dobbyengine/config.go and template.yaml: does `task_headroom` default to 500, accept 0, and refuse a negative value naming the key (D3)?"
  - "Does the diff leave passes of runs already started, the dobby view, `lym batten overview` and hand `lym batten run` untouched, as the Scope Out list says?"
  - "Docs in the same diff: `internal/proc/doc.go`, `internal/dobbyengine/doc.go` (including the bounded 'starts on the dobby's next tick' sentence) and the template comment (D6); also check new comments against the repo's conventions."
  - "Tests: do the new tests use a temp-dir cgroup fixture and the fake `Deps` without reading the host's real cgroup, cover each acceptance criterion, and avoid duplicating existing coverage?"
---

Seed focus from the board entry and decision record; the range is the single commit over `161a4a7f`.
