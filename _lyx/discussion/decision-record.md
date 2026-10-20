# Decision record — verify-diff-scope

## Goal

A darn run stops paying for the whole-module suite at gates where its diff does not need it: the writer and fix-round gates run the diff's impacted set plus the repo-wide scans, and a diff that touches no production code runs only its trees' tests plus the scans at every site, landing included.

## Scope

In:

- Darn's verify sites: the Darn-Write gate, the Darn-Burler-Fix gate, Merge-In's verify, Finalize's catch-up verify, and Publish's `publish_verify` in a darn run.
- Tree scope: hub-wide `darn.yaml` config naming the non-production trees and the tree-scoped command, the classification of the task diff against it, and the command it yields.
- The impact-scoped form of darn's two verify gates, built on `internal/impactset`, whose derived command moves its test steps onto `lyx gate test`;
  this changes the command loom's round-form verify gate runs at Webster-Burler-Fix and Merge-Burler-Fix too, and nothing else of loom.
- A `--guards` flag on `lyx gate test`.
- The module-loader check on `lyx config`'s hub-wide editor save, which D1 needs and which applies to every hub-wide module.
- The stencil, doc and PATTERN background text that describe these sites.

Out:

- Loom's recipe, apart from the derived command of `impactset` that loom's fix-round gate runs.
  The board's Seen sentence "the same command runs for every darn and at loom's Merge-In and Publish" is read as darn runs passing through the landing rows, since loom's Merge-In runs the plan's own `## verify:`, which the plan writer scopes per task, and loom's card, batch and fix-round gates are already scoped, the fix-round gate changing only the command `impactset` derives.
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

Both keys are closed to the local layer, `tree_scope_verify` as `CategoryExecutable`, as `verify` is, and `tree_scope` as `CategoryGuard`, since widening it hollows the gate; neither a task's tree nor its `run_settings` can set them.
`tree_scope` is an open map of `darn.yaml` (`ConfigOpenMaps`), so `lyx config darn --set tree_scope.<tree>=<target>` writes a tree and reconcile carries the hub's trees whole.
Tree scope is on only when both are non-empty.
A trailing `/` on a tree key is trimmed.
`LoadConfig` refuses the following, each reaching the operator with its way forward from the prime:
a non-empty `tree_scope_verify` while `tree_scope` holds no tree, naming `lyx config darn --set tree_scope.<tree>=<target>` in the same call, or `lyx config darn --set tree_scope_verify=` to turn tree scope off;
a non-empty `tree_scope_verify` without `{targets}`, naming `lyx config darn --set tree_scope_verify=<command holding {targets}>`;
and a tree key that is empty, absolute, or holds an empty, `.` or `..` segment, such as `tools//x` or `./tools`, or a backslash, or two tree keys equal after the trailing `/` is trimmed, such as `tools` and `tools/`.
The malformed-key refusal names the offending key as written, and the duplicate-key refusal names both keys;
neither carries a way forward in `LoadConfig`'s own text, since `LoadConfig` cannot tell which caller it serves, and each caller adds the one its state allows.
At run start, where the hub file holds the key, the way forward is `lyx config darn`'s editor, since `--set` cannot remove a key;
at an editor save it is the editor refusal's way forward below;
a `--set` write that adds such a key is rolled back when `LoadConfig` then fails with a new error, and that refusal's way forward is `newLoadFailure`'s `--set` one, which for a duplicate means the spelling the file already holds.
Only one trailing `/` is trimmed, so `tools//` is refused for its empty segment.
A `tree_scope` without `tree_scope_verify` is accepted and leaves tree scope off, and so are both keys empty, as the template ships them.
Both hub-wide write paths of `lyx config darn`, `--set` and the editor, load the written file through darn's own loader and roll back a write after which `LoadConfig` fails with a new error, and `--set` never removes a key from an open map, so:
tree scope is turned on by one `lyx config darn` call carrying both keys' `--set` pairs, or by the trees first and the command after;
it is turned off by `lyx config darn --set tree_scope_verify=` alone, which leaves the trees inert;
and a tree is removed only by editing the map in `lyx config darn`'s editor, where removing the last tree while `tree_scope_verify` is non-empty requires turning tree scope off first with `lyx config darn --set tree_scope_verify=`, or clearing `tree_scope_verify` in the same editor save.
The editor path does not load through darn's loader today: `copyStagedToHub` in `internal/configcli/hubwide.go` gains the `module.CheckLoad` check before and after the write that `setHubWide` runs, rolling back through `rollBack`, so an editor save must leave the file loading or failing with the error it failed with before.
`editHubWide` today removes the staging copy after every write error except `errHubFileChanged`, so the new load refusal gets its own sentinel error, checked the way `errHubFileChanged` is, and keeps the staging copy.
The editor refusal carries its own way forward, never `newLoadFailure`'s `--set` one: it names the load error and the staging copy's path, and, for a last tree removed while `tree_scope_verify` is non-empty, says to run `lyx config darn --set tree_scope_verify=` first;
otherwise it says to copy the staging copy out of its path, re-open `lyx config darn` and re-apply the edit from that copy with the named key fixed, because every open of the editor overwrites the staging copy with the hub file's bytes.
The template ships both empty.

