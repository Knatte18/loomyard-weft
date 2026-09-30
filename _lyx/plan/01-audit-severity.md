# Card 1 — audit-severity

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/gitwrap.go`

**Create:**
- `internal/websterengine/auditseverity_test.go`

**Intent:**
Give every fork-audit finding a stable identity and a D4 severity class, so later cards can disposition each finding once instead of refusing on it forever.

In `audit.go`:

- `AuditViolation` gains `Key string` and `Path string`.
  `Path` is the transcript-recorded write path for `parent-write` and `fork-contract-write`, empty for every other class.
  `Key` is a deterministic per-finding key that `CheckFork` and `CheckParent` fill:
  a fork finding is `fork:<transcript path>:<class>:<ordinal>`, where the ordinal counts that class's violations within that transcript;
  a parent finding is `parent:<class>:<ordinal>`, where the ordinal is 1..`NamedSpawns` for `named-spawn`, the index into `ForkAudit.ParentWrites` for `parent-write`, and the index into `ParentBashCommands` for `fabric-reference`.
  Indexes into the transcript-ordered slices keep a key stable while Master's session grows.
- New `AuditSeverity` string type with `AuditSeverityCorrectness` ("correctness") and `AuditSeverityPolicy` ("policy").
- New exported `ClassifyViolation(v AuditViolation, geom Geometry) (AuditSeverity, error)` implementing D4, correctness rule checked first:
  `fork-contract-write` is correctness;
  a `parent-write` whose path (resolved with `resolveWritePath` against `geom.WorktreeRoot`) lies under `geom.WebsterDir`, `geom.ReportsDir`, `geom.PlanDir`, or the anchor's `_lyx` directory (`filepath.Dir(Dir(geom.AnchorRoot))`, so no path segment is declared here) is correctness;
  a `parent-write` inside `geom.WorktreeRoot` that git does not ignore is correctness;
  every other `parent-write` (outside the worktree, or a git-ignored path outside `_lyx`), and every `fabric-reference`, `named-spawn` and `nested-agent`, is policy.
  The error return is only the git probe's failure.

In `gitwrap.go`, add `ignoredPath(worktree, path string) (bool, error)`, which reports whether git ignores `path` in `worktree` through `gitexec.Run` (for example `git status --porcelain --ignored -- <path>` and a `!!` line), following `dirty`'s carve-out.
A path that does not exist and is not ignored reads as not ignored, the conservative answer.

`auditseverity_test.go` (`//go:build integration`, real scratch repo from the existing helpers) table-drives `ClassifyViolation` over every class and every `parent-write` path family: a tracked file, an untracked non-ignored file, a git-ignored file outside `_lyx`, `_lyx/webster/state.json`, a report under the reports dir, a plan file, and a path outside the worktree.
It also asserts `Key` values for two parent writes and two named spawns are distinct and stable when the audit grows by one more entry.

**ImpactSummary:** Additive fields and helpers on the audit types; existing CheckFork/CheckParent callers keep compiling and only gain populated Key/Path.
