# Decision record — verify-diff-scope

## Goal

A darn run stops paying for the whole-module suite at gates where its diff does not need it: the writer and fix-round gates run the diff's impacted set plus the repo-wide scans, and a diff that touches no production code runs only its trees' tests plus the scans at every site, landing included.

## Scope

In:

- Darn's verify sites: the Darn-Write gate, the Darn-Burler-Fix gate, Merge-In's verify, Finalize's catch-up verify, and Publish's `publish_verify` in a darn run.
- Tree scope: hub-wide `darn.yaml` config naming the non-production trees and the tree-scoped command, the classification of the task diff against it, and the command it yields.
- The impact-scoped form of darn's two verify gates, built on `internal/impactset`, whose derived command moves its test steps onto `lyx gate test`.
- A `--guards` flag on `lyx gate test`.
- The stencil, doc and PATTERN background text that describe these sites.

Out:

- Loom's recipe.
  The board's Seen sentence "the same command runs for every darn and at loom's Merge-In and Publish" is read as darn runs passing through the landing rows, since loom's Merge-In runs the plan's own `## verify:`, which the plan writer scopes per task, and loom's card, batch and fix-round gates are already scoped.
- Test selection for non-Go hubs (`dotnet-scoped-verify`, `gate-language-neutral`, `dotnet-verify-protocol`): a non-Go hub gets tree scope from config, and its writer gates keep the whole command because `impactset` falls back without `go.mod`.
  When `gate-language-neutral` declares the hub's tiers in `language.yaml`, the two `darn.yaml` keys of D1 are expected to fold into that tier mechanism, and `--guards` stays on the Go verb it wraps.
- Setting this hub's `darn.yaml` values on Bolt, an operator step from the prime after landing.
- Measuring the gain on OSL-1046, which the board's Order section leaves to the operator.

## Decisions

### D1 — Tree scope is darn.yaml config, off by default

`darn.yaml` gains two keys:

- `tree_scope`: a map from a worktree-relative tree path to the test target string the tree contributes, such as `tools: ./tools/...` and `docs: ""`.
  An empty target means the tree has no tests of its own.
- `tree_scope_verify`: the tree-scoped shell command, holding the placeholder `{targets}`, such as `lyx gate test --tags integration --guards {targets}`.
  It must run with an empty `{targets}`, since a docs-only or empty diff expands it so; the key's template comment says so.

Both keys are closed to the local layer (`CategoryGuard`), as `verify` is, so neither a task's tree nor its `run_settings` can set them.
Tree scope is on only when both are non-empty.
A trailing `/` on a tree key is trimmed.
`LoadConfig` refuses one key set without the other, a `tree_scope_verify` without `{targets}`, and a tree key that is empty, `.`, absolute, or holds `..` or a backslash, each naming `lyx config darn --set` from the prime.
The template ships both empty.

Rationale: the board forbids lyx hard-coding Go or a tree layout, and asks that the trees and the scoped command be config of the module that owns the recipe.
Closing both keys to the local layer keeps a task from widening its own non-production trees or choosing its own command.

### D2 — Classification of the task diff

The task diff is every warp path changed from the pair's merge base with its recorded parent to HEAD, by new and old path.
`fabricengine.ChangedPaths` as it stands lists only a rename's new path, so the read used here must report both sides, such as through `--no-renames`; a rename or deletion out of production code then counts as production.
A path lies in a tree when it equals the tree path or starts with the tree path and a `/`.
The diff is tree-scoped when every path lies in some tree, and an empty diff is tree-scoped with no targets.
The tree-scoped command is `tree_scope_verify` with `{targets}` replaced by the non-empty targets of the touched trees, deduplicated and in tree-path order, joined by spaces, so the same diff always yields the same string.
The classification and expansion are a pure function in `internal/darnengine`, told the config and the changed paths; the `loomcli` wiring reads the merge base and the paths through `internal/fabricengine`.