Rationale: the board forbids lyx hard-coding Go or a tree layout, and asks that the trees and the scoped command be config of the module that owns the recipe.
Closing both keys to the local layer keeps a task from widening its own non-production trees or choosing its own command.

### D2 — Classification of the task diff

The task diff is every warp path changed from the pair's merge base with its recorded parent to HEAD, by new and old path.
`fabricengine.ChangedPaths` lists only a rename's new path, so the classification uses a new `fabricengine` read beside it, run with `-z` and `--no-renames`, which reports both sides of a rename; a rename or deletion out of production code then counts as production.
`ChangedPaths` and its other readers keep their current output.
The new read reports each path verbatim, as `-z` does, never in core.quotePath's C-quoted form, so a non-ASCII path under a tree lies in that tree.
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
At each Publish call, after its moved-parent check, the seam classifies the task diff as D2 does, against the merge base Merge-In's merge left; when that diff is tree-scoped Publish skips `publish_verify` through a seam the darn wiring tells landingshed, and takes the path of an unset `publish_verify`, which removes the Publish failure record; a nil seam, loom's, runs it as today.

Rationale: the whole-module suite runs at Merge-In right before Publish for every diff that touches production code, and at the writer gates too wherever `impactset` falls back, so the import graph never decides the last gate.
For a tree-scoped diff the tree-scoped command is the hub's whole statement of what such a diff needs.

Bound: the Publish skip covers exactly the diff classified at that call, never one classified at run start or at an earlier Publish.
A tree-scoped darn run lands with no `tmux`-tier run, and its Merge-In does not whole-verify parent code merged into the task; the parent's code was verified by the parent's own landing, which for an unlanded task or umbrella parent is still ahead of it.
A hub that wants more for such a diff puts it in `tree_scope_verify`.

### D5 — `lyx gate test --guards`

`lyx gate test` gains `--guards`: after the named packages it runs every top-level test marked `//lyx:guard`, by name in its own package, inside the same slot.
With `--guards` the package list may be empty, and the verb refuses `--guards` unless its tags include `integration`, the tier every guard compiles under.
The marker scan is `impactset.FindGuardTests`, the scan the round gate and the card gate already use.
`--guards` scans every package directory of the module at the `-C` directory; a guard-scan error refuses the verb before any slot, naming the file and the way forward its kind gives: move a misplaced marker directly above its `func Test…` line, compile a marked test's file under the `integration` tag, or, for the parse kind, the way forward below.
The two marker kinds name the line too, and the parse kind names it only when the parser reports one.
A test file that cannot be read arrives as the parse kind, with the operating system's error and no line, and `GuardScanError` carries no kind that tells it from a parse failure, so the parse kind's way forward is to make the file readable or fix it until it parses.
A scan failure that is not a guard-scan error, refuses the verb before any slot with the error: an unreadable package directory names that directory, and a failed package enumeration names the `-C` module directory and the enumeration's error, and the way forward is to make the directory readable or fix the module until `go list ./...` succeeds there.

Rationale: the board's open item names the scan packages as config.
This settles it differently: a hand-kept package list goes stale whenever a scanning test is added elsewhere, while the `//lyx:guard` marker is already the repository's contract for a test that reads files outside its package.
The hub's config still decides whether the scans run, through `tree_scope_verify`.

