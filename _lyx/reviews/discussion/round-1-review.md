---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:182
    summary: Open risks says stale claude_* keys are ignored before the first walk, but a shuttle.local.yaml holding claude_prompt_cache_ttl or claude_prompt_cache_ttl_roles makes every shuttle.yaml load refuse, which blocks stop and teardown and breaks D2's guarantee
    fix: >-
      Add to D4 a decision on how shuttle's loader treats the moved keys before reconcile has moved them, and correct Open risks to match.
      Recommended: shuttle's ConfigSpec declares the five moved source keys as retired keys that configengine.ValidateLocal admits and the loader drops from the merged document, so an unreconciled local layer loads as if the keys were absent, while reconcile still moves them and prunes them.
      State the bound in D4: a retired key is never read, it exists only to be moved, and it is admitted only at the layers and categories it was admitted at before, so the TTL keys at the local layer and nothing else.
      Add an acceptance row: before any reconcile, a pair whose shuttle.local.yaml holds claude_prompt_cache_ttl_roles loads shuttle.yaml, and stop and strand removal work.
      If the writer instead keeps the refusal, D4 must say so, the refusal must name `lyx config reconcile --apply` as the way forward (PATTERN-refusal-way-forward), and D2's sentence that a bad file never blocks stop or teardown must be narrowed to say this case does.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:182
    summary: Open risks lists the binary path, the Agent deny and the TTL roles as the settings not in force before a walk, but omits a switched-off python deny, which resolves to the template's true until D3's rename runs
    fix: >-
      In the Open risks bullet at decision-record.md:182, add the python switch to the list: an operator's claude_deny_python false in an unreconciled base or worktree shuttle.yaml is ignored, so deny_python resolves to true and the python guard is installed in a Python target repo until the walk renames the key.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:77
    summary: At a sparse layer, D4 treats a target key equal to the template default as unset and overwrites it, though there it can be an explicit override of a lower layer, and D4 does not say that overwrite is reported
    fix: >-
      In D4 at decision-record.md:77, limit the template-default overwrite to the base layers, where the scaffolding problem the Rationale names arises.
      At a sparse layer, any value the target already holds wins and the source key is dropped, as a kept-target collision.
      In the sentence at decision-record.md:78, add that a template-default target overwritten by a source value is reported too.
      Extend the acceptance row at decision-record.md:194 with a worktree-layer target holding a template-default value that is kept.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:197
    summary: The acceptance row says PrepareRunSettings turns claude_deny_python into deny_python, but run settings are local-layer documents and deny_python is Guard, so the step after the rename refuses it
    fix: >-
      Rewrite the second clause of the acceptance row at decision-record.md:197 to: a run settings entry `shuttle: {claude_deny_python: …}` is refused as a RunSettingsError whose cause names deny_python.
      In D3 at decision-record.md:67, keep the rename on run settings but add that a renamed deny_python is then refused there like every Guard key.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:129
    summary: D8 claims a required constructor input makes a forgotten site a compile error, but its cycle exception lets a site take the union from a caller, a nil or empty union compiles, and no current construction site needs the exception
    fix: >-
      In D8, delete the sentence at decision-record.md:129: the production reedengine.New sites outside the cli packages, in internal/hubgeom and internal/pairteardown, are outside shuttleengines' dependency closure, so each can import shuttleengines.
      Then bound the claim at decision-record.md:133: state that the compile error catches an omitted argument, not an empty one, and pick the guard for the empty case.
      Recommended: reedengine.New panics on an empty scrub set as a programmer error, and the D8 acceptance row at decision-record.md:201 adds that construction with an empty set fails.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:106
    summary: D5's Rationale narrates the rejected in-process lyx hook verb, a rejected alternative that support-log.md already records and Scope Out already excludes
    fix: >-
      Delete the clause from the first sentence of D5's Rationale at decision-record.md:106, so it reads: the board asks for one neutral table each engine wraps in its own hooks.
      The exclusion stays in Scope Out at decision-record.md:26 and the reason in support-log.md's Rejected alternatives.
---

### [BLOCKING] A stale local-layer TTL key refuses shuttle.yaml before the walk

**Location:** `_lyx/discussion/decision-record.md:182`, with D4 at `:83` and D2 at `:57`

**Issue:** Open risks says that until a layer is reconciled, its stale `claude_*` keys in `shuttle.yaml` are ignored.
That holds for the base and worktree layers only.
`configengine.Layers` (`internal/configengine/layers.go:229`) runs `ValidateLocal` on every local file it reads, and `ValidateLocal` refuses any path that is not in the template (`layers.go:171`, "key %q is not in the %s template").
`claude_prompt_cache_ttl` and `claude_prompt_cache_ttl_roles` are `CategoryNone` today (`internal/shuttleengine/config.go`), so a local file may hold them.
A task's run settings write them there: `battenshed`'s `writeRunSettings` writes each prepared module document as the pair's local override (`internal/battenshed/seamchild.go:144`).
Once the deploy drops them from the shuttle template, every `shuttleengine.LoadConfig` in such a pair fails with `ErrInvalid` until a walk moves the keys.
The walk does not run before every verb: `hubreconcile.Ensure` is called from `fabriccli`, `loomcli`, `orchcli`, `battencli` and `preflightshed` only, not from `shuttlecli`, `reedcli` or `pairteardown`.
So `lyx shuttle stop`, strand removal and the reads that build engines from `shuttle.yaml` refuse in that pair, and so does a pair the walk skips, which Open risks itself names.
That contradicts D2's guarantee that a bad engine config never blocks stop, sweep, removal or reads.
Before this task the same keys could not cause it, because the shuttle template held them.
The design rests on a false premise about how pre-walk stale keys behave, and no Decision bounds the refusal or names its way forward.