Rationale: tree, never import graph, per the 2026-10-18 decision.
A deterministic string lets a later site skip on the verified-tree record of an earlier one.
`{targets}` is filled only from config values, never from a diff path, so a path cannot inject shell text.

Bound: a tree-scoped diff gets no whole-module run before it lands.
A break outside the listed trees that neither the trees' tests nor the guard-marked scans exercise, such as a scan of `tools/` that lacks `//lyx:guard`, can reach `main`.
That ends, for tree-scoped diffs, `PATTERN-test-speed`'s "caught by the full plan verify at Publish or Finalize, never on `main`"; the hub bounds it by what it lists as non-production.

### D3 — Darn's writer and fix-round gates take an impact-scoped form

Darn's Darn-Write and Darn-Burler-Fix verify gates leave the whole-diff form.
At each arrival the gate, in order:

1. runs the headroom lint and the comment lint from the merge base, as today;
2. replays a Publish failure record's failing tests by name, as today;
3. runs the tree-scoped command when D2 classifies the diff as tree-scoped, with no trailing `tmux` package pass;
4. otherwise runs the command `impactset.Derive` gives from the merge base: the packages the diff touches, their reverse dependencies and every `//lyx:guard` test, with today's `tmux` pass over the impacted packages for a `publish_verify` record;
5. runs the hub's whole `verify` where `impactset` names a fallback, with today's `./...` `tmux` pass for such a record.

The told `verify` stays the site's base command, so a pass keeps its record entry.
`impactset`'s derived command runs its test and guard steps through `lyx gate test --tags integration`, so they inherit the held slot and get the prebuilt `lyx` and the trace-id strip that keep Go's test cache warm; its build and vet steps stay.
Loom's round gate gets the same derived command; its round and whole-diff forms are otherwise unchanged.

Rationale: this is the scoped gate the 2026-10-20 decision asks for, and `impactset` already computes it for loom's fix rounds.
A raw `go test` builds `lyx` once per `lyxbin` package and misses the cache that a cached whole `lyx gate test` run hits, so without the move the scoped gate could cost more than the run it replaces.
The base is always the merge base, so every attempt covers the whole task diff and a PR-Gate rework is never narrowed to a stale pass.

Bound: an impact-scoped pass lets through a break only the whole module shows, such as a test reading another package's files without `//lyx:guard`; Merge-In's whole run catches it for every diff that touches production code.

### D4 — Landing sites in a darn run

The darn verify source told to Merge-In and Finalize resolves at each call: the tree-scoped command for a tree-scoped diff, else the hub's whole `verify`.
Merge-In resolves it after its merge, so the diff is the task's own side against the parent tip just merged.
Finalize resolves the same way, and on the tree Merge-In verified it skips through the verified-tree record.
In a tree-scoped darn run Publish skips `publish_verify` through a seam the darn wiring tells landingshed, and takes the path of an unset `publish_verify`, which removes the Publish failure record; a nil seam, loom's, runs it as today.

Rationale: the whole-module suite runs once, at Merge-In right before Publish, for every diff that touches production code, so the import graph never decides the last gate.
For a tree-scoped diff the tree-scoped command is the hub's whole statement of what such a diff needs.

Bound: a tree-scoped darn run lands with no `tmux`-tier run, and its Merge-In does not whole-verify parent code merged into the task; the parent's code was verified by the parent's own landing, which for an unlanded task or umbrella parent is still ahead of it.
A hub that wants more for such a diff puts it in `tree_scope_verify`.

### D5 — `lyx gate test --guards`

`lyx gate test` gains `--guards`: after the named packages it runs every top-level test marked `//lyx:guard`, by name in its own package, inside the same slot.
With `--guards` the package list may be empty, and the verb refuses `--guards` unless its tags include `integration`, the tier every guard compiles under.
The marker scan is `impactset.FindGuardTests`, the scan the round gate and the card gate already use.

