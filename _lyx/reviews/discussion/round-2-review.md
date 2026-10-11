---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (Its layers merge by D1, so an alias mapping in models merges key by key)
    summary: D2 merges a models alias key by key across layers, reversing modelspec's whole-entry replacement, so an override inherits a lower layer's defaults it cannot remove and existing alias overrides change what they resolve to
    fix: In decision-record.md D2, replace the sentence that a models alias mapping merges key by key with a rule that each top-level models alias is carried whole from the highest layer holding it, as D1 carries an open map, so an upper layer's alias entry replaces the lower one outright; extend D1's open-map sentence, or add one beside it, to name models aliases as whole units; in D6, state that a models alias is compared and stripped as one unit, as an open map is; and add an acceptance criterion that a local models alias without a defaults block resolves with no defaults from the layers below.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 (exempt from the template-lacks refusal) and D8 (is a models alias)
    summary: D5 and D8 exempt a models alias the template lacks but not a key path under an alias the template holds, so a local or run_settings opus.defaults.version, which modelspec accepts, is refused, against D2's rule that a seed-only module's key set is the operator's
    fix: In decision-record.md D5, change the models exemption to cover every key path under a top-level alias in the models local file, whether or not the template holds that alias, with the alias block validated by the models loader at load; make D8's validation clause say the same (a key path under any models alias); and widen the acceptance criterion reading A models local file and run_settings accept an alias the template lacks to also accept an entry such as opus.defaults.version under an alias the template holds.
---

### [MEDIUM] models aliases merge key by key, reversing modelspec's whole-entry replacement

**Location:** `_lyx/discussion/decision-record.md` D2, and D1's merge rule it applies.

**Issue:** D2 says a seed-only module's layers merge by D1, "so an alias mapping in `models` merges key by key".
The models loader today does the opposite on purpose.
`internal/modelspec/load.go` (`LoadRegistry`) replaces each built-in alias with the file's entry whole, with the comment "Whole-entry replacement ... no field-level merge, so an override can never leak a stale built-in default."
Under D2 the template becomes a layer below every models file, and D1 says no layer can delete a key from a layer below.
Two consequences follow, and the record names neither:

- An existing models file whose `opus` entry has no `defaults` block resolves today with no defaults; under D2 it picks up the template's `defaults.effort: medium`.
  D6 then promises that the strip "never changes what a key the worktree file holds resolves to", but D2's merge has already changed it before any strip runs.
- A local or `run_settings` override `opus: {engine: claude, model: <id>}` keeps every `defaults` key the repo default holds, including a `version` default (a known `Entry.Defaults` key in `modelspec`'s `knownParams`) pinned to a different model.
  The override has no way to drop it, which is the stale-default leak `modelspec` was written to prevent.

The models alias is an open-ended, operator-owned entry, the same shape of thing D1 already carries whole for an open map.
D1's open-map rule fits it, and D2's own rationale, that the operator owns the seed-only key set, argues for it.
The class is `design` because the decision rests on a premise about today's models resolution that the code contradicts.

### [LOW] D5's models exemption stops at the alias name

**Location:** `_lyx/discussion/decision-record.md` D5 ("an alias in the `models` local file") and D8 ("or is a `models` alias"), and the acceptance criterion "A `models` local file and `run_settings` accept an alias the template lacks."

**Issue:** The exemption is worded for an alias name.
A key path under an alias the template does hold, such as `opus.defaults.version`, is neither an alias the template lacks nor under an open map.
D5's refusal and D8's path check therefore reject it, though `modelspec`'s `knownParams` admits `version` in `Entry.Defaults` and D2 says a seed-only module's key set is the operator's at every layer.
The acceptance criterion pins only the alias-the-template-lacks case, so the plan writer reads the narrower rule.
If F1's fix makes each alias a whole unit, this exemption becomes "any alias block", which is the natural wording to adopt alongside it.

## Settled focus questions

- D4's per-worktree bound holds.
  `reed.shell`, `reed.tmux`, `reed.tmux_conf`, `reed.mouse`, `reed.debug_log`, `shuttle.claude`, `shuttle.run_dir` and every `shuttle.claude_deny_*` key are in `LocalDenied`.
  D5 refuses them at load, and D8 refuses them at board write.
  D4 states the bound and the residual: the untracked local file is unseen at landing, which R3 names along with the `.env` route.
  I tested the keys D4 leaves open against its categories.
  `webster.verify_gate_attempts` at 0 would switch off the plan-level verify gate, but `websterengine`'s load refuses any value below 1.
  `darn.verify_attempts` is refused below 1 by `loomrecipe.NewDarn`, and loom's `review_max_bounces` below 1 by `loomrecipe.New`.
  `logger`'s retention keys sweep only the anchor's own `.lyx/logs`, so they are not a cross-pair fact.
  No open key I checked switches off a guard.
- The seed-only rule is stated.
  D2 says reconcile never rewrites, prunes or migrates a seed-only layer.
  The reconcile acceptance criterion exempts seed-only modules, and D4 closes `burler`'s local layer entirely.
  D5 and D8 agree on the exemption for a `models` alias the template lacks, apart from F2.
- D6 states that a pin equal to the seeded default becomes inherited, and that a later `--layer default` change reaches it.
  Its bound is the per-worktree `stripped` list, plus the `inherited` list for keys the worktree lacked.
- D5, D8, the acceptance criteria and R1 and R3 are consistent with D4's current lists.
  No requirement the plan writer needs lives in `support-log.md` only.