Bound: an unmarked scanning test is not run, as for the round gate today.

### D6 — Prompt and PATTERN text land with the code

The darn writer stencil stops saying the handoff runs the full verify command, and says it runs a verify scoped to the diff.
`pattern/PATTERN-test-speed.md`'s guard bound and `pattern/PATTERN-verified-tree.md`'s darn sites and `publish_verify` sentence are restated to match D2 to D4.
`internal/configcli/doc.go`'s sentence that every `--set` write is checked through the module's own loader is restated to cover the hub-wide editor save too.

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

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - editor-refusal-keeps-edit-false (class design, severity MEDIUM, open in rounds 3)
  - no-targets-refusal-covers-empty-value (class consistency, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - darn-tree-scope-drops-verify-way-forward (class scope, severity MEDIUM, open in rounds 2)
  - landing-sites-editor-way-forward-untested (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- With tree scope off, the writer and fix-round gates run the impact-scoped command, Merge-In runs the whole `verify`, and Publish runs `publish_verify`.
- A diff wholly under configured trees yields the tree-scoped command at the Darn-Write gate, the Darn-Burler-Fix gate, Merge-In and Finalize, with targets deduplicated and ordered, and Publish runs no `publish_verify` and removes a Publish failure record.
- With no Publish failure record present, a Merge-In after a writer-gate pass on the same tree with the same tree-scoped command skips through the verified-tree record.
- A diff touching one production path, including a rename whose old path is production, runs the impact-scoped command at the writer gates and the whole `verify` at Merge-In.
- A tree path matches only at a path boundary: `tools` does not claim `toolsx/a.go`, and a key written `tools/` matches `tools/a.go`.
- An empty diff yields the tree-scoped command with an empty `{targets}`.
- `impactset` naming a fallback makes the writer gate run the whole `verify`.
- The impact-scoped command's test and guard steps run through `lyx gate test`.
- A run tree-scoped at its first Publish whose PR-Gate rework adds a production path runs `publish_verify` at the next Publish.
- A tree-scoped Darn-Write arrival with a `publish_verify` failure record replays the failing tests and runs the tree-scoped command with no `tmux` package pass.
- A diff tree-scoped at the writer gates whose Merge-In merge resolution touches a production path runs the whole `verify` at Merge-In and `publish_verify` at Publish.
- `LoadConfig` refuses a non-empty `tree_scope_verify` with no tree, a non-empty template without `{targets}`, a malformed tree key, naming it as written, and either key set in the local layer, each with its way forward, and accepts trees with no `tree_scope_verify` as tree scope off.
- `LoadConfig` loads both keys empty, as the template ships them, with tree scope off, and `lyx config darn --set tree_scope_verify=` is accepted when `tree_scope` holds no tree.
- `LoadConfig` refuses a tree key holding a `.` segment, such as `./tools`, or an empty segment, such as `tools//x` and `tools//`.
- `LoadConfig` refuses two tree keys equal after the trailing `/` is trimmed, such as `tools` and `tools/`, naming both keys.
- A diff path outside ASCII under a tree, such as `docs/ä.md` under `docs`, lies in that tree.
- A `lyx config darn` editor save that leaves `tree_scope_verify` non-empty with an emptied `tree_scope`, or holds `tools` and `tools/`, is refused, leaves the board file unchanged and keeps the edit at the staging copy.
- `lyx config darn --set tree_scope.tools=./tools/...` writes the tree, and reconcile keeps the hub's trees.
- `lyx gate test --guards` with no packages runs only the guard-marked tests, with packages runs both, and without `integration` among its tags is refused.
- `lyx gate test --guards` over a module whose guard scan fails is refused before any slot, naming the file and line for a marker kind, the file for an unreadable test file, and the directory for an unreadable package directory.
- Loom runs keep their gates and landing behaviour apart from the derived command's `lyx gate test` steps.
- The change description names the one command that turns tree scope on for this hub, `lyx config darn --set tree_scope.tools=./tools/... --set tree_scope.docs= --set "tree_scope_verify=lyx gate test --tags integration --guards {targets}"`, and the one that turns it off, `lyx config darn --set tree_scope_verify=`.