### [LOW] Open risks omits the python switch

**Location:** `_lyx/discussion/decision-record.md:182`

**Issue:** D3 renames `claude_deny_python` to `deny_python` only through `MigrateConfig` during reconcile.
Before the walk, an operator's `claude_deny_python: false` at the base or worktree layer is an unknown key, which `load` neither refuses nor reads.
`deny_python` then resolves to its template value `true`.
The python guard is installed in a Python target repo, the case the template comment says the switch exists for.
The Open risks bullet lists every other setting that stops being in force but not this one.
The direction is fail-closed, so it is LOW.
It still belongs in the list the plan writer and operator read.

### [LOW] Template-default overwrite at a sparse layer

**Location:** `_lyx/discussion/decision-record.md:77`

**Issue:** D4 overwrites a target key whose value equals the target template's default with the source value.
The Rationale justifies this by a base `claude.yaml` scaffolded from the template by `--set` or seeding.
At a sparse layer such as the worktree over a repo default, a present key equal to the template default is not scaffolding.
It overrides the layer below.
Take an operator who sets `deny_agent_tool: true` in a worktree `claude.yaml` over a repo default of `false`, while that worktree's `shuttle.yaml` still holds a stale `claude_deny_agent_tool: false`.
The move replaces the explicit `true` with `false`, which downgrades a guard.
D4 reports moves and kept-target collisions, but it does not say that an overwritten target is reported.

### [LOW] An acceptance row expects an output that the next validation step refuses

**Location:** `_lyx/discussion/decision-record.md:197`

**Issue:** `PrepareRunSettings` runs a module's `Migrate` and then `configengine.ValidateLocal` (`internal/configreg/runsettings.go:63`).
`claude_deny_python` is `CategoryGuard` today, so run settings naming it are already refused.
After D3, `deny_python` is Guard too, so the renamed key is refused at the same step.
The row reads as if `PrepareRunSettings` returns a document holding `deny_python`, but its only observable outcome is a refusal.
A plan test written from the row as it reads would fail.

### [LOW] D8's compile-error claim and its cycle exception

**Location:** `_lyx/discussion/decision-record.md:129`

**Issue:** The focus asked whether the compile-error claim matches the exception for a site that cannot import `shuttleengines`.
The claim fails in two ways.
First, the production `reedengine.New` sites outside the cli packages are `internal/hubgeom/spawnorder.go:110` and `internal/pairteardown/teardown.go:366`.
`go list -deps ./internal/shuttleengines` includes neither package, so both can import `shuttleengines` without a cycle, and the exception has no current user.
Second, a required slice or struct input catches an omitted argument, but a nil or empty value compiles.
A site that passes its caller's value through, or a site that passes `nil`, spawns a server that scrubs nothing, the stale-identity leak D8's Rationale names.
The claim is true only for omission, and the Decision leaves the empty case unbounded.

### [NIT] Rejected-alternative narrative in D5's Rationale

**Location:** `_lyx/discussion/decision-record.md:106`

**Issue:** "the in-process `lyx` hook verb was rejected on the board for starting a process on every shell call" is deliberation narrative.
I ran the completeness-before-leanness test.
The requirement it implies, no hook verb, is already in Scope Out at `:26`, and the reason is already in `support-log.md`'s Rejected alternatives, so cutting the clause loses nothing the plan writer needs.

## Rejected

- D5 payload contract: on a payload that does not put the shell command under a `"command"` string key, a guard is partly inert, not wholly inert.
  `commandPositionPattern`'s first alternative anchors on `"command"`, so a first-word command goes unmatched.
  The separator, escaped-newline and wrapper alternatives still match, and so do the raw-go env-name alternative and the rm expansion pattern, since they match anywhere in the payload.
  D5 states the bound: an engine renders against that payload shape, or the task that adds it changes the table and its contract.
  Open risks names `codex-engine` as the task that carries it.
  The rubric's attack-surface test requires a stated bound, and one is stated, so I raise no finding.
- D9 empty-tag mapping: for a non-Claude strand added outside shuttle, the `claude` namer types nothing.
  `repairNames` skips a strand with no `SessionID`.
  `Claude.SessionNameDrift` answers no drift when the transcript is missing (`internal/shuttleengine/claudeengine/sessionname.go:43`), so no `/rename` is typed.
  The Rationale states this bound, so it is not a defect.
- D4 run-settings path: the move runs before each module's migrate step, and `ValidateLocal` against the target's spec follows it.
  So a moved `binary`, `deny_agent_tool` or `deny_ask_user_question` is refused, as the old keys were, and no value reaches a key the target's local rule refuses.
  The stated bound holds for run settings.
  The reconcile-path gap is F1, and F3 covers sparse-layer reconcile.
- D10 relocation pass: the package-doc list and the PATTERN background additions name files the plan must put under `Edit:`, and the `gateslot` departure text is the PATTERN entry's own content.
  All three are requirements, so none of them is relocated.
- Scope and acceptance mapping: every Scope In item maps to a Decision.
  Every acceptance row traces to D1 to D9, with D10's docs covered by CLAUDE.md's same-commit rule.
  No finding beyond F4.
