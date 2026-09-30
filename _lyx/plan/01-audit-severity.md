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
- `CheckParent` emits one `named-spawn` violation per named spawn, ordinals 1..`NamedSpawns`, in place of today's single aggregate violation.
  A new spawn then gets a new identity while every earlier one keeps its own, which a single violation keyed on the total could not give.
- New `AuditSeverity` string type with `AuditSeverityCorrectness` ("correctness") and `AuditSeverityPolicy` ("policy").
- New exported `ClassifyViolation(v AuditViolation, geom Geometry) (AuditSeverity, error)` implementing D4, correctness rule checked first:
  `fork-contract-write` is correctness;
  a `parent-write` whose path (resolved with `resolveWritePath` against `geom.WorktreeRoot`) lies under `geom.WebsterDir`, `geom.ReportsDir`, `geom.PlanDir`, or the run's `_lyx` directory (`filepath.Dir(geom.WebsterDir)`: the parent of a told field, so no path segment is declared here and no accessor is re-run on `AnchorRoot`, per the Told-Geometry constraint; in standalone mode it is the state directory's `_lyx`, which holds the run's state, reports and plan) is correctness;
  a `parent-write` inside `geom.WorktreeRoot` that git does not ignore is correctness;
  every other `parent-write` (outside the worktree, or a git-ignored path outside `_lyx`), and every `fabric-reference`, `named-spawn` and `nested-agent`, is policy.
  Every prefix test compares canonical paths: the write path and each told directory are resolved through links (`filepath.EvalSymlinks` on the nearest existing ancestor, the missing tail re-joined) before comparison.
  In hub mode the warp's `_lyx` is a link (a junction on Windows) to the weft's `_lyx`, so a Master write spelled through the weft path is the same `_lyx` content and must class as correctness, not as a write outside the worktree.
  Resolving a told path's links derives no new path segment, so the Told-Geometry constraint holds.
  The error return is only the git probe's or the link resolution's failure.

In `gitwrap.go`, add `ignoredPath(worktree, path string) (bool, error)`, which reports whether git ignores `path` in `worktree` through `gitexec.Run` (for example `git status --porcelain --ignored -- <path>` and a `!!` line), following `dirty`'s carve-out.
A path that does not exist and is not ignored reads as not ignored, the conservative answer.

`auditseverity_test.go` (`//go:build integration`, real scratch repo from the existing helpers) table-drives `ClassifyViolation` over every class and every `parent-write` path family: a tracked file, an untracked non-ignored file, a git-ignored file outside `_lyx`, `_lyx/webster/state.json`, a report under the reports dir, a plan file, and a path outside the worktree.
A further row makes the fixture's `_lyx` a directory link to a sibling directory through `fslink.CreateDirLink` and spells a `parent-write` to its `webster/state.json` through the link's target, expecting correctness.
It also asserts `Key` values for two parent writes and two named spawns are distinct and stable when the audit grows by one more entry.

**ImpactSummary:** Additive fields and helpers on the audit types; existing CheckFork/CheckParent callers keep compiling and gain populated Key/Path, and a run with N named spawns now yields N named-spawn violations instead of one, which only changes a joined error's length until card 8 dispositions them.
