---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4, Publish skip sentence
    summary: D4 says Merge-In and Finalize classify the diff at each call but never says when the Publish skip seam classifies, so a seam fixed at wiring or carried from an earlier pass skips publish_verify on a diff that a PR-Gate rework turned production
    fix: In D4, replace the Publish sentence's opening with "At each Publish call, after its moved-parent check, the seam classifies the task diff as D2 does, against the merge base Merge-In's merge left; when that diff is tree-scoped Publish skips `publish_verify` ..." and add to D4's Bound that the skip covers exactly the diff classified at that call, never one classified at run start or at an earlier Publish; add an Acceptance criterion that a run tree-scoped at its first Publish whose PR-Gate rework adds a production path runs `publish_verify` at the next Publish.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 and the last Acceptance criterion
    summary: D1's config shape does not survive lyx config; tree_scope is not declared an open map, so --set refuses tree_scope.<tree> and reconcile drops the hub's tree keys, and the half-set refusal rolls back a --set of either key alone, so on and off each need both pairs in one call
    fix: In D1 add "`tree_scope` is an open map of `darn.yaml` (`ConfigOpenMaps`), so `lyx config darn --set tree_scope.<tree>=<target>` writes a tree and reconcile carries the hub's trees whole"; add "tree scope is turned on, and off, by one `lyx config darn` call carrying both keys' `--set` pairs, since a write that leaves one key set without the other is refused and rolled back", and state how a set `tree_scope` map is emptied to turn scope off; rewrite the last Acceptance criterion to name one command in `--set` spelling, such as `lyx config darn --set tree_scope.tools=./tools/... --set tree_scope.docs= --set "tree_scope_verify=lyx gate test --tags integration --guards {targets}"`.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1, category sentence
    summary: D1 says both keys are CategoryGuard as verify is, but verify is CategoryExecutable, and tree_scope_verify runs a shell command, so the premise is false and a literal plan gives the executable key the wrong category
    fix: In D1 replace "Both keys are closed to the local layer (`CategoryGuard`), as `verify` is" with "Both keys are closed to the local layer, `tree_scope_verify` as `CategoryExecutable`, as `verify` is, and `tree_scope` as `CategoryGuard`, since widening it hollows the gate".
  - id: F4
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 tree-key rules and D2 ordering
    summary: A tree key written ./tools is neither trimmed nor refused and silently matches nothing, and keys tools and tools/ both trim to tools, so tree-path order ties and the expanded string can vary between sites
    fix: In D1 extend the refusal list to a tree key holding a `.` segment, such as `./tools`, and to two keys that are equal after the trailing `/` is trimmed; add an Acceptance criterion row for each refusal.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: No criterion covers D3 step 3's no-tmux-pass for a tree-scoped arrival with a publish_verify failure record, or a diff the writer gates classify tree-scoped that Merge-In classifies production after a conflict resolution
    fix: Add two Acceptance criteria - a tree-scoped Darn-Write arrival with a `publish_verify` failure record replays the failing tests and runs the tree-scoped command with no `tmux` package pass; a diff tree-scoped at the writer gates whose Merge-In merge resolution touches a production path runs the whole `verify` at Merge-In and `publish_verify` at Publish.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Scope In and Out against D3 last paragraph
    summary: Scope's Out excludes loom's recipe and calls loom's fix-round gate already scoped, while D3 changes the command loom's round gate runs, and Scope In names that change only through whose derived command
    fix: In Scope's In list, extend the impact-scoped bullet with "this changes the command loom's Webster-Burler-Fix round gate runs too, and nothing else of loom"; in the Out bullet, change "Loom's recipe." to "Loom's recipe, apart from that derived command."
  - id: F7
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5
    summary: D5 leaves open what lyx gate test --guards does when the marker scan fails, such as a misplaced //lyx:guard, and which directories it scans
    fix: In D5 add "`--guards` scans every package directory of the module at the `-C` directory; a scan error refuses the verb before any slot, naming the file and line and the way forward to move the marker directly above its `func Test…` line".
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 Rationale
    summary: D4's rationale says the whole-module suite runs once, at Merge-In, but a production diff touching a non-.md file outside a Go package directory makes impactset fall back, so D3 step 5 runs the whole verify at every writer and fix-round arrival too
    fix: In D4's Rationale replace "runs once, at Merge-In right before Publish, for every diff that touches production code" with "runs at Merge-In right before Publish for every diff that touches production code, and at the writer gates too wherever `impactset` falls back".
---

The holistic pass held the record against the board entry's three operator sections and the rubric.
The tree-scope design follows the 2026-10-18 decision (tree, never import graph), and D2's and D4's bounds state that tree-scoped diffs no longer get a whole-module run before landing.
D5 departs from the board's "scan packages are config" and says so.
No Decision edits the task's own board entry.
Every Turn 7 fix and every rejected alternative in the support log that carries a requirement is already in `decision-record.md`, so nothing needs to move.

### [MEDIUM] Publish skip seam has no stated evaluation point

