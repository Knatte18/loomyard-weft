# Add beams: project entries, pairs from a parent pair, lyx beam verbs

A beam is a long-lived pair that groups board work, spawns its tasks from its own branch, takes them back, and may have an orch of its own.
This change lays the groundwork and adds the `lyx beam` module.
The word is beam throughout; nothing is named umbrella.

Closes #724

## What changes

- **Board**: a third entry kind, `project` (handle letter `P`), with a `beam` flag and an ordered `order` list.
  Tasks and notes gain a `project` field.
  A project takes no `depends_on`, `recipe`, `profile`, `run_settings` or `priority`, and no task may depend on one.
  Open projects list under their own layer, `Project`, and the README gains a `## Projects (n)` section showing each project's members in plan order.
  `LoadConfig` refuses a type label whose letter is `P`.
- **Slug rule (#724)**: fabric exports its pair slug rule and the board applies it, through a seam `boardcli` fills, when a task or project slug is made (new upsert, note promote, rename).
  `lyx batten cleanup`'s skip of a run that is not done now names `lyx batten cancel <slug>`, which covers the blocked-run half of the issue.
- **Fabric**: `AddOptions.Parent` forks a pair from a named parent pair's branch, exposed as `lyx fabric add --parent`.
  `lyx fabric add --adopt-local` adopts an unmarked local branch; batten and adopt keep the `ErrBranchExists` refusal.
  The origin record gains a `beam` mark.
  `RegenerateHubWorkspace` writes the hub's workspace file with no pair under a beam, plus one `<repo>-<beam>.code-workspace` per beam.
- **Batten, adopt, cleanup**: they run from the prime and from any beam, with all state kept under the prime.
  A task pair that is not a beam is refused, naming the nearest beam or the prime.
  `--window` opens in the reed session of the run's parent worktree.
  Worktree-Create forks a member from its beam.
- **Orch**: `lyx orch` runs in the prime and in any beam, one per worktree, as strand `<shortname>:<slug>:orch`.
  Tender, hibernate, overview and cleanup read every orch of the hub.
  The orch stencil tells an orch to act only on entries whose `parent_beam` is its own worktree.
- **`lyx beam`**: `create` (resumable, step by step), `promote` (turns a spawned task into a beam, ending its run) and `land`.
  `land` runs `lyx batten run` on the project entry with the new `beam` recipe: a Go description producer, then Finalize.
- **Notices**: parent names and notices go to the nearest live orch up the beam chain, the prime last.
  The ready notice goes to the prime's orch for a prime task, and to a beam's own orch only when that orch is live here.

## Design decisions

- One parent resolver: `hubgeom.ParentOf` reads the slug's origin record and falls back to `boardengine.ParentBeam` over the board only when no record exists.
  Every owner reader takes it, and `fabricengine.ParentSlug` is removed.
  `PATTERN-pair-parent` records the rule.
- `beam` is written only by `lyx beam create` and `lyx beam promote`, through the owner setter; any other payload value is refused.
- Adding a member with `Parent` named refuses no tracked dirt, since the pair forks from the parent's last commit.
  Landing a member into a dirty beam waits at Finalize's existing Stuck.
- `promote` resets the pair's weft to its parent's content in a destruction executor in `internal/fabricengine/destroy.go`, behind the shared check pipeline.
  The run's discussion, plan and records stay reachable only in weft history.
- A beam's landing never reroutes: the `beam` recipe's Finalize declares no `on_reroute`.
  A conflict resolution or a clean merge of the parent's newer work is bounded only by the `finalize` gate, since no member's review sees it.
- A landing is refused while any open member, local child pair, or live orch strand or watcher of the beam exists.

## Check by hand

- Every machine of the hub must deploy this build before the first project entry is written; an older binary fails its board replay on `kind: project`.
- A hub whose `board.yaml` has a type label beginning with `P` refuses to load its board until the label is renamed.
- A run's driver keeps the notice rule it was launched with, so a beam orch started or stopped later moves batten's notices but not the driver's own parent message until its next launch.
- Two orchs on one hub share the board and gate slots; only the stencil filter keeps one from starting another's task.
- Finding, `internal/battencli` start guard: the refusal for a beam whose local pair has a missing records worktree or records branch (naming `lyx fabric reconcile`) lacks a test for the missing-branch case.
- Finding, `lyx beam promote`: a task whose pair is a root pair (an adopted default branch, empty parent) has no clear way through; confirm what promote does and says there.
- A pair made on another machine with `--parent <beam>` and no board entry is not seen by the landing's child-pair refusal.
- Not run: a real multi-machine adopt of a beam, and a `lyx beam land` of a beam with a live orch against a real reed session.
