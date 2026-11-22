```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced what the diff forces outside the lines it touches: callers, tests, docs, skills, stencils and rule documents.

**Per change, the set found and whether it is covered**

- **Board fields `pull_request` and `notify_on_land` removed.** Every Go caller is updated: `Task`, `TaskView`, `View`, `WaitingTask`, `boardcli` upsert help and verbs, replay, run lock, sketch, raddle, render, prune and their tests. `contracts/orchrole/orch-role.md`, `docs/overview.md` (Land) and `battenshed` `LandDeps` are updated. A search for the old Go symbols finds only the fold and the refusal hint, which is intended.
  - Not covered: `plugins/ly/skills/board/SKILL.md:260`, see F1.
- **The resolver's inputs moved from parent mode plus entry field to `Mode`.** `landingshed` call sites, the `envkit` fixture, both wire-seam tests, the beamcli `ready` and `pr close` paths, `PATTERN-landing.md` and `landingshed` `doc.go` are updated.
  - Not covered: `docs/overview.md:1378`, see F2.
- **New lookup read through `configengine.TopLevelValue`.** The `PATTERN.md` line and the new "Per-task beam settings" section landed.
  - Not covered: the same file's Strictness section (F3), and `reinterpret.go` reading outside the lookup (F4).
- **pull_request and land_notice values are named in `dobbyengine`.** The other packages that use these values keep their own literals or constants, see F5.
- **Meaning change of `pull_request` and `land_notice` in `beam.yaml`.** The reinterpretation is registered in `configreg` and the `LocalDenied` test is updated. The reinterpretation covers the hub file and flags the prime only, see F6.
- **View JSON keys renamed.** The top-level `land_notice` is gone and a run's `notify_on_land` became `land_notice`. The only consumer is `battencli`, which is updated. Covered.
- **`role-beams.md`, `beamcli` `--help` and `doc.go`, `boardcli` `--help`, `help_test`.** Updated. Covered.
- **Commit messages and departures.** There are no `Pattern-Departure`, `Loosened-Check` or `Root-Cause` trailers, and none is needed: this is not a bug fix, and no departure is claimed. No commit claims agreement or improvement. Nothing to file.

**Findings**

- **F1 — LOW, consistency.** `plugins/ly/skills/board/SKILL.md:260` still says "Prune keeps an entry that still carries `notify_on_land` until its dobby has sent the line and cleared the field."
  - The diff removes that prune rule and the field (`internal/boardengine/store.go`, Prune hunk) but leaves the agent-facing skill text wrong.
  - Fix: drop the line, or say that a landing notice comes from `beam.land_notice` in run_settings.
- **F2 — LOW, consistency.** `docs/overview.md:1378` still reads "The resolver decides from the parent's `beam.yaml`, the entry and GitHub's rules for the parent branch."
  - The resolver now takes the landing run's per-task lookup value and no entry field (`internal/landingshed/pullrequestmode.go`, `ResolvePullRequest`).
  - Fix: name the per-task lookup from the run's worktree up, and drop "the entry".
- **F3 — LOW, consistency.** `pattern/PATTERN-config-layers.md`, Strictness section, still says a caller adopts exactly one of `Load` or `LoadOrTemplate`, "except `runprofile`".
  - The Layers rule above it, as this diff amends it, now exempts "the per-task lookup", which reads raw layers through `TopLevelValue`. The two sections of the same background file now disagree.
  - Fix: add the per-task lookup as a further stance, as `runprofile` is described there.
- **F4 — BLOCKING, design.** `internal/dobbyengine/reinterpret.go:65-80` (`flagPrimeRetiredValues`) reads `pull_request` and `land_notice` from the prime's files through `configengine.TopLevelValue` directly.
  - That is neither `Load`, `LoadOrTemplate` nor the per-task lookup. The amended rules in `pattern/PATTERN-config-layers.md` forbid it: the Layers rule ("callers use `Load` or `LoadOrTemplate`, bar `runprofile` and the per-task lookup") and the new "Per-task beam settings" rule ("Reading a per-task beam setting … only `dobbyengine`'s one lookup").
  - Fix: either have the per-task lookup code expose the raw own-file read the reinterpretation uses, or name the reinterpretation as an allowed reader in both rules.
- **F5 — BLOCKING, design.** `PATTERN-code-conventions` "Named constants" says a name that can go stale is "one named constant, declared once and referenced everywhere, never a repeated literal". The diff breaks this in three places:
  - `internal/landingshed/pullrequestmode.go:20-24` now declares its own `pullRequestMergeIn = "merge-in"`, along with `on` and `merge`. Before the diff it referenced the one shared constant, `boardengine.PullRequestMergeIn`. `dobbyengine` now declares the same values as `PullRequestOn`, `PullRequestMerge` and `PullRequestMergeIn` (`internal/dobbyengine/config.go:30-41`).
  - `internal/beamcli/pr.go:21-22` declares `pullRequestModeMerge = "merge"`, though `beamcli` already imports `dobbyengine` (`mergeparent.go`).
  - `internal/boardengine/store.go:251-299` repeats the literals `"beam"`, `"pull_request"`, `"land_notice"` and `"on"` in the fold and in the upsert refusal hints.
  - Fix: declare the per-task keys and values once, in a package that `landingshed`, `boardengine`, `beamcli` and `dobbyengine` can all import, and reference it at every use site.
- **F6 — MEDIUM, design.** `PATTERN-config-layers` "Config reinterpretation" requires a meaning change to rewrite each value to its old meaning, or flag it with a way forward. `internal/dobbyengine/reinterpret.go:35-49` rewrites only the hub base file and flags only the prime's own files.
  - A beam worktree file holding `required`, `all` or `per-task` is neither rewritten nor flagged. Now that `LoadConfig` refuses those values (`internal/dobbyengine/config.go:115-123`), that beam's `beam.yaml` stops loading and its dobby with it.
  - A worktree-level `pull_request` also changed meaning without a rewrite or flag. It used to decide landings into that worktree's branch; it now decides that worktree's own landing too, and is inherited downward.
  - The summary names the beam gap as an open risk, but it is a gap against the rule.
  - Fix: flag every beam worktree's retired values too. Rewriting them is better.

**Tool calls**

1. grep of `docs`, `contracts`, `plugins` and `cmd` for pull_request, land_notice and the old seam names. The manifest's earlier search excluded neither pattern nor covered `docs/*.md` for these terms.
2. grep of `docs/overview.md`, `contracts` and `plugins` for the resolver's `beam.yaml` and entry wording, plus a search for the old Go symbols. The manifest did not cover the overview's PR-Review paragraph.
3. Read `docs/overview.md:1370-1385` and checked `internal/creelcli`. I needed the stale sentence's exact text and to rule out a creelcli hit; it turned out to be a false match on "EntryFields".
4. grep of `beamcli`, `battencli` and `cmd` for other consumers of the view JSON keys `land_notice` and `notify_on_land`. Consumers of the changed view keys were not in the manifest.
