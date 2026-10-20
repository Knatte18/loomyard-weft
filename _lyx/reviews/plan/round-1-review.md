---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/plan/08-retire-burler-module.md:51
    summary: Card 8 pins a dry-run report that D4 rules out (the lenses module loop reporting the template seed's names as added while lenses.yaml is absent), and the departure is recorded only in Shared Decisions, with no Added after Discussion entry in the decision record
    fix: Either carry D4 as written or route the departure. To carry it, add `internal/configsync#ReconcileHubWideAt` to card 8's Edit list (this passes `lyx loom validate-plan --overlay` with no new blocking finding), and add an Intent bullet saying that on a dry run the module loop reports no seed additions for a module whose absent board file the same pass's key moves would create. The rule holds for every move target, not lenses alone. Change the dry-run row (line 51) to assert the `lenses` result carries no `Added`, and drop the D4 bullet from `00-overview.md`'s `## Shared Decisions` and the matching bullet at card 8 line 46. To route it instead, the plan writer asks its parent to record the departure with `lyx loom decision add` as an `Added after Discussion` entry, and keeps the row only once the record carries that entry.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:guard-reach, lens:affected-set
    location: _lyx/plan/05-burler-reads-lenses.md:57
    summary: TestRunCmd's subtest refusing a cluster-fan reviewer on an engine without forks names template fan standard over a zero Config, which resolves today only through burler's per-name template fallback; with lenses.ResolveFan having no fallback, the type-forced swap the card prescribes makes validateReview refuse the unknown fan before the engine refusal the subtest asserts
    fix: In card 5's Intent, after the sentence at line 57, add that TestRunCmd's subtest "a cluster-fan reviewer on an engine without forks is refused naming the engine" (internal/burlercli/wiring_test.go:834-839) builds a `lenses.Config` that defines the `standard` fan and its lenses, or loads the template through `lenses.Load`, because a zero `lenses.Config` no longer resolves a template fan.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/05-burler-reads-lenses.md:60
    summary: The burler.yaml text sweep misses two members, burlerengine/doc.go lines 77 and 88 in the fixer-fan section (card 5 limits the doc change to the cluster fan-out section) and the fan-key clause of docs/overview.md line 551 (card 6 names only the discussion_lenses and plan_lenses clause)
    fix: In card 5's Intent at line 60, widen the burlerengine/doc.go sentence to cover the "# The fixer fan" section too, so the FixFan sentence (doc.go:77) and the define-it way forward (doc.go:88) name lenses.yaml. In card 6's Intent at line 48, add that docs/overview.md's loom config paragraph (line 551) names lenses.yaml for the fans the eight `*_reviewer_fan` and `*_fixer_fan` keys name, besides the discussion_lenses and plan_lenses clause.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/03-lenses-module.md:62
    summary: Card 3 moves ResolveFan as it stands without settling its refusal prefix, so the lenses module keeps the burler prefix on its unknown, empty, over-cap and undefined-lens refusals, and no test forces the change
    fix: In card 3's ResolveFan bullet (line 62), state that each refusal carries the lenses prefix that Load's errors carry, never the burler prefix, and have TestResolveFan's refusal rows assert the prefix.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set, handler
    location: _lyx/plan/02-configengine-per-name-spec.md:11
    summary: configengine's package doc has an Open maps section documenting Spec.OpenMaps, but card 2 documents the new PerNameMaps field only in the Spec comment and does not target internal/configengine/doc.go
    fix: Add `internal/configengine/doc.go` to card 2's Edit list after `internal/configengine#setSparse` (this passes `lyx loom validate-plan --overlay` with no new blocking finding), and add an Intent sentence saying the package doc gains a per-name maps paragraph beside "# Open maps", naming the template comparisons that honour the field.
  - id: F6
    severity: LOW
    class: decision
    origin: handler, lens:way-forward
    location: _lyx/plan/01-yamlengine-per-name-maps.md:39
    summary: D2 says --set takes lenses.<name> as a string scalar and fans.<name> as a flow list, but card 1 spells every per-name value through scalarOrListNode, so a lens prose starting with [ is written as a list and refused by the strict decode, and quoting it as the refusal advises still yields a list
    fix: In card 1's SetValues and SetSparse bullet (line 39), make a per-name value's shape follow the kind of the template's entries at P. A mapping of scalars (lenses) takes the value as a string scalar whatever its first character; a mapping of lists (fans) takes it as a YAML flow list through replaceSequence, with a non-list value refused. Add the matching `TestSetValues` row, a lens prose starting with `[` written as a string.
  - id: F7
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/03-lenses-module.md:63
    summary: Because newLoadFailure compares the whole CheckLoad error text, a --set that fixes one of two already-failing fans changes the joined text and is refused with --set the key to a value it takes, a way forward that cannot succeed for a partial fix
    fix: In card 3's CheckLoad bullet (line 63) and in the doc.go sentence on fan resolution, state that a lenses.yaml with several failing fans takes every fix in one `--set`, or one `--edit`, because a partial fix changes the joined failure and is rolled back.
  - id: F8
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/01-yamlengine-per-name-maps.md:39
    summary: The non-mapping state Reconcile refuses (such as lenses holding a list) recurs unguarded on --set; SetValues marshals from the template tree and overwrites the operator's value, and setChecked lets it through because the before load already failed and the after load passes
    fix: In card 1's SetValues and SetSparse bullet, add that a value at P that is neither a mapping nor null is refused with an error naming P and its shape, as setOpenMapEntry refuses an open map holding a list. Add a `TestSetValues_Refusals` row whose existing file holds a list at P.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/01-yamlengine-per-name-maps.md:36
    summary: Card 1's Reconcile refusal for a non-mapping at P names the path and shape but no way forward, though it fails lyx config reconcile and, through the hub walk, every start verb, while the sibling open-map refusal names one
    fix: In card 1's non-mapping refusal sentence (line 36), add that the error ends with the way forward `rewrite P as a map of name to value`, and have the TestReconcile wantErr row assert it.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/08-retire-burler-module.md:40
    summary: The newly reachable refusal of run settings holding a burler object names no next step, and the moved lenses.local.yaml that the lenses load refuses has no named way forward either
    fix: In card 8's run-settings bullet (line 40), state the next step, which is to remove the `burler` object from the entry's run_settings, since both keys are guard keys a run setting cannot hold. Name that step in the refusal's text for an unregistered module that a declared key move retired. In the bound at line 45, add that the way forward for a refused `lenses.local.yaml` is to delete that local file.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/01-yamlengine-per-name-maps.md:40
    summary: Card 1 prescribes that SetValues turns a null at P into an empty mapping before appending the name, but names no row that reaches that branch; the TestReconcile null row covers Reconcile only
    fix: Add a TestSetValues row to card 1's rows list. Its existing file holds `names:` as null, a pair sets `names.x=prose`, and the row asserts the merged file holds `names` as a mapping with `x`.
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/03-lenses-module.md:87
    summary: Card 3's new TestPrintResolved subtest asserts that a lenses template lens and fan are annotated default, behaviour TestPrintResolved's existing subtest annotates each unit with its layer already pins for every module through the same Provenance path
    fix: Drop the TestPrintResolved subtest and `internal/configcli#TestPrintResolved` from card 3's Edit list. The acceptance criterion that `lyx config lenses` shows the resolved file rests on the lenses registration, which the TestModules rows pin, and on the generic annotation subtest.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/06-loom-reads-lenses.md:52
    summary: Card 6's loom table rows resolve template fans over lenses.Load of an empty dirs, which read as configengine.Dirs{} configengine.load refuses for naming neither an anchor nor a board dir
    fix: Reword card 6's sentence to say the rows resolve their template fans with `lenses.ResolveFan` over `lenses.Load` of a temp dir holding no `_lyx/`, which degrades to the template.
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/04-configsync-per-name-reconcile.md:24
    summary: hubreconcile's logResult logs every report field of a Result, keptTarget included, but card 4 does not add the new KeptExtra field, so a hub-only lens a walk keeps never appears in the walk's log
    fix: Add `internal/hubreconcile#logResult` to card 4's Edit list. Add an Intent sentence saying it logs `keptExtra` beside `keptTarget`, and run the overlay check before applying, since this adds a target.
---

### [MEDIUM] Card 8 pins a dry-run report D4 rules out

**Location:** `_lyx/plan/08-retire-burler-module.md:51`, with `_lyx/plan/00-overview.md:28` and card 8 line 46.

**Issue:** D4 says: "A dry run writes nothing and reports the two moves, but not the additions, which the module loop reports only once `lenses.yaml` exists."
`lyx config reconcile --layer default` runs `ReconcileHubWideAt` with `SeedDefaults: true` on its dry run (`internal/configcli/hubwide.go:175-180`), as the test's `board` call does (`internal/configsync/configsync_test.go:1287`).
The moves write nothing on a dry run, so the module loop meets an absent `lenses.yaml` and takes the absent-file branch, reporting the template seed's names as added with seed `template` (`internal/configsync/configsync.go:570-600`).
The plan sees this and records in `## Shared Decisions` that it pins the seed report instead of meeting D4, on the ground that a fix would be a one-module special case.
That ground fails: the rule can be stated for every target a pass's moves would create, so the existing claude moves take it too.
It is also a change to a settled bound, which the plan writer's stencil routes through the parent as an `Added after Discussion` entry, never through the plan.
The pinned preview misleads as well: it shows a template seed, while the apply writes burler's moved values and adds only the names those lack.

### [LOW] TestRunCmd's cluster-fan engine refusal loses its reach

**Location:** `_lyx/plan/05-burler-reads-lenses.md:57`.

**Issue:** `internal/burlercli/wiring_test.go:834-839` pairs `cluster-fan: standard` with `burlerengine.Config{}`.
`validateReview` resolves the fan before any spawn (`internal/burlerengine/profile.go:172-178`), and today's `ResolveFan` resolves it from the template (`internal/burlerengine/config.go:117-123`).
`lenses.ResolveFan` reads the loaded `Config` alone, so a `lenses.Config{}` swap refuses the unknown fan, and the subtest's engine-requirements assertion is never reached.

### [LOW] The burler.yaml text sweep misses two members

**Location:** `_lyx/plan/05-burler-reads-lenses.md:60` and `_lyx/plan/06-loom-reads-lenses.md:48`.

**Issue:** `internal/burlerengine/doc.go:77` ("Profile.FixFan names a fan from the burler.yaml lens/fan library") and `:88` ("define it in burler.yaml") sit in the fixer-fan section, outside the cluster fan-out section card 5 names.
`docs/overview.md:551` names "a `burler.yaml` fan" for the eight fan keys as well as for `discussion_lenses` and `plan_lenses`, and card 6 names only the latter.
The scope says every text naming `burler.yaml` as the home of lenses or fans names `lenses.yaml`.

### [LOW] ResolveFan's refusal prefix is left unsettled

**Location:** `_lyx/plan/03-lenses-module.md:62`.

**Issue:** `internal/burlerengine/config.go:126,134,137,147` prefix every `ResolveFan` refusal with `burler: `.
Card 3 moves the function and sets the `lenses: ` prefix for `Load` alone, so the lenses module would keep refusing as burler, and no test catches it.

### [LOW] configengine's package doc is not updated for PerNameMaps

**Location:** `_lyx/plan/02-configengine-per-name-spec.md:11`.

**Issue:** `internal/configengine/doc.go:73-77` documents `Spec.OpenMaps` and its pass-through to yamlengine.
Card 2 adds the parallel `PerNameMaps` field and documents it only on `Spec`, against the rule that cross-cutting infrastructure updates its package doc in the same commit.

### [LOW] A per-name value's shape ignores its path

**Location:** `_lyx/plan/01-yamlengine-per-name-maps.md:39`.

**Issue:** D2 says `--set` takes `lenses.<name>` as a string scalar and `fans.<name>` as a flow list.
Card 1 routes every per-name value through `scalarOrListNode` (`internal/yamlengine/set.go:243-258`), so the value's first character, not its path, decides the shape.
A lens prose starting with `[` becomes a list, which the strict decode then refuses.
The refusal's advice to quote each entry still yields a list.

### [LOW] A partial fix of several failing fans has no working way forward

**Location:** `_lyx/plan/03-lenses-module.md:63`.

**Issue:** `newLoadFailure` (`internal/configcli/configcli.go:265-270`) lets a write through only when the after-load error text equals the before-load text.
With two fans failing, a `--set` that fixes one changes the joined text and is rolled back, under the way forward "--set the key to a value it takes".
D2 prescribes the joined report, and a first-failure report would refuse the same partial fix, so the defect is that the plan states no way forward that works, not the join itself.

### [LOW] --set overwrites a non-mapping per-name value unguarded

**Location:** `_lyx/plan/01-yamlengine-per-name-maps.md:39`.

**Issue:** Card 1 refuses a non-mapping at P in `Reconcile` only.
`SetValues` builds the merged file from the template tree, and the existing `lenses: [a]` matches no template leaf, so the written file replaces the operator's value with the template's mapping.
`setChecked` and `setHubWide` let that write through: the before `CheckLoad` fails on `FillMissing`'s shape mismatch, and the after one passes.

### [LOW] Reconcile's non-mapping refusal names no way forward

**Location:** `_lyx/plan/01-yamlengine-per-name-maps.md:36`.

**Issue:** The refusal stops `lyx config reconcile` and the hub walk behind every start verb.
It names only the path and shape, while the open-map refusal beside it names one (`internal/yamlengine/set.go:213`).

### [LOW] The retired burler module's refusals name no next step

**Location:** `_lyx/plan/08-retire-burler-module.md:40` and `:45`.

**Issue:** `PrepareRunSettings` refuses with `module "burler" is not registered` (`internal/configreg/runsettings.go:49`) and names nothing to do.
Moving the keys into a `lenses` object is refused too, since both are guard keys.
The bound at line 45 leaves a moved `lenses.local.yaml` refused by the lenses load, which burler's wiring reads unconditionally, and names no way to clear it.

### [LOW] SetValues' null-at-P branch has no row

**Location:** `_lyx/plan/01-yamlengine-per-name-maps.md:40`.

**Issue:** Card 1 prescribes the null-to-mapping conversion for `SetValues`, but its rows list reaches only `Reconcile`'s null branch.

### [LOW] The TestPrintResolved lenses subtest repeats an existing pin

**Location:** `_lyx/plan/03-lenses-module.md:87`.

**Issue:** `TestPrintResolved`'s subtest "annotates each unit with its layer" (`internal/configcli/configcli_test.go:1461`) already pins the `# default` annotation through `Provenance`, which is the same for every module.
A lenses subtest adds no behaviour that test leaves uncovered, which `PATTERN-test-economy` forbids.

### [NIT] "An empty dirs" reads as a refused load

**Location:** `_lyx/plan/06-loom-reads-lenses.md:52`.

**Issue:** `configengine.load` refuses a `Dirs{}` that names neither an anchor nor a board dir (`internal/configengine/config.go:141-148`); only a temp dir with no `_lyx/` degrades to the template.

### [NIT] The hub walk's log omits KeptExtra

**Location:** `_lyx/plan/04-configsync-per-name-reconcile.md:24`.

**Issue:** `internal/hubreconcile/walk.go:364-366` logs each report field of a `Result`, `keptTarget` among them.
Card 4 adds `KeptExtra` to the envelope only.

## Rejected

- guard-reach: card 8's `KeepTarget` line form might miss `takeKey`'s sparse `!base` return. Rejected: card 8's Intent states the line form for a kept target "at a base layer as at a sparse one", and its sparse row asserts it, which covers the early return whatever the implementation order.
- way-forward: W1's proposal to change `newLoadFailure`'s comparison. Rejected as out of scope: the finding is kept as F7, downgraded to LOW, with a stated-way-forward fix, since D2 prescribes the joined report the comparison reads.
- guard-reach: the TestRunCmd finding at MEDIUM. Kept as F2 at LOW: the card gate's test run catches the broken row before anything lands.

## Holistic notes

Card order holds: cards 5, 6 and 7 read `internal/lenses` built in card 3, card 4 uses the registration card 3 adds, and card 8 deletes burler's config only after every reader has switched.
Verify covers every package a card targets.
`burlerengine`'s llm-tagged smoke files are vetted, and `loomcli`'s tmux tier references no symbol card 8 deletes.
Every other Decision and Constraint is carried.
D1 by cards 3, 5 and 8; D2 by cards 1 to 4; D3 by card 3; D5 by card 3; D6 by card 8's hub-walk row; the doc constraint by cards 3, 5, 6 and 7, apart from F3 and F5.
The never-prune rule is bounded by the module's declared per-name paths and the guard classification that closes the local layer, which cards 2 and 4 state.
`KeepTarget` is bounded to the two burler moves, which card 8 states.