Rationale: the board's open item names the scan packages as config.
This settles it differently: a hand-kept package list goes stale whenever a scanning test is added elsewhere, while the `//lyx:guard` marker is already the repository's contract for a test that reads files outside its package.
The hub's config still decides whether the scans run, through `tree_scope_verify`.

Bound: an unmarked scanning test is not run, as for the round gate today.

### D6 — Prompt and PATTERN text land with the code

The darn writer stencil stops saying the handoff runs the full verify command, and says it runs a verify scoped to the diff.
`pattern/PATTERN-test-speed.md`'s guard bound and `pattern/PATTERN-verified-tree.md`'s darn sites and `publish_verify` sentence are restated to match D2 to D4.

Rationale: both texts become false with this change.

## Constraints

- `PATTERN-verified-tree`: every site still runs through `verifytree.Verify`; the told `verify` stays each darn gate's base command.
- `PATTERN-gate-slot`: every scoped command runs through `Verify`'s slot, and `--guards` and the `lyx gate test` steps run inside the held slot.
- `PATTERN-fabric-git`: the wiring reads the merge base and changed paths through `internal/fabricengine`.
- `PATTERN-hub-wide-config`, `PATTERN-config-layers`: the new keys are hub-wide `darn.yaml` keys read through `configengine`.
- `PATTERN-refusal-way-forward`: each new refusal names its way forward, and a test reaches it.
- Operator rule: the import graph never decides the last gate for a diff touching production code.

## Auto-mode assumptions

- The board's "the whole-module suite still runs once, at Publish" means Merge-In's verify, since in darn Merge-In runs `verify` right before Publish and Publish runs only `publish_verify`.
- A tree-scoped diff's last gate is tree-scoped too: the 2026-10-20 decision settles the open question only for diffs touching production code, and the 2026-10-18 decision scopes every gate for non-production diffs.
- An empty task diff is tree-scoped with no targets.

## Open risks

- The gain is unmeasured: the build and three vet passes of the impact-scoped command still run at each gate.
- The scoped gate does not settle `darn-verify-parent-fix` or `darn-verify-tier-unrunnable`: a replayed failure in a package the task does not touch still fails Darn-Write, and a production diff still meets the module-wide `tmux` tier at Publish.

## Acceptance criteria

- With tree scope off, the writer and fix-round gates run the impact-scoped command, Merge-In runs the whole `verify`, and Publish runs `publish_verify`.
- A diff wholly under configured trees yields the tree-scoped command at the Darn-Write gate, the Darn-Burler-Fix gate, Merge-In and Finalize, with targets deduplicated and ordered, and Publish runs no `publish_verify` and removes a Publish failure record.
- With no Publish failure record present, a Merge-In after a writer-gate pass on the same tree with the same tree-scoped command skips through the verified-tree record.
- A diff touching one production path, including a rename whose old path is production, runs the impact-scoped command at the writer gates and the whole `verify` at Merge-In.
- A tree path matches only at a path boundary: `tools` does not claim `toolsx/a.go`, and a key written `tools/` matches `tools/a.go`.
- An empty diff yields the tree-scoped command with an empty `{targets}`.
- `impactset` naming a fallback makes the writer gate run the whole `verify`.
- The impact-scoped command's test and guard steps run through `lyx gate test`.
- `LoadConfig` refuses half-set tree scope, a template without `{targets}`, a malformed tree key, and either key set in the local layer, each with its way forward.
- `lyx gate test --guards` with no packages runs only the guard-marked tests, with packages runs both, and without `integration` among its tags is refused.
- Loom runs keep their gates and landing behaviour apart from the derived command's `lyx gate test` steps.
- The change description names the `lyx config darn --set` commands that turn tree scope on for this hub, such as `tree_scope: {tools: ./tools/..., docs: ""}` and `tree_scope_verify: "lyx gate test --tags integration --guards {targets}"`.
