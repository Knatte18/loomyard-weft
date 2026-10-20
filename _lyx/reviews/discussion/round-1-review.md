---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 Bound, D5, Open risks bullet 2
    summary: An old binary's re-seeded burler.yaml overwrites a migrated lenses.fans that equals the new template default, permanently reverting twill-discussion to the old six lenses, so the Open risk's claim that burler's values are dropped as kept-target and no lens value is lost is false.
    fix: In D4, replace the base-layer rule for these two moves with one that never overwrites a present lenses.yaml - at a base layer where lenses.yaml exists, its lenses and fans are kept whatever their value and burler's are dropped and reported as kept-target; the template-default overwrite applies only to a target started from the template in the same pass. State it as D4's bound. Rewrite Open risks bullet 2 to that rule, and add there that an operator edit made to burler.yaml through an old binary after migration is dropped, and that a re-seeded burler.yaml stays on Bolt until the new binary's next walk, which runs only at its next build change or an absent repo default, so the orch's post-landing check can see one. Add an acceptance criterion - a board dir holding a migrated lenses.yaml equal to the template plus an old-template burler.yaml reconciles to an unchanged lenses.yaml, burler.yaml deleted and the drop reported as kept-target.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D2
    summary: Once the first reconcile writes every lens and fan into the hub lenses.yaml, no later template change to a lens's prose or a fan's membership reaches that hub, where today every lens the hub file lacks follows the template; no Decision states this or its way forward.
    fix: Add to D2's Bound that reconcile keeps the file's value for every present lens and fan, so a later template edit to a lens or fan the hub file holds never reaches the hub, and name the way forward an operator or a landing task uses to take a template change (deleting the entry from the hub file so reconcile adds the template's, or editing it in). Rewrite the template header comment requirement in Scope to say the same.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 Bound, D3
    summary: An operator deleting a template lens or fan from a hub file can never make the deletion stick - D3's fill resolves it from the template at every load and D2's reconcile writes it back as added - and no Decision states this removed capability, which today's template header calls a supported permanent choice.
    fix: Add to D2's Bound - a lens or fan the template holds cannot be removed from a hub; deleting it from the file makes the next load resolve the template's value and the next reconcile write it back, reported added; an operator turns a fan off by pointing the fan key elsewhere or emptying the key. Make the Scope bullet on the template header name this bound.
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 third bullet
    summary: The new --set edge that knows any name under lenses and fans states no bound - which layers accept it, that a template name is a tune, whether a lens name may hold a dot, and that the module's CheckLoad only decodes, so a fan over the length cap or naming an undefined lens is written and refused only at the next loom or darn config load.
    fix: Add a Bound to D2 for --set - it writes the default or worktree layer only, the local layer refusing both guard keys; a name the template holds is a tune of it; a lens or fan name is one dot-free YAML key; and either the lenses module's CheckLoad resolves every fan, so the length cap and the undefined-lens refusal reject the write, or the Decision states that such a write is accepted and every fan key naming the fan refuses at config load. Add the chosen refusal to the --set acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D2, D3
    summary: Every load's warnUnknownKeys prunes against the spec's open maps alone, so each hub-only lens or fan D2 keeps is logged as a key no loader reads with a reconcile that removes it as the way forward, both false under D2; the record gives the per-name kind to yamlengine and configsync only.
    fix: In D2, state that the per-name kind is declared on the lenses module's configengine.Spec and honoured by every template comparison that spec drives - the load's fill and unknown-keys warning, Set's unknown-key check and the sparse-layer prune - so a hub-only lens or fan is never warned as unread. Add to the acceptance criteria that loading a lenses.yaml holding a hub-only lens logs no unknown-keys warning.
  - id: F6
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D3 Bound
    summary: The load expands only ${env:NAME} markers, so a bare $ is harmless, while hub-held prose D2 lets anyone add that holds a literal ${env:NAME} with NAME unset fails the whole lenses load and every burler, loom and darn consumer; D3's bound names the wrong character and covers template prose only.
    fix: Rewrite D3's Bound - lens prose, template and hub-held alike, passes through the load's ${env:...} expansion; a bare $ is untouched, and a ${env:NAME} marker in prose expands, or fails the load when NAME is unset and no default is given; no template prose holds such a marker.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D6, Open risks bullet 1
    summary: Whether the board commit stages burler.yaml's deletion is already settled - PullThenCommitWritten documents that a path which no longer exists stages its deletion and the applied burler move result's path reaches it - yet D6 and Open risks leave it open.
    fix: In D6, replace "the plan makes that commit stage the deletion if it does not already" with the fact that the board commit stages the deletion of a written path that no longer exists, as legacy-file removals already rely on. Delete Open risks bullet 1.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria, D1 Bound
    summary: D4's kept-target drop and D1's run_settings refusal have no acceptance criterion that would fail if the behaviour were wrong, and D1's bound omits that a run_settings burler object left empty, dropped today, is now refused too.
    fix: Add two criteria - a board dir holding a non-default lenses.yaml and a burler.yaml keeps lenses.yaml unchanged, deletes burler.yaml and reports the drop as kept-target, and likewise at a sparse layer for any target value; and run settings holding a burler object, empty or not, are refused as naming an unregistered module. Extend D1's Bound to say an empty burler object is refused as well.
  - id: F9
    severity: NIT
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: The kept-target line D4 names as the drop's report renders the whole dropped lenses mapping and the kept one, every lens's prose, on one line, so a dropped tuned value cannot be read from it.
    fix: In D4's Bound, state that the kept-target line for these two moves names the dropped and kept keys without rendering their mappings, or say that the line carries both whole mappings so the operator compares the files.
---

The target is the discussion record and support log of `lenses-module`, judged against the rubric and the board entry.
Scope matches the board entry's Want and the related entries; no In item is unrequested, and nothing in the record needs relocating to the support log.

### [BLOCKING] Old-binary re-seed reverts the seven-lens fan

**Location:** `_lyx/discussion/decision-record.md` D4 Bound, D5, Open risks bullet 2
**Issue:** The hub's `standard` and `full` fans equal the template's (`_board/_lyx/config/burler.yaml` against `internal/burlerengine/template.yaml:314-330`), so after migration and reconcile the hub `lenses.fans` equals the new template's `fans`.
An old binary's board pass re-seeds an absent seed-only `burler.yaml` from its template (`internal/configsync/configsync.go:594-600`), which holds the six-lens `twill-discussion`.
At the new binary's next walk, `takeKey` finds the target equal to its default by `nodesEqual`, which ignores key order (`internal/configsync/movekeys.go:216-224`, `:288-318`), and takes the `overwritten` branch.
Reconcile then carries every present fan list whole, so the hub stays on the old fan for good, against D5's purpose and the Open risk's "no lens value is lost".
Bound-breaker also found that an operator edit made through an old binary after migration is dropped, and that the re-seeded file waits on Bolt until the new binary's next build change (`internal/hubreconcile/walk.go:81-91`).

### [MEDIUM] Template edits stop reaching a hub

**Location:** `_lyx/discussion/decision-record.md` D2
**Issue:** Reconcile keeps every present value (`internal/yamlengine/reconcile.go:15-17`), and after D4 and D2 the hub file holds every lens and fan.
The template's lenses changed in eight recent commits (`git log -- internal/burlerengine/template.yaml`), and the hub's prose already differs from the template's, for example in naming `CONSTRAINTS.md` where the template names PATTERN entries.
Today a lens the hub file lacks follows the template; after this task none does, and the record does not say so.

### [MEDIUM] A template lens or fan cannot be deleted

**Location:** `_lyx/discussion/decision-record.md` D2 Bound, D3
**Issue:** `FillMissing` appends a missing template key per name (`internal/yamlengine/fill.go:97-101`), and D2's reconcile reports and writes it as added.
Today's header calls deleting a standard lens "a supported, permanent choice" (`internal/burlerengine/template.yaml:1-7`), and `pattern/PATTERN-config-strictness.md` names the same effect for `runprofile`.
D2's Bound covers only hub-only entries.

### [MEDIUM] `--set` on any name has no bound

**Location:** `_lyx/discussion/decision-record.md` D2 third bullet
**Issue:** `--set` is checked by the module's own load (`internal/configcli/configcli.go:253-266`), and burler's load only decodes; the length cap and the undefined-lens refusal live in fan resolution (`internal/burlerengine/config.go:126-150`).
So a bad fan is written and refused later by `ValidateFanKeys` at every loom or darn start that names it.
The layers `--set` accepts and the name grammar are unstated.

### [MEDIUM] Hub-only entries are warned as unread

**Location:** `_lyx/discussion/decision-record.md` D2, D3
**Issue:** `warnUnknownKeys` runs `yamlengine.Prune(spec.Template, layer.Bytes, spec.OpenMaps...)` at every load of a default or worktree layer (`internal/configengine/config.go:241-276`).
It logs "no loader reads them", with a reconcile that removes them as the way forward, for every lens or fan the template lacks.
D2 gives the per-name kind to `yamlengine` and `configsync` and is silent on `configengine.Spec`.

### [LOW] Expansion bound names the wrong exposure

**Location:** `_lyx/discussion/decision-record.md` D3 Bound
**Issue:** `envMarkerRe` matches only `${env:NAME}` and `${env:NAME:-default}` (`internal/yamlengine/resolve.go:19`), and an unset required marker fails the load (`resolve.go:148`).
Burler's `StackLayers` load never expanded prose (`internal/burlerengine/config.go:76-87`).

### [LOW] Deletion staging is already settled

**Location:** `_lyx/discussion/decision-record.md` D6, Open risks bullet 1
**Issue:** `PullThenCommitWritten` documents "A path that no longer exists stages its deletion" (`internal/fabricengine/bolt.go:178`), and `reconcileHubWide` passes every applied result's path, the burler move's included (`internal/hubreconcile/walk.go:203-212`).

### [LOW] Two behaviours lack a criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, D1 Bound
**Issue:** The first criterion covers only an absent `lenses.yaml`, so nothing pins the kept-target drop, and no criterion pins D1's refusal.
`moveRunSettings` leaves `burler: {}`, which `PrepareRunSettings` reaches before its empty-object skip and refuses (`internal/configreg/runsettings.go:46-50`), so an empty object dropped today is now refused.

### [NIT] Kept-target line renders whole mappings

**Location:** `_lyx/discussion/decision-record.md` D4 Bound
**Issue:** `nodeText` flattens a mapping node onto one line (`internal/configsync/movekeys.go:322-331`), and `takeKey` renders both the dropped and the kept value (`movekeys.go:208,220`).

## Rejected

- Walkthrough's MEDIUM rating of the re-seed revert: merged into F1 at BLOCKING, since the revert is silent, permanent and contradicts a claim the record makes.
- Bound-breaker's note that a standalone dry run reports the whole template as added: no criterion or Decision claims otherwise.