**Location:** `decision-record.md` D4, Publish skip sentence.
**Issue:** D4 says "resolves at each call" for Merge-In and Finalize only.
For Publish it says "In a tree-scoped darn run", which reads as a property of the run.
Today `internal/landingshed/publish.go:119` reads a static `p.deps.Config.PublishVerify`, and `wireDarn` (`internal/loomcli/darnwiring.go`) wires once per process.
Counterexample: the first Publish is tree-scoped, PR-Gate rejects, and the Darn-Write rework adds production code.
Merge-In then runs the whole `verify`, but a seam fixed earlier still skips `publish_verify`, so a production diff lands with no `tmux` tier and its failure record is removed.
Four lenses raised this independently.

### [MEDIUM] Tree scope cannot be turned on or off the way D1 and the last criterion say

**Location:** `decision-record.md` D1 and the last Acceptance criterion.
**Issue:** `ConfigOpenMaps` (`internal/darnengine/config.go`) lists only the model and fan keys.
Without `tree_scope` declared as an open map, three things break, per `internal/yamlengine/doc.go`:
- `SetValues` refuses `tree_scope.tools` as a key the template does not know.
- `Reconcile`, which `hubreconcile.Ensure` runs after a binary change, reports the hub's trees as removed and drops them.
- A `--set` value not led by `[` is written as a string, so the criterion's `{tools: ..., docs: ""}` is no `--set` spelling.

`setChecked` and `setHubWide` (`internal/configcli/configcli.go:244-270`, `hubwide.go:330-350`) roll back a write after which `LoadConfig` fails with a new error.
D1's half-set refusal therefore rolls back a `--set` of either key alone, and turning scope off has the same problem.

### [MEDIUM] D1's category premise is false

**Location:** `decision-record.md` D1.
**Issue:** `verify` is `configengine.CategoryExecutable` (`internal/darnengine/config.go` `ConfigSpec`), not `CategoryGuard`.
`internal/configengine/layers.go:48-55` defines executable as a key that runs an executable, and `tree_scope_verify` is one.
Both categories refuse the local layer, so the protection holds.
What is wrong is the classification and the refusal's wording.

### [LOW] Tree-key normalisation gaps

**Location:** `decision-record.md` D1 rules, D2 ordering.
**Issue:** `git diff --name-only` (`internal/fabricengine/configchanges.go:88-99`) never emits a `./` prefix, so a `./tools` key matches nothing and tree scope stays off with no refusal, the same failure the trailing-slash trim fixed.
`tools` and `tools/` are distinct YAML keys that collapse to one tree.
That breaks D2's "the same diff always yields the same string", which the verified-tree skip across sites relies on.

### [LOW] Acceptance criteria miss two new behaviours

**Location:** `decision-record.md` Acceptance criteria.
**Issue:** D3 step 3's "no trailing `tmux` package pass", added in Turn 7, has no criterion.
D2's merge-base definition and D4's merge-time diff use one definition evaluated at two moments, so they agree except where Merge-In's merge resolution changes the paths.
That case, which flips the classification to production, has no criterion either.

### [LOW] Scope does not name the change to loom's round gate

**Location:** `decision-record.md` Scope against D3's last paragraph and criterion 11.
**Issue:** The Out bullet "Loom's recipe" is literally true, since the recipe file is unchanged.
But its rationale calls loom's fix-round gate already scoped, while D3 changes the command that gate runs (`internal/impactset/doc.go`, `internal/loomshed/verifygate.go`).
Scope, D3 and criterion 11 agree in substance and disagree in wording.

### [LOW] `--guards` scan-error behaviour is undecided

**Location:** `decision-record.md` D5.
**Issue:** `FindGuardTests` returns an error naming the file for a misplaced marker (`internal/impactset/guards.go`), and the round gate turns it into a writer finding (`internal/loomshed/verifygate.go:86-89`).
D5 does not say whether the verb refuses or skips on that error, or which directories it scans, so `PATTERN-refusal-way-forward` has no stated refusal to hold.

### [LOW] D4 rationale overstates "once"

**Location:** `decision-record.md` D4 Rationale.
**Issue:** `impactset` falls back for any changed file that is neither `.md` nor under a Go package directory (`internal/impactset/doc.go`), for example under `contracts/`.
For such a production diff, D3 step 5 runs the whole `verify` at every writer and fix-round arrival as well.

## Rejected

- interactions F2, that `LoadConfig` should catch a `tree_scope_verify` using `--guards` without `integration`: rejected, because `LoadConfig` cannot parse a shell command. A misconfigured hub `verify` fails at the gate in the same way today, and D1 already has the key's template comment state the requirement.
- Focus 2's sub-question, a diff that is tree-scoped only through config the hub controls: not a defect. Both keys are closed to the local layer and `run_settings`, and D2's Bound states that the hub bounds the skip by what it lists. The remaining gap is F1.
- Focus 4, guards removed by D1 and D5: not a defect. The whole-module run at Publish is bounded in D2's Bound, and the `tmux` tier in D4's Bound, each in the Decision that removes it. Open risks only cross-reference other board entries.
